# Grok Bot task-ingress probe (2026-10-06, Windows host, read-only)

Target: Grok Bot desktop app 0.66.0 at `C:\Program Files\Grok Bot\`
(`Grok Bot.exe` FileVersion 0.66.0, Electron, `resources/app.asar`).

## Verdict

**Task ingress EXISTS via the `grok` CLI (`@xai-official/grok` 1.0.13,
on PATH at `%APPDATA%\npm\grok`). No other programmatic surface exists.**
Computer-use/GUI automation is NOT needed.

## 1. `grok` CLI — real headless ingress (recommended route)

- Binary: `grok 1.0.13 (5e9a58528b76)`, `grok --help` exits 0 non-interactively.
- Headless task forms:
  - `grok -p "PROMPT"` — single-turn, prints response to stdout and exits
    (plus `--output-format json|streaming-json`, `--model`, `--cwd`,
    `--permission-mode`, `--max-turns`, `--prompt-file`, `--session-id`).
  - `grok agent {stdio|headless|serve|leader}` — programmatic agent
    (ACP over stdio, headless over Grok WebSocket relay, or WS server).
- Auth state: signed in (`~/.grok/auth.json` present).
- Live demo (minimal, one short echo prompt):
  `grok -p "Reply with exactly: GROKBOT-PROBE-OK"` was accepted by the CLI
  and reached the API, which answered:
  `API error (status 402 Payment Required): Grok Build usage balance exhausted`.
  Path proven end-to-end (CLI -> auth -> API); only billing blocks execution.
  **Orchestrator implication: fund/attribute usage, then route Bots-pool
  tasks via `grok -p` / `grok agent`.**

## 2. `grokbot://` URL protocol — NOT task ingress

- Registered: `HKCU\Software\Classes\grokbot` (`URL Protocol`,
  `URL:grokbot`) -> `"C:\Program Files\Grok Bot\Grok Bot.exe" "%1"`.
- asar evidence: the only concrete URL shapes are
  `grokbot://mcp/oauth/callback` (MCP OAuth return) and template-import
  links (`grokbot://app/...` with path prefixes `/grok-bot/link`,
  `/sand/link`, resolved via server `path:"/v1/bot-template"` —
  "Import Template: Paste template link or share ID", i.e. shared **bot
  configs**, not task prompts). No prompt/task payload action exists.

## 3. Local HTTP API — none

- `netstat`: no listener owned by Grok Bot (app was not running; nothing on
  app-specific ports).
- asar `localhost:*` strings (`:4000 :4443 :8000 :8787 :8969 :9999`) are
  OAuth loopback callbacks (`http://localhost:8787/callback`,
  `x-cursor-mcp-oauth-relayed-loopback`, Cursor-derived) and internal
  stubs — no serving socket, no REST/RPC ingress.

## 4. Misc (negative checks)

- `Grok Bot.exe --help` prints nothing and just launches the GUI
  (no CLI surface on the Electron binary).
- No `grokbot`/`grok` CLI on PATH besides the npm `grok` shim above;
  no App Paths entry, no uninstall-registry version key (version taken
  from exe metadata), desktop shortcut targets the exe with no args.
- App state dir: `%APPDATA%\Grok Bot` (Electron profile: Preferences,
  Local State, Session Storage, etc.). Untouched.

## Method

Read-only: registry queries, `netstat`, `tasklist`, `rg -a` string scans
of `app.asar` (never extracted/modified), `grok --help` /
`grok agent --help`, one minimal `grok -p` call. A stray GUI launch from
the `--help` probe was killed; app left intact.
