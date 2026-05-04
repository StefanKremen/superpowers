# Visual Brainstorming Refactor: Browser Displays, Terminal Commands

**Date:** 2026-02-19
**Status:** Approved
**Scope:** `lib/brainstorm-server/`, `skills/brainstorming/visual-companion.md`, `tests/brainstorm-server/`

## Problem

During visual brainstorming, Claude run `wait-for-feedback.sh` as background task, block on `TaskOutput(block=true, timeout=600s)`. Seize TUI entirely — user can't type to Claude while visual brainstorming run. Browser become only input channel.

Claude Code execution model turn-based. No way for Claude to listen on two channels simultaneously in single turn. Blocking `TaskOutput` pattern wrong primitive — simulate event-driven behavior platform don't support.

## Design

### Core Model

**Browser = interactive display.** Show mockups, let user click to select options. Selections recorded server-side.

**Terminal = conversation channel.** Always unblocked, always available. User talk to Claude here.

### The Loop

1. Claude write HTML file to session dir
2. Server detect via chokidar, push WebSocket reload to browser (unchanged)
3. Claude end turn — tell user check browser, respond in terminal
4. User look at browser, optionally click to select option, then type feedback in terminal
5. Next turn, Claude read `$SCREEN_DIR/.events` for browser interaction stream (clicks, selections), merge with terminal text
6. Iterate or advance

No background tasks. No `TaskOutput` blocking. No polling scripts.

### Key Deletion: `wait-for-feedback.sh`

Deleted entirely. Purpose was bridge "server log events to stdout" and "Claude need receive those events." `.events` file replace this — server write user interaction events directly, Claude read with whatever file-reading mechanism platform provide.

### Key Addition: `.events` File (Per-Screen Event Stream)

Server write all user interaction events to `$SCREEN_DIR/.events`, one JSON object per line. Give Claude full interaction stream for current screen — not just final selection, but user exploration path (clicked A, then B, settled on C).

Example contents after user explore options:

```jsonl
{"type":"click","choice":"a","text":"Option A - Preset-First Wizard","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Manual Config","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid Approach","timestamp":1706000115}
```

- Append-only within screen. Each user event appended as new line.
- File cleared (deleted) when chokidar detect new HTML file (new screen pushed) → prevent stale events carrying over.
- File doesn't exist when Claude read → no browser interaction → Claude use only terminal text.
- File contain only user events (`click`, etc.) — not server lifecycle events (`server-started`, `screen-added`). Keep small + focused.
- Claude read full stream to understand exploration pattern, or look at last `choice` event for final selection.

## Changes by File

### `index.js` (server)

**A. Write user events to `.events` file.**

In WebSocket `message` handler, after logging event to stdout: append event as JSON line to `$SCREEN_DIR/.events` via `fs.appendFileSync`. Only write user interaction events (those with `source: 'user-event'`), not server lifecycle events.

**B. Clear `.events` on new screen.**

In chokidar `add` handler (new `.html` file detected), delete `$SCREEN_DIR/.events` if exist. Definitive "new screen" signal — better than clearing on GET `/` which fire every reload.

**C. Replace `wrapInFrame` content injection.**

Current regex anchor on `<div class="feedback-footer">` — being removed. Replace with comment placeholder: remove existing default content inside `#claude-content` (`<h2>Visual Brainstorming</h2>` and subtitle paragraph), replace with single `<!-- CONTENT -->` marker. Content injection become `frameTemplate.replace('<!-- CONTENT -->', content)`. Simpler, won't break if template formatting change.

### `frame-template.html` (UI frame)

**Remove:**
- `feedback-footer` div (textarea, Send button, label, `.feedback-row`)
- Associated CSS (`.feedback-footer`, `.feedback-footer label`, `.feedback-row`, textarea + button styles within)

**Add:**
- `<!-- CONTENT -->` placeholder inside `#claude-content`, replacing default text
- Selection indicator bar where footer was, two states:
  - Default: "Click an option above, then return to the terminal"
  - After selection: "Option B selected — return to terminal to continue"
