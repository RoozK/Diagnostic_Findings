# Cursor prompt — V1 (record → transcribe → blurb)

Paste everything below the line into Cursor. Make sure Cursor has this
repo open so it can read `docs/findings-inputs.md` for context if needed
(not required for V1, but useful later).

---

Build a single-page local web app that records a dental consultation, transcribes it, and generates a short patient-friendly blurb I can send.

## Stack
- Next.js 15 (App Router) + TypeScript + Tailwind CSS
- No database, no auth, runs on localhost
- API keys in .env.local

## Flow
1. Page shows a big Record button.
2. Click Record → browser captures mic audio via MediaRecorder. Show elapsed time + a Stop button.
3. Click Stop → POST the audio blob to /api/transcribe, which calls Deepgram's pre-recorded transcription endpoint (model: nova-3). Show the returned transcript in a panel I can collapse/edit.
4. Below the transcript, show a row of style buttons (the "blurb styles" below).
5. Click a style → POST { transcript, style } to /api/blurb, which calls the Anthropic Messages API (model: claude-sonnet-4-6) with a system prompt that turns the transcript into a short patient-facing summary in that style. Stream the response into a text area below the buttons.
6. The text area is editable. A "Copy" button puts the final blurb on the clipboard.
7. I can click a different style at any time to regenerate from the same transcript.

## Blurb styles (each is a different system prompt; aim for 2–4 sentences)
- **Warm & friendly** — conversational, reassuring, plain language.
- **Clinical-lite** — accurate but accessible, like a doctor's after-visit summary.
- **Kid-friendly** — short sentences, simple words, optimistic tone.
- **Spanish (warm)** — same warm tone, written in Spanish.

Hard-code these in `lib/blurb-styles.ts`. Each entry has a `label`, a `systemPrompt`, and an optional `language`.

## File layout
- `app/page.tsx` — single page: Recorder, Transcript panel, Style buttons, Blurb textarea, Copy button
- `app/api/transcribe/route.ts` — POST audio → Deepgram → transcript
- `app/api/blurb/route.ts` — POST { transcript, style } → Anthropic → blurb (stream response)
- `components/Recorder.tsx` — MediaRecorder UI
- `lib/blurb-styles.ts` — the style presets
- `.env.example` — `DEEPGRAM_API_KEY=`, `ANTHROPIC_API_KEY=`
- `README.md` — how to run

## Don't add
- No login, no database, no patient identity capture
- No findings form, no structured extraction
- No live/streaming transcription — record-then-process is fine
- No AWS/Bedrock — direct API calls to Deepgram and Anthropic
- No design system — plain Tailwind, default fonts, generous whitespace

## Acceptance
- `npm install && npm run dev` starts on http://localhost:3000
- I can record speech, see a transcript, click a style, and get a 2–4 sentence blurb
- The blurb is editable and copyable
- Switching styles regenerates from the same transcript without re-recording
- Missing API keys produce a clear in-UI error, not a crash
