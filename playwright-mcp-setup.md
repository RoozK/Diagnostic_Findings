# Setting up Playwright MCP for authenticated browser access

Goal: let Claude Code drive a real Chromium browser on your machine so it can
read pages on `app.berrystudio.ai` (or any other site you're logged into) and
extract their content into files in this repo.

This guide is for **local Claude Code** running on your own machine. The web /
cloud container is a poor fit for authenticated browsing because each session
starts from a fresh VM with no persistent browser profile.

---

## 1. Install Claude Code CLI locally

If you don't already have it:

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# or via npm
npm install -g @anthropic-ai/claude-code
```

Then sign in:

```bash
claude /login
```

Use the same claude.ai account you use on the web so your web sessions and
local sessions share auth.

Verify:

```bash
claude --version
```

## 2. Add Playwright MCP

Playwright MCP is Microsoft's official Model Context Protocol server for
driving Chromium / Firefox / WebKit. Add it at the user scope so it's
available across all your projects:

```bash
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
```

That runs `npx -y @playwright/mcp@latest` as a stdio MCP server. The first
time it runs it will download the Playwright browser binaries (~200 MB).

Confirm it registered:

```bash
claude mcp list
```

You should see `playwright` in the list.

## 3. Persist your Berry Studio login

By default, Playwright MCP launches a fresh browser context every session, so
you'd have to log in every time. Fix that by pointing it at a persistent
user-data directory.

Remove and re-add the server with extra args:

```bash
claude mcp remove playwright

mkdir -p ~/.playwright-mcp-profile

claude mcp add --scope user playwright -- \
  npx -y @playwright/mcp@latest \
  --user-data-dir ~/.playwright-mcp-profile \
  --browser chromium
```

The `--user-data-dir` flag persists cookies, localStorage, and IndexedDB
between sessions, just like a real Chrome profile.

> Treat `~/.playwright-mcp-profile` as sensitive — it contains your
> Berry Studio session cookies. Don't commit it, sync it, or share it.

## 4. First-run: log in to Berry Studio once

Start Claude Code in this repo:

```bash
cd /path/to/Diagnostic_Findings
claude
```

Then ask:

```
Open https://app.berrystudio.ai in the browser and wait for me to log in.
```

Playwright MCP will launch a Chromium window (non-headless by default with a
persistent profile). Log in normally. From now on the cookies are stored in
`~/.playwright-mcp-profile` and subsequent sessions will already be
authenticated.

## 5. Extract the findings-inputs page for redesign planning

The goal is a **structural map** of the page so you can plan v2 — not a copy
of patient 85's data. A target schema lives at
`docs/findings-inputs.template.md`; the prompt below tells Claude to fill it
in.

Once logged in, paste this prompt:

```
Use Playwright MCP. Navigate to
https://app.berrystudio.ai/practice/patient/view/85/care-timeline/findings-inputs
and wait until the page is fully loaded and interactive.

Fill in docs/findings-inputs.md following the exact schema in
docs/findings-inputs.template.md. Specifically:

1. Save a full-page screenshot to docs/screenshots/findings-inputs-full.png,
   plus one screenshot per major section.

2. For every form field, record: label, input type (text / number / date /
   select / multi-select / checkbox / radio / textarea / file / toggle),
   placeholder, helper text, required-indicator (yes/no), and any visible
   validation hints.

3. For every <select>/dropdown, click it to expose the options and list ALL
   of them. For radio and checkbox groups, list every choice.

4. Note any conditional fields — fields that appear or change based on a
   previous selection. Test a couple of representative selections to surface
   them, but do not submit or save anything.

5. Capture navigation context: breadcrumbs, tabs, side nav, parent route.
   Note where this page sits in the broader care-timeline flow.

6. List every button/action on the page (label + role: primary / secondary /
   destructive / link).

7. Do NOT include patient 85's actual values in the markdown. Replace
   pre-filled values with `<example: ...>`. This file will be committed.

8. If anything is unclear (collapsed section, behind a modal, gated by data
   you don't want to enter), note it as a TODO at the bottom rather than
   guessing.
```

Claude will use the `browser_navigate`, `browser_snapshot`, `browser_click`,
and `browser_take_screenshot` tools provided by Playwright MCP.

## 6. Optional: lock the browser version

If you want reproducible runs (same Chromium version on every machine), pin
the package version when adding the server:

```bash
claude mcp add --scope user playwright -- \
  npx -y @playwright/mcp@0.0.30 \
  --user-data-dir ~/.playwright-mcp-profile
```

Check the latest version at https://www.npmjs.com/package/@playwright/mcp.

---

## Troubleshooting

**"command not found: npx"** — install Node.js 20+ from https://nodejs.org or
via `nvm`. Playwright MCP needs npx on your PATH.

**Browser window doesn't open** — Playwright defaults to headless when there's
no display. To force a visible window, add `--no-headless` after the package
name in the `claude mcp add` command.

**"Server playwright failed to connect"** — run `claude mcp list` to check
status, and inspect logs with `claude --debug` for stderr from the MCP
process.

**Berry Studio logs me out between sessions** — confirm `--user-data-dir` was
actually included in the registered command with `claude mcp get playwright`.
If the path uses `~`, expand it: macOS / Linux shells may not expand `~`
inside the quoted command for some MCP transports. Use an absolute path like
`/Users/you/.playwright-mcp-profile`.

**OAuth / SSO redirects fail** — some identity providers block automated
browsers. If Berry Studio uses such a provider, complete the OAuth step in
your normal browser, then copy the resulting `app.berrystudio.ai` cookies
into the Playwright profile (Chromium DevTools → Application → Storage), or
use Playwright's `storageState` import via a separate seeding script.

---

## Why not the web / cloud session?

For reference, here's why the cloud container is awkward for this:

- **Ephemeral filesystem** — `.playwright-mcp-profile` resets every new
  session, so you'd log into Berry Studio on every run.
- **Restricted network** — the default Trusted allowlist doesn't include
  `app.berrystudio.ai`. You'd need to switch the environment to **Custom**
  network access (cloud icon → edit environment) and add the host plus any
  OAuth domains.
- **No display** — headless only, so any flow that requires solving a CAPTCHA
  or completing 2FA in-browser is harder.

If you ever do want it in the cloud, commit a project-scoped `.mcp.json` at
the repo root with:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--headless"]
    }
  }
}
```

…and handle auth by injecting a Berry Studio session cookie via environment
variable in your cloud environment settings.
