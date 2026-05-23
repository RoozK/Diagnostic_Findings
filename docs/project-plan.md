# Voice-Driven Findings App — Project Plan

## What we're building (one paragraph)

A tool that listens to a consultation conversation, transcribes it with
Deepgram, and uses an LLM to fill out the Berry Studio **Findings Inputs**
form. Later versions generate (a) a short, friendly patient-facing blurb
summarizing what was discussed, with selectable prompt styles, and (b) a
formal letter (referring provider, treatment plan, etc.).

## Constraints (locked — see `docs/decisions.md`)

- **Runtime:** web app on clinic laptop (Next.js + TS + Tailwind, single repo).
- **Transcription:** Deepgram (Enterprise tier with BAA).
- **LLM:** Anthropic Claude via AWS Bedrock (uses AWS BAA).
- **Berry sync:** manual paste-back of JSON/text (no Berry API integration v1).
- **Capture:** live streaming preview during consult + batch re-extraction after.
- **PHI:** single-clinic use, BAAs required from day one.
- **Storage:** AWS RDS Postgres + S3 (SSE-KMS) for audio/transcripts.
- **v1 output:** short patient-friendly summary only (SMS-length, selectable tone). Letters deferred.

---

## Architecture sketch (subject to your answers)

```
                                     ┌────────────────────────┐
  Mic ──► Deepgram streaming ──────► │   Transcript buffer    │
                                     │  (rolling, time-coded) │
                                     └──────────┬─────────────┘
                                                │  (every N sec
                                                │   or "extract now")
                                                ▼
                                     ┌────────────────────────┐
                                     │  LLM extractor         │
                                     │  → strict JSON shaped  │
                                     │  by findings schema    │
                                     └──────────┬─────────────┘
                                                │
                                                ▼
                                     ┌────────────────────────┐
                                     │  Findings state store  │
                                     │  (per-patient session) │
                                     └──────────┬─────────────┘
                                                │
                          ┌─────────────────────┼─────────────────────┐
                          ▼                     ▼                     ▼
                  Form UI (review,        Patient blurb         Referral letter
                  edit, confirm)          generator              generator
                          │                     │                     │
                          ▼                     ▼                     ▼
                  Berry Studio sync      Copy / email / SMS     PDF / DOCX
```

---

## Phased plan

### Phase 0 — Discovery (1–2 days)

Goals: lock down scope before writing code.
- Finalize the full field schema (`docs/findings-inputs.md` → a JSON Schema).
- Pull every dropdown's actual options from Berry Studio (Chief Complaint, etc.).
- Decide on the **delivery surface** for filled findings (API push? browser
  automation? clipboard JSON? CSV export?).
- Decide on the **runtime surface** (web app on laptop in operatory? iPad?
  desktop Electron? phone?).
- Pick a tech stack (likely Next.js + TS, or a small FastAPI + React, or
  Electron — depends on where it runs).

**Deliverable:** `docs/schema.json` + `docs/decisions.md`.

### Phase 1 — Skeleton & schema (3–5 days)

- Repo scaffolding (frontend + backend).
- `findings.schema.json` — canonical schema with all enums, mm fields,
  tooth lists. Generate TS types from it.
- A static form UI that round-trips the schema (no voice yet) — every
  field renders, every value can be set, validation works.
- A "sample findings" fixture so we can develop downstream features
  without doing real consultations.

**Deliverable:** working form that produces valid JSON.

### Phase 2 — Voice in, structured out (1–2 weeks)

- Deepgram **streaming** integration (live mic) and **batch** (upload a
  recording) — both paths.
- Live transcript view (so you can see what was heard).
- Extraction pipeline: take rolling transcript → LLM call → JSON patch
  against current findings state. Two modes:
  1. **Continuous** — re-extract every ~10s; merge.
  2. **On-demand** — "extract now" button at end of consultation.
- Confidence per field (LLM reports `confidence` and `source_quote`).
  Low-confidence fields highlight for review.
- Test corpus: 10+ recorded mock consultations to evaluate.

**Deliverable:** speak a mock consult → form fills out → human reviews → saved.

### Phase 3 — Push to Berry Studio (depends on Phase 0 decision)

Three branches; pick the one that matches what's actually possible:

- **API path**: call Berry's API to write findings. Requires API docs &
  credentials. Cleanest.
- **Browser automation path**: Playwright drives the Berry UI. Robust to
  API absence, but fragile to UI changes.
- **Manual paste path**: produce a structured payload (JSON or
  Berry-formatted text) the user pastes / dictates back in. Lowest tech
  risk, highest friction.

**Deliverable:** end-to-end consult → Berry record updated.

### Phase 4 — Patient blurb generator

- Library of prompt templates (e.g. "warm & casual", "clinical-lite",
  "spanish translation", "kid-friendly", "post-op care notes").
- Template picker in the UI; user can also tweak the prompt inline before
  generating.
- Output formats: plain text (for SMS / chat) and short HTML (for email).
- Variables: pull patient name, key findings, recommended next step from
  the filled form so the blurb is grounded, not hallucinated.
- Save custom templates per practitioner.

**Deliverable:** click "Blurb" → pick style → editable draft → copy / send.

### Phase 5 — Letter generator

- Long-form templates (referral letter to specialist, treatment plan
  letter, insurance pre-auth narrative).
- Doc rendering: HTML → PDF (via headless Chromium) and / or DOCX.
- Practice letterhead, signature, NPI etc. configurable per user.
- Recipient field, address block, salutation handled by template.

**Deliverable:** click "Letter" → pick template → PDF / DOCX download.

### Phase 6 — Hardening

- Auth (who can use this and see what).
- Audit log (who edited what, when — important for clinical).
- PHI policy: encryption at rest, in transit, retention rules,
  BAAs with Deepgram + LLM provider.
- Error handling: what happens if mic dies mid-consult, network blips,
  LLM returns invalid JSON.
- Backup / export of all sessions.

---

## Risks & open issues

1. **PHI / HIPAA.** Deepgram has a BAA on Enterprise; the LLM provider
   needs one too. This blocks Phase 2 if not resolved before deploying
   beyond your single laptop.
2. **No Berry API yet known.** Phase 3 has three very different costs
   depending on this.
3. **Domain-specific transcription.** Dental terms (e.g. "Class III
   bilateral", "overjet two millimeters", "UL3 peg lateral", FDI
   numbering) are not in general-purpose ASR vocab. Deepgram supports
   **keyterm prompting** (Nova-3) — we'll need a dental keyterm list.
4. **Consultation mixes practitioner + patient + assistant voices.**
   Diarization needed; need to decide whether only the **practitioner's**
   utterances drive form-fill or everyone's.
5. **Tooth notation.** Conversational phrasing ("upper right three") vs.
   stored notation (FDI / Universal). Need a robust normalizer.
6. **Mobile vs. desktop.** Realtime mic + a form with this many fields is
   tight on a phone; iPad or laptop in operatory is more likely.

---

## What I need from you before I write code

See the questions in the next message — answering them locks scope.
