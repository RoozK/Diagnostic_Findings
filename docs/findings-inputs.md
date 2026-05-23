# Findings Inputs — Field Inventory

Source: Berry Studio → Patient → Care Timeline → Findings Inputs (Lite view + edit modal).
Extracted from screenshots provided 2026-05-23. Some sub-options are inferred from
the read-only summary in the lite view where the edit modal was not visible; those
are marked _(inferred)_.

This document is the source of truth for the voice-to-form mapping in the
companion app. Every field below needs a canonical key, a value domain, and a
list of natural-language phrases that should map to each value.

---

## Patient Header (context, not editable from this form)

- Name
- Sex • Age (e.g. "Female • 28y, 10m")
- Date of birth
- Contract Start Date
- Exam Date
- Treatment status (e.g. "Inactive Tx")
- Phase / Aligner phase
- Location (e.g. "Napa")

---

## 1. General

| Field                        | Type              | Values / Notes                                          |
| ---------------------------- | ----------------- | ------------------------------------------------------- |
| Chief Complaint              | dropdown (single) | e.g. "Supernumerary tooth/teeth" — full list TBD        |
| Chief Complaint (Words)      | free text         | verbatim patient quote                                  |
| Dentition                    | radio (one of)    | Primary, Mixed, Permanent                               |

---

## 2. Skeletal Diagnosis

### Growth
- Radio: **Grower** / **Non-Grower**

### Anteroposterior
- Class: **Class 1** / **Class 2** / **Class 3** (radio)
- **Mx**: Retrognathic / Prognathic _(radio, optional)_
- **Md**: Retrognathic / Prognathic _(radio, optional)_

### Vertical
- **Mx**: Vertical Excess (checkbox)
- **Md**: Hypodivergent / Hyperdivergent / Normodivergent (radio)

### Transverse
- **Mx**: Constricted (checkbox)

### Asymmetry
- **Mx**: Left / Right (radio, optional)
- **Md**: Left / Right (radio, optional)

---

## 3. Dental Diagnosis

### Anteroposterior
- Class: **Class 1** / **Class 2** / **Class 3** (radio)
- If Class II/III: side — **Left** / **Right** / **Bilateral** (radio)
- **Pseudo** (checkbox, applies to Class III)
- **Div I / Div II** _(inferred — appears in summary "Class II • Bilateral • Div I")_
- **Functional shift**: mm (numeric input)
- **Overjet**: mm (numeric input) _or_ **Negative Overjet** (radio toggle)
- **Anterior crossbite – Teeth**: tooth-chart picker (UR, UL, LR, LL quadrants, individual teeth)
- **Complete anterior crossbite**: mm (numeric input)

### Vertical
- **Overbite**: mm (numeric input) — exclusive with Deepbite/Openbite
- **Deepbite** (radio)
- **Openbite** (radio)

### Transverse
- **Arch Shape Upper**: Ovoid / Tapered / Square / Asymmetric
- **Arch Shape Lower**: Ovoid / Tapered / Square / Asymmetric
- **Midline Shift Upper**: Left / Right + mm
- **Midline Shift Lower**: Left / Right + mm
- **Posterior CrossBite**:
  - Unilateral / Bilateral (checkbox)
  - Left / Right (radio, if unilateral)
  - Teeth (tooth-chart picker)
  - Brodie bite (checkbox)
- **Functional Shift** _(also surfaced here in summary)_: mm

### Alignment _(from lite summary)_
- **Alignment Crowding – Upper**: mm
- **Alignment Crowding – Lower**: mm
- **Alignment Spacing – Upper**: mm
- **Alignment Spacing – Lower**: mm
- **Bolton Discrepancy**: None / Upper / Lower
- **Missing Teeth**: list (e.g. "Upper Right 4, Bottom Right 3")
- **Impacted Teeth**: list (e.g. "Upper Right 3")
- **Position** (for missing/impacted): free text or tooth picker
- **Peg Lateral**: list of teeth (e.g. "UL")
- **Misshaped Teeth**: list (e.g. "Upper Right 3")

### Smile Esthetics _(from lite summary)_
- **Excess gingival show**: mm
- **Cant**: Upper / Lower
- **Smile Line**: Flat / Consonant / Reverse
- **Upper Incisal Display**: Excess / Deficient / Normal
- **Buccal corridors**: Narrow / Wide / Normal

### General Pathology _(from lite summary; each item lists affected teeth)_
- Root Canal Treated Teeth
- Crowns
- Implant
- 3rd Molar Impaction
- Supernumerary Teeth
- Root Resorption
- Enamel Destruction
- Other Pathology (free text)

---

## 4. Periodontal Analysis

- **Oral Hygiene**: Good / Fair / Poor
- **Frenum**: multi-select — Frenectomy Recommended / High Mx Labial / High Md Labial
- **Periodontitis**: Yes / No
- **Gingivitis**: Yes / No
- **Recession (Quadrant-Based)**: UR / UL / LR / LL (multi-select)
- **Bone Loss (Quadrant-Based)**: UR / UL / LR / LL (multi-select)
- **Thin Gingival Phenotype**:
  - Mx (Maxilla): Yes / No
  - Md (Mandible): Yes / No

---

## 5. Soft Tissue Diagnosis

### Anteroposterior
- **Profile**: Concave / Convex / Straight
- **NL Angle**: Average / Acute / Obtuse
- **Lips**: U Protrusive / U Retrusive / L Protrusive / L Retrusive (multi)

### Vertical
- **Lower Facial Height**: Increased / Decreased / Normal
- **Lips**: Competent / Incompetent
- **Upper Lip**: Short / Normal / Long

### Transverse
- **Chin Deviation**: L (Left) / R (Right) / None

---

## 6. TMJ Analysis

- **Clicking/Popping (Present)**: Left / Right / Bilateral / None
- **Clicking/Popping (History)**: Left / Right / Bilateral / None
- **Crepitus (Present)**: Left / Right / Bilateral / None
- **Crepitus (History)**: Left / Right / Bilateral / None
- **Limited range of motion**: mm
- **Pain/Tenderness**: site picker — e.g. "Bilateral Masseter", "Left TMJ", etc.
- **Headaches/Neckaches/Shoulder aches**: Yes / No
- **Bruxism**: Yes / No

---

## 7. Sleep Apnea Screening — Adult

### Symptoms (each Yes/No)
- Snoring
- Gasping During Sleep
- Mouth Breathing
- Daytime Fatigue / Sleepiness

### History
- History of Sleep Apnea (Hx): Yes / No
- CPAP Use: Yes / No

### Clinical Evaluation
- **Mallampati Score**: Class I / II / III / IV
- **Tonsil Grade**: Grade 0 / 1 / 2 / 3 / 4

### Sleep Questionnaire
- Administer Sleep Questionnaire: Yes / No

> Pediatric variant likely exists too — confirm whether the app needs to support both.

---

## 8. Habits (each None / Present)

- Thumbsucking
- Tongue thrusting
- Lip sucking
- Nail biting
- Grinding/Clenching

---

## Open questions about the form itself

1. Full enumerations for dropdowns — most importantly **Chief Complaint** — need to be captured from the live app (every option in the dropdown).
2. Numeric fields: what's the valid range / decimal precision? Are decimals (e.g. 2.5 mm) accepted?
3. Tooth charts use a notation that mixes "Upper Right 3" prose with UR/UL/LR/LL codes and standard tooth numbering (1–8 per quadrant). Decide on a single canonical key per tooth (FDI 11–48? Universal 1–32? Berry's own scheme?).
4. Are any fields required for save, or is the whole form optional/partial?
5. Is there an API to write back to Berry Studio, or does this app produce a fillable artifact (JSON, copy-paste, browser-automation)?
