# OpenDPE - Spec V1 Carte DPE OpenLayers/Rust

Date: 2026-06-10
Statut: valide pour plan d'implementation

## 1. Objectif

La V1 ajoute a OpenDPE une carte immobiliere centree sur un DPE. Elle permet de partir d'un numero DPE, d'une adresse DPE ou de la derniere analyse DPE disponible, puis d'afficher les donnees publiques utiles autour du logement: localisation, cadastre, DPE ADEME proches et risques principaux.

Cette version remplace les enrichissements geographiques non deterministes par des appels API publics explicites. En particulier, les donnees reglementaires ne doivent plus etre produites par `InvokeLLM`.

## 2. Perimetre Fonctionnel V1

### 2.1 Entrees Utilisateur

La page carte accepte trois modes d'entree:

1. Numero DPE ADEME.
2. Adresse saisie ou issue d'une analyse DPE.
3. Derniere analyse `DPEAnalysis` existante dans OpenDPE.

Priorite de localisation:

1. `_geopoint` ADEME si le DPE le fournit.
2. Coordonnees deja stockees dans `Comparison.ban_data.coordinates`.
3. Geocodage BAN/Géoplateforme a partir de l'adresse.
4. Etat d'erreur recuperable si aucune localisation fiable n'est trouvee.

### 2.2 Carte

La carte V1 utilise OpenLayers et affiche:

- un fond de carte OSM par defaut;
- un marqueur "logement DPE";
- un cercle de recherche configurable: 100 m, 250 m, 500 m;
- des couches vectorielles GeoJSON;
- une fiche detail pour l'objet selectionne;
- une legende et un etat de chargement par couche.

Le rayon par defaut est 250 m.

### 2.3 Couches V1

#### Localisation

Sources:

- BAN/Géoplateforme pour geocodage.
- API Geo pour commune/code INSEE si necessaire.

Donnees affichees:

- adresse normalisee;
- latitude/longitude;
- code postal;
- commune;
- code INSEE;
- score ou niveau de confiance.

#### DPE ADEME

Source:

- ADEME `dpe03existant`.

Donnees affichees:

- DPE cible;
- DPE proches dans le rayon;
- classe energie;
- classe GES;
- date d'etablissement;
- surface habitable;
- type de batiment;
- annee de construction si disponible.

Style carte:

- points colores par etiquette DPE;
- DPE cible mis en avant.

#### Cadastre

Sources:

- API Carto Cadastre / cadastre.data.gouv.fr selon disponibilite.

Donnees affichees:

- parcelle intersectant le point DPE;
- parcelles voisines dans le rayon;
- identifiant parcelle;
- section;
- numero;
- commune;
- surface si disponible.

Style carte:

- parcelle cible en contour bleu plein;
- parcelles voisines en contour gris;
- remplissage leger pour eviter de masquer le fond.

#### Risques Principaux

Source:

- API Géorisques.

Risques V1:

- PPR;
- retrait-gonflement des argiles;
- radon communal;
- sismicite;
- SIS / sols pollues;
- CASIAS/BASIAS si disponible;
- ICPE si disponible.

Donnees affichees:

- type de risque;
- niveau d'exposition;
- distance ou intersection quand l'API le permet;
- source;
- date ou version si disponible.

Style carte:

- zonages en polygones transparents;
- points de risque avec icone et couleur par severite;
- resume textuel dans le panneau detail.

## 3. Hors Perimetre V1

Les elements suivants sont exclus de la V1:

- DVF, DVF+ et transactions immobilieres.
- BDNB complete et RNB vector tiles.
- Urbanisme GPU/PLU/SUP.
- Logement social, vacance, copropriete detaillee.
- Scores composites de qualite immobiliere.
- Export PDF avance.
- PostGIS obligatoire.
- Sources a acces limite: MAJIC, DV3F restreint, LOVAC detaille nominatif, Patrim, BIEN/PERVAL, fichiers proprietaires.

## 4. Architecture

### 4.1 Vue D'ensemble

La V1 utilise une architecture React/OpenLayers + proxy Rust.

```text
React Vite
  |
  | HTTP JSON/GeoJSON
  v
geo-context-api Rust
  |
  | appels API publics + cache
  v
BAN / ADEME / Cadastre / Géorisques
```