- CSS for indicator bar (subtle, similar visual weight to existing header)

**Keep unchanged:**
- Header bar with "Brainstorm Companion" title + connection status
- `.main` wrapper + `#claude-content` container
- All component CSS (`.options`, `.cards`, `.mockup`, `.split`, `.pros-cons`, placeholders, mock elements)
- Dark/light theme vars + media query

### `helper.js` (client-side script)

**Remove:**
- `sendToClaude()` fn + "Sent to Claude" page takeover
- `window.send()` fn (tied to removed Send button)
- Form submission handler — no purpose without feedback textarea, add log noise
- Input change handler — same reason
- `pageshow` event listener (added to fix textarea persistence — no textarea now)

**Keep:**
- WebSocket connection, reconnect logic, event queue
- Reload handler (`window.location.reload()` on server push)
- `window.toggleSelect()` for selection highlighting
- `window.selectedChoice` tracking
- `window.brainstorm.send()` + `window.brainstorm.choice()` — distinct from removed `window.send()`. Call `sendEvent` which log to server via WebSocket. Useful for custom full-document pages.

**Narrow:**
- Click handler: capture only `[data-choice]` clicks, not all buttons/links. Broad capture needed when browser was feedback channel; now just for selection tracking.

**Add:**
- On `data-choice` click, update selection indicator bar text → show which option selected.

**Remove from `window.brainstorm` API:**
- `brainstorm.sendToClaude` — no longer exist

### `visual-companion.md` (skill instructions)

**Rewrite "The Loop" section** → non-blocking flow above. Remove all refs to:
- `wait-for-feedback.sh`
- `TaskOutput` blocking
- Timeout/retry logic (600s timeout, 30-min cap)
- "User Feedback Format" section describing `send-to-claude` JSON

**Replace with:**
- New loop (write HTML → end turn → user respond in terminal → read `.events` → iterate)
- `.events` file format docs
- Guidance: terminal message = primary feedback; `.events` provide full browser interaction stream for additional context

**Keep:**
- Server startup/shutdown instructions
- Content fragment vs full document guidance
- CSS class reference + available components
- Design tips (scale fidelity to question, 2-4 options per screen, etc.)

### `wait-for-feedback.sh`

**Deleted entirely.**

### `tests/brainstorm-server/server.test.js`

Tests need updating:
- Test asserting `feedback-footer` presence in fragment responses → update to assert selection indicator bar or `<!-- CONTENT -->` replacement
- Test asserting `helper.js` contain `send` → update to reflect narrowed API
- Test asserting `sendToClaude` CSS variable usage → remove (fn no longer exist)

## Platform Compatibility

Server code (`index.js`, `helper.js`, `frame-template.html`) fully platform-agnostic — pure Node.js + browser JS. No Claude Code-specific refs. Already proven on Codex via background terminal interaction.

Skill instructions (`visual-companion.md`) = platform-adaptive layer. Each platform's Claude use own tools to start server, read `.events`, etc. Non-blocking model work naturally across platforms — don't depend on platform-specific blocking primitive.

## What This Enables

- **TUI always responsive** during visual brainstorming
- **Mixed input** — click in browser + type in terminal, naturally merged
- **Graceful degradation** — browser down or user don't open? Terminal still works
- **Simpler architecture** — no background tasks, no polling scripts, no timeout management
- **Cross-platform** — same server code work on Claude Code, Codex, any future platform

## What This Drops

- **Pure-browser feedback workflow** — user must return to terminal to continue. Selection indicator bar guide them, but one extra step vs old click-Send-and-wait flow.
- **Inline text feedback from browser** — textarea gone. All text feedback through terminal. Intentional — terminal better text input channel than small textarea in frame.
- **Immediate response on browser Send** — old system had Claude respond moment user clicked Send. Now gap while user switch to terminal. In practice = seconds, user get to add context in terminal message.