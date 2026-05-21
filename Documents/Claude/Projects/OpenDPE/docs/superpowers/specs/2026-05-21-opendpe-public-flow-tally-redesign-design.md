# OpenDPE Public Flow Redesign

Date: 2026-05-21
Status: approved conversation design, pending user review of this written spec
Scope: `opendpe-net` public flow only

## Intent

Redesign the OpenDPE public journey around the business path that matters:

1. identify a property from ADEME-backed context or supported import;
2. generate three precise renovation scenarios;
3. let the user choose one scenario;
4. validate and refine every action line before estimation;
5. feed the validated scope into a devis estimator;
6. send the estimate toward professional confirmation, partner-first with a user-chosen third-party alternative.

The redesign should follow Tally's attention-management model: calm focus, low-noise UI, clear steps, and one dominant decision per screen. It should not copy Tally literally or turn OpenDPE into a generic form builder.

## Product Positioning

### Primary public promise

OpenDPE helps renovation professionals turn a property and its DPE context into a detailed renovation scenario that can be reviewed line by line and advanced toward a professionally confirmed devis.

### Audience priority

Primary:

- renovation professionals qualifying a property and refining renovation scope;
- professionals who need a traceable scenario before asking for a quote confirmation.

Secondary:

- particuliers who need guided understanding of renovation options;
- particuliers intentionally using advanced DPE tools.

### Simulator position

The raw 3CL simulator is not the public business driver.

It remains:

- an expert engine for renovation professionals;
- an advanced path for particuliers trying to establish their own DPE;
- accessible from the public experience without being the primary funnel destination.

The redesign must avoid selling the difficult simulator as the main outcome.

## Existing Context

The active app is the React/Vite app in `opendpe-net`.

Relevant implementation signals observed before design:

- React 18 and Vite from `opendpe-net/package.json`;
- Tailwind CSS and shadcn-style HSL variables from `src/index.css` and `tailwind.config.js`;
- `Nunito` loaded by `src/Layout.jsx`;
- `framer-motion` installed and already used in public and app surfaces;
- many route-level pages exist, so a root-level app redesign would be multi-page, while this work is intentionally limited to the public funnel.

## Approved Design Direction

### Recommended model

Use a **scenario review pipeline**.

The public flow is not a marketing page that points at a simulator. It is a guided operational path:

```text
Property context
  -> Three scenarios
  -> Choose one
  -> Editable scope review
  -> Devis estimate
  -> Professional confirmation
```

### Why this model

This model:

- aligns the interface with the business driver;
- keeps professional work visible and reviewable;
- preserves OpenDPE's technical engine without over-promoting it;
- supports a Tally-like reduction of noise while retaining enough density for professionals.

## Journey Design

### Step 1. Property context

The public entry surface should focus on identifying the property context needed to generate renovation scenarios.

Primary actions may include:

- search by address or DPE reference when ADEME data is available;
- supported import or continuation paths where the current product already supports them.

The page should explain in concise language that ADEME-backed data is used when available.

### Step 2. Three renovation scenarios

After property identification, the product should produce three precise renovation scenarios.

Each scenario should expose enough information for a serious choice:

- scope of work;
- expected DPE effect when known by the product;
- key assumptions;
- estimate readiness or review needs.

The screen should drive a single decision: choose one scenario to review.

### Step 3. Scenario review checklist

The selected scenario opens into the core public business surface.

Use a review checklist with editable lines rather than a one-item-at-a-time questionnaire.

Each renovation line should support:

- validation state;
- action label;
- quantity;
- quality or specification assumption;
- edit or replace behavior where supported;
- removal where appropriate;
- search/add for additional renovation choices or alternatives.

The whole scope must remain scannable so a professional can inspect the scenario as a coherent package before estimation.

### Step 4. Devis estimator

The estimate should be generated only from the reviewed scenario scope.

The estimator handoff must make two points explicit:

- the estimate is based on validated actions, quantities, and quality assumptions;
- professional confirmation is still required.

### Step 5. Professional confirmation

The default handoff is partner-first.

The UI should:

- visually favor confirmation through selected partners;
- provide "use my own professional" as a clear secondary alternative;
- avoid implying that the computed estimate is already a final accepted quote.

## Surface Structure

### Public entry

Replace the current public emphasis on the 3CL simulator with professional property intake and the scenario pipeline promise.

### ADEME lookup / DPE search

Keep public lookup utility, but shape its outcome around continuation into scenario generation rather than browsing or detouring into expert tooling.

### Scenario selection