React gere l'affichage, les interactions carte et l'etat UI. Rust gere la normalisation, le cache, les erreurs API, les timeouts et les transformations en contrats stables.

### 4.2 Frontend

Dependance a ajouter:

- `ol`

Fichiers cibles:

- `src/pages/DpeMap.jsx`: page principale carte DPE V1.
- `src/components/map/OpenLayersMap.jsx`: initialisation OpenLayers et rendu des couches.
- `src/components/map/DpeMapHeader.jsx`: adresse, classe DPE, statut de geocodage.
- `src/components/map/LayerPanel.jsx`: activation, opacite et statut des couches.
- `src/components/map/FeatureInspector.jsx`: fiche de l'objet selectionne.
- `src/services/geoContext/client.js`: client HTTP vers Rust.
- `src/services/geoContext/layers.js`: catalogue frontend des couches V1.

Leaflet peut rester utilise ailleurs. La V1 ne doit pas refactorer toutes les pages contextuelles existantes.

### 4.3 Backend Rust

Nouveau service:

- `geo-context-api`

Stack recommandee:

- `axum`;
- `tokio`;
- `reqwest`;
- `serde`, `serde_json`;
- `geojson`;
- `tower-http` pour CORS, tracing et compression;
- cache memoire TTL pour V1.

Stockage V1:

- cache memoire TTL par URL/API et parametres;
- pas de base obligatoire.

Evolution prevue:

- PostgreSQL/PostGIS en V2+ pour cache geospatial persistant.

## 5. Contrats API

### 5.1 Health

```http
GET /health
```

Reponse:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

### 5.2 Geocodage

```http
GET /geocode?q=8%20bd%20du%20port
```

Reponse:

```json
{
  "query": "8 bd du port",
  "label": "8 Boulevard du Port 80000 Amiens",
  "lat": 49.894,
  "lon": 2.295,
  "postcode": "80000",
  "city": "Amiens",
  "cityCode": "80021",
  "score": 0.97,
  "source": "BAN"
}
```

### 5.3 Localisation DPE

```http
GET /dpe/:numeroDpe/location
```

Reponse:

```json
{
  "numeroDpe": "2475E0133744A",
  "location": {
    "lat": 48.8566,
    "lon": 2.3522,
    "source": "ADEME",
    "confidence": "high"
  },
  "dpe": {
    "etiquetteDpe": "D",
    "etiquetteGes": "B",
    "adresse": "adresse normalisee",
    "dateEtablissement": "2024-01-15",
    "surfaceHabitable": 62.4,
    "typeBatiment": "appartement"
  }
}
```

### 5.4 Contexte Global

```http
GET /context?lat=48.8566&lon=2.3522&radius=250
```

Reponse:

```json
{
  "center": {
    "lat": 48.8566,
    "lon": 2.3522
  },
  "radiusMeters": 250,
  "layers": [
    {
      "id": "dpe-ademe",
      "status": "ready",
      "featureCount": 12
    },
    {
      "id": "cadastre-parcelles",
      "status": "ready",
      "featureCount": 8
    },
    {
      "id": "georisques",
      "status": "partial",
      "featureCount": 3,
      "warnings": ["ICPE indisponible pour ce point"]
    }
  ]
}
```

### 5.5 Features par Couche

```http
GET /layers/:layerId/features?lat=48.8566&lon=2.3522&radius=250
```

Reponse GeoJSON:

```json
{
  "type": "FeatureCollection",
  "metadata": {
    "layerId": "cadastre-parcelles",
    "source": "API Carto Cadastre",
    "license": "open data",
    "generatedAt": "2026-06-10T10:00:00Z"
  },
  "features": []
}
```

## 6. Modele Couche

```ts
type LayerDefinition = {
  id: string;
  label: string;
  domain: "location" | "dpe" | "cadastre" | "risk";
  sourceName: string;
  geometry: "point" | "line" | "polygon" | "mixed";
  defaultVisible: boolean;
  minZoom?: number;
  maxRadiusMeters: number;
  styleKey: string;
};
```

Couches V1:

