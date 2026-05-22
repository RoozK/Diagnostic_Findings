# Findings Inputs — Structural Map

> **Source:** https://app.berrystudio.ai/practice/patient/view/85/care-timeline/findings-inputs
> **Captured:** <YYYY-MM-DD>
> **Purpose:** input to redesign planning. Structure only — no PHI.

## Page context

- **Route pattern:** `/practice/patient/view/:patientId/care-timeline/findings-inputs`
- **Parent flow:** <one sentence: where does this page live in the user journey>
- **Breadcrumbs:** <list, top → current>
- **Tabs / side nav at this level:** <list>
- **Page-level layout:** <e.g., header + left side nav + main column + right rail>

## Full-page screenshot

![Findings inputs — full page](./screenshots/findings-inputs-full.png)

## Sections

<!--
Repeat the block below for every distinct section on the page (in visual
order, top to bottom). A "section" is any grouping with its own heading,
card, accordion, or tab.
-->

### Section: <name>

![<name> section](./screenshots/findings-inputs-<slug>.png)

**Purpose (inferred):** <one sentence>

**Layout:** <columns, grouping, accordion/collapsed-by-default, etc.>

**Fields:**

| # | Label | Type | Required | Placeholder | Helper / validation | Options (for select/radio/checkbox) | Notes |
|---|---|---|---|---|---|---|---|
| 1 | <label> | <type> | yes/no | <placeholder> | <hint> | <opt1; opt2; opt3> | <e.g., shown only when X = Y> |

**Actions in this section:**

- `<Button label>` — <role: primary / secondary / destructive / link> — <what it appears to do>

---

### Section: <next>

<!-- ...repeat... -->

## Global / page-level actions

- `<Save>` — primary — <behaviour>
- `<Cancel>` — secondary — <behaviour>
- `<Delete>` — destructive — <behaviour, confirmation modal?>

## Conditional logic observed

- When `<field A>` = `<value>`, `<field B>` appears / becomes required / changes options.
- ...

## Cross-cutting observations

- **Validation pattern:** <inline? on blur? on submit?>
- **Empty / loading / error states observed:** <list>
- **Autosave vs explicit save:** <which>
- **Keyboard / accessibility cues noticed:** <list>

## TODOs / unclear

- <anything you couldn't verify — e.g., "Section X is behind a modal triggered by saving, did not open">