Create or reshape a surface where three scenarios can be compared and one can be chosen with confidence.

### Scenario review

Treat this as the product center of gravity for the funnel.

It should feel closer to a structured work review than a promotional card grid:

- editable lines;
- visible assumptions;
- clear validation progress;
- additive search for missing or alternative actions.

### Estimate and handoff

Use a calm summary and decision surface for estimate readiness and professional confirmation.

### Expert simulator

Keep the simulator reachable, but place it in an explicit advanced/expert branch.

## Visual System

### Genre and tone

- Genre: modern-minimal.
- Tone: professional, calm, exact.

### Tally influence

Adopt:

- focus-first composition;
- low-noise chrome;
- generous spacing around the active decision;
- clear progression;
- restrained confirmation and validation states.

Do not adopt:

- generic consumer-form softness where it weakens professional credibility;
- literal cloning of Tally page structure or branding;
- oversimplification that hides renovation scope detail.

### OpenDPE identity

DPE class colors remain semantic data colors, not broad decoration.

The public flow should read as a diagnostic and renovation-scoping instrument, not a renovation marketplace landing page.

### Typography

Retain `Nunito` as the first implementation constraint for continuity unless implementation review shows it fails the sharper professional hierarchy.

Use:

- stronger hierarchy by weight, scale, and spacing;
- concise screen headings;
- controlled explanatory copy;
- compact but readable line-item text in checklist surfaces.

### Color and surfaces

Move public flow surfaces toward:

- a light, restrained paper-like base;
- strong dark text;
- one controlled progression accent;
- semantic DPE and validation colors only where meaning requires them.

Prefer:

- flow sections;
- review rows;
- attached summaries;
- visible status rails.

Avoid:

- card-heavy marketing grids;
- decorative gradients that compete with DPE data;
- competing CTA clusters.

### CTA voice

Each screen should have one dominant primary action.

Secondary actions should be visible but quieter:

- expert simulator branch;
- third-party professional path;
- optional detail expansion.

## Interaction Rules

One screen, one dominant decision:

- identify the property;
- choose the scenario;
- validate the scope;
- estimate the devis;
- choose confirmation path.

Scenario review must not degrade into a hidden-step wizard. Professionals need full-scope scanability.

Search/add in the checklist is part of the main workflow, not an afterthought.

All estimate language must preserve the distinction between:

- calculated estimate from reviewed scope;
- final professional confirmation.

## Architecture Guidance

Implementation should preserve existing route boundaries where practical and make them feel like a continuous journey through shared flow context.

Expected architecture direction:

- shared public-flow shell for progress and continuity;
- focused entry, scenario selection, review, estimate, and confirmation surfaces;
- explicit expert simulator branch rather than simulator-first home routing;
- reuse existing scenario, recommendation, estimator, and quote-related modules where behavior already exists;
- avoid touching unrelated authenticated, admin, CRM, analytics, pricing, and helpdesk surfaces in the first redesign pass.

The exact route mapping should be decided during implementation planning after the existing public pages and scenario/devis modules are inventoried.

## Error And Trust States

Public flow must account for:

- missing or incomplete ADEME property data;
- scenario generation not ready or incomplete;
- unvalidated quantities or quality assumptions;
- estimate blocked until required review conditions are satisfied;
- explicit disclaimer that confirmation is professional, not purely automated.

Fallback messaging should keep the user oriented in the pipeline rather than dumping them into the raw simulator by default.

## Testing And Verification

Implementation planning should include:

- route-level verification of the public flow continuity;
- responsive checks for entry, scenario comparison, checklist editor, estimate summary, and handoff;
- interaction tests for choosing a scenario, editing checklist lines, searching/adding actions, and selecting partner-first versus own-professional handoff;
- regression checks that the expert simulator remains reachable without becoming the main funnel;
- visual QA against the approved modern-minimal, form-first direction.

## Out Of Scope

This redesign spec does not include:

- admin, CRM, analytics, helpdesk, or authenticated dashboards;
- a full rewrite of the 3CL engine;
- removing or replacing the simulator;
- partner marketplace logic beyond the partner-first versus own-professional handoff posture;
- inventing factual metrics, savings promises, or quote guarantees not already supported by the product.

## Open Implementation Questions

These should be resolved in the implementation plan from the codebase:

- which existing public route should host scenario comparison;
- which existing scenario/recommendation components already support action-level quantity and quality data;
- how much of the devis estimator path already exists in `QuoteRequest`, `ProjectPlan`, or related modules;
- what persistence model already exists for a selected and edited scenario.