```ts
[
  {
    id: "dpe-ademe",
    label: "DPE ADEME",
    domain: "dpe",
    sourceName: "ADEME dpe03existant",
    geometry: "point",
    defaultVisible: true,
    maxRadiusMeters: 500,
    styleKey: "dpe-class"
  },
  {
    id: "cadastre-parcelles",
    label: "Parcelles cadastrales",
    domain: "cadastre",
    sourceName: "API Carto Cadastre",
    geometry: "polygon",
    defaultVisible: true,
    maxRadiusMeters: 500,
    styleKey: "cadastre"
  },
  {
    id: "georisques",
    label: "Risques principaux",
    domain: "risk",
    sourceName: "Géorisques",
    geometry: "mixed",
    defaultVisible: true,
    maxRadiusMeters: 500,
    styleKey: "risk"
  }
]
```

## 7. Gestion Erreurs et Donnees Partielles

Chaque couche a son propre etat:

- `idle`;
- `loading`;
- `ready`;
- `empty`;
- `partial`;
- `error`.

Une erreur sur une source ne bloque pas la carte. Le panneau couche affiche l'erreur lisible et permet de relancer la couche.

Timeouts recommandes:

- geocodage: 4 s;
- DPE ADEME: 8 s;
- cadastre: 8 s;
- risques: 10 s.

Le backend normalise les erreurs API dans ce format:

```json
{
  "error": {
    "code": "UPSTREAM_TIMEOUT",
    "message": "La source Géorisques n'a pas repondu dans le delai imparti.",
    "source": "georisques"
  }
}
```

## 8. Securite, Donnees et Conformite

- Ne pas integrer de donnees proprietaires ou nominatives.
- Ne pas appeler les sources a acces limite dans la V1.
- Ne pas stocker les reponses brutes contenant des champs inutiles.
- Afficher source et licence quand disponibles.
- Eviter les cles API cote frontend. Si une source necessite une cle, elle passe par Rust.
- Journaliser uniquement les metadonnees techniques: source, statut, duree, nombre d'entites.

## 9. Tests et Verification

### 9.1 Frontend

Tests attendus:

- rendu page sans DPE;
- rendu avec DPE localise;
- activation/desactivation couche;
- affichage fiche detail;
- gestion erreur couche.

Verification manuelle:

- carte non blanche;
- zoom et centrage corrects;
- couches visibles;
- selection feature fonctionnelle;
- responsive desktop/tablette.

### 9.2 Backend

Tests attendus:

- `/health`;
- geocodage adresse valide;
- geocodage adresse introuvable;
- normalisation reponse ADEME;
- normalisation GeoJSON cadastre;
- erreur source transformee proprement;
- cache TTL.

### 9.3 Jeux d'Essai

La V1 doit etre testee sur au moins cinq situations:

1. adresse urbaine dense;
2. maison individuelle rurale;
3. copropriete/appartement;
4. zone avec risques argiles ou PPR;
5. adresse sans `_geopoint` ADEME necessitant BAN.

## 10. Criteres d'Acceptation

La V1 est acceptable si:

- une adresse ou un numero DPE affiche une carte OpenLayers centree correctement;
- les couches DPE, cadastre et risques chargent de facon independante;
- une erreur API n'empeche pas les autres couches de fonctionner;
- chaque feature selectionnee affiche une fiche detail claire;
- chaque couche affiche sa source;
- aucun enrichissement reglementaire n'est produit par `InvokeLLM`;
- `npm run build` passe cote frontend;
- les tests backend Rust passent.

## 11. Decisions Validees

- OpenLayers est le moteur carte de la V1.
- Leaflet n'est pas supprime globalement en V1.
- Rust sert de proxy/normalisateur d'API publiques.
- Le cache V1 est en memoire avec TTL.
- PostGIS, DVF, GPU et BDNB complete sont repousses apres V1.

## 12. Prochaine Etape

Apres validation de cette spec, produire un plan d'implementation detaille decoupe en lots:

1. ajout du service Rust `geo-context-api`;
2. client frontend `geoContext`;
3. page OpenLayers;
4. connecteurs BAN/ADEME/Cadastre/Géorisques;
5. tests et verification navigateur.
