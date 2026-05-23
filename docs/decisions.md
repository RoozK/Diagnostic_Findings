# Locked decisions — v1 scope

Captured 2026-05-23. Anything not on this list is still open.

| Area               | Decision                                                                                                 |
| ------------------ | -------------------------------------------------------------------------------------------------------- |
| Runtime            | **Web app on clinic laptop**, browser-based.                                                             |
| Berry Studio sync  | **Manual paste-back.** App outputs JSON/text; clinician copies into Berry. No Berry API integration v1.  |
| Capture mode       | **Both** — live streaming preview during the consult, plus a batch re-extraction pass after.             |
| PHI posture        | **Single-clinic use with BAAs in place from day one.**                                                   |
| LLM provider       | **Anthropic Claude via AWS Bedrock** (uses the AWS BAA).                                                 |
| Speaker handling   | **Whole conversation as input.** No diarization-gated extraction in v1; LLM disambiguates.               |
| Patient context    | **Manual name + DOB entry** at session start. No Berry roster lookup in v1.                              |
| v1 output          | **Short patient-friendly summary only** (SMS-length, selectable tone). No letters / referrals in v1.     |
| Tech stack         | **Next.js + TypeScript + Tailwind**, single repo. Self-hosted on AWS (not Vercel) for BAA coverage.      |
| Data storage       | **AWS RDS Postgres** for structured findings + sessions. **S3** (with SSE + BAA) for audio + transcripts.|
| Next code step     | **None yet.** Docs committed; awaiting review before scaffolding.                                        |

## Implied follow-ups these decisions create

- **AWS account & BAA** — need to confirm the clinic AWS account has a signed BAA covering Bedrock, RDS, and S3 before any real PHI touches it.
- **Deepgram Enterprise plan + BAA** — required before any real consult audio is sent. Confirm tier covers streaming + diarization.
- **VPC / network design** — Bedrock and RDS should be in a private VPC; the Next.js server runs in that VPC. No direct browser→Bedrock calls.
- **Encryption** — TLS everywhere, RDS at-rest encryption, S3 SSE-KMS with a customer-managed key, transcripts not written to disk on the laptop.
- **Audit log requirement** — even in single-clinic v1, we should log who viewed/edited which session. Cheap to add early, expensive to retrofit.
- **Retention policy** — how long do we keep audio? Transcripts? Default proposal: audio = 30 days, transcripts = 1 year, findings = indefinite. Confirm with whoever owns clinic policy.
- **Backup / DR** — RDS automated backups + S3 versioning are table stakes. Cross-region replication is probably overkill for v1.

## Things we still need to decide (not blocking v1 scaffold)

1. **Chief Complaint** and other dropdown enums — pull from the live Berry app.
2. **Tooth notation** — pick FDI vs Universal vs Berry's own scheme as canonical.
3. **Validation rules** — required fields, mm ranges, decimal precision.
4. **Blurb tone library** — what presets ship in v1? Need ~3–5 starter prompts.
5. **Session lifecycle** — when does a session "close"? Auto after 24h? On explicit save?
6. **Multi-user prep** — v1 is single-clinic but probably more than one practitioner. Auth provider choice (Cognito? Auth0?) deferred but worth flagging.
