# Zero-Dependency Brainstorm Server

Replace brainstorm companion server vendored node_modules (express, ws, chokidar — 714 tracked files) with single zero-dep `server.js` using only Node.js built-ins.

## Motivation

Vendoring node_modules into git repo → supply chain risk: frozen deps miss security patches, 714 files third-party code committed without audit, mods to vendored code look like normal commits. Real risk low (localhost-only dev server) but eliminating cheap.

## Architecture

Single `server.js` (~250-300 lines) using `http`, `crypto`, `fs`, `path`. Two roles:

- **Run direct** (`node server.js`): start HTTP/WebSocket server
- **Required** (`require('./server.js')`): export WebSocket protocol fns for unit tests

### WebSocket Protocol

RFC 6455 text frames only:

**Handshake:** Compute `Sec-WebSocket-Accept` from client `Sec-WebSocket-Key` via SHA-1 + RFC 6455 magic GUID. Return 101 Switching Protocols.

**Frame decoding (client → server):** Three masked length encodings:
- Small: payload < 126 bytes
- Medium: 126-65535 bytes (16-bit extended)
- Large: > 65535 bytes (64-bit extended)

XOR-unmask payload via 4-byte mask key. Return `{ opcode, payload, bytesConsumed }` or `null` for incomplete buffers. Reject unmasked frames.

**Frame encoding (server → client):** Unmasked frames, same three length encodings.

**Opcodes handled:** TEXT (0x01), CLOSE (0x08), PING (0x09), PONG (0x0A). Unknown opcode → close frame status 1003 (Unsupported Data).

**Skipped:** Binary frames, fragmented messages, extensions (permessage-deflate), subprotocols. Unneeded for small JSON text between localhost clients. Extensions/subprotocols negotiated in handshake — not advertising → never active.

**Buffer accumulation:** Per-connection buffer. On `data`, append + loop `decodeFrame` til returns null or buffer empty.

### HTTP Server

Three routes:

1. **`GET /`** — Serve newest `.html` from screen dir by mtime. Detect full docs vs fragments, wrap fragments in frame template, inject helper.js. Return `text/html`. No `.html` files → serve hardcoded waiting page ("Waiting for Claude to push a screen...") with helper.js injected.
2. **`GET /files/*`** — Serve static files from screen dir, MIME type lookup from hardcoded extension map (html, css, js, png, jpg, gif, svg, json). 404 if missing.
3. **Else** — 404.

WebSocket upgrade via `'upgrade'` event on HTTP server, separate from request handler.

### Configuration

Env vars (all optional):

- `BRAINSTORM_PORT` — bind port (default: random high port 49152-65535)
- `BRAINSTORM_HOST` — bind interface (default: `127.0.0.1`)
- `BRAINSTORM_URL_HOST` — hostname for URL in startup JSON (default: `localhost` when host = `127.0.0.1`, else same as host)
- `BRAINSTORM_DIR` — screen dir path (default: `/tmp/brainstorm`)

### Startup Sequence

1. Create `SCREEN_DIR` if missing (`mkdirSync` recursive)
2. Load frame template + helper.js from `__dirname`
3. Start HTTP server on configured host/port
4. Start `fs.watch` on `SCREEN_DIR`
5. On listen success, log `server-started` JSON to stdout: `{ type, port, host, url_host, url, screen_dir }`
6. Write same JSON to `SCREEN_DIR/.server-info` so agents find conn details when stdout hidden (background exec)

### Application-Level WebSocket Messages

TEXT frame from client:

1. Parse as JSON. Parse fail → log stderr, continue.
2. Log stdout as `{ source: 'user-event', ...event }`.
3. Event has `choice` prop → append JSON to `SCREEN_DIR/.events` (one line per event).

### File Watching

`fs.watch(SCREEN_DIR)` replaces chokidar. HTML file events:

- New file (`rename` event for existing file): delete `.events` if present (`unlinkSync`), log `screen-added` stdout JSON
- File change (`change` event): log `screen-updated` stdout JSON (do NOT clear `.events`)
- Both events: send `{ type: 'reload' }` to all connected WebSocket clients

Debounce per-filename ~100ms timeout → prevent duplicate events (common macOS + Linux).

### Error Handling

- Malformed JSON from WebSocket clients: log stderr, continue
- Unhandled opcodes: close status 1003
- Client disconnect: remove from broadcast set
- `fs.watch` errors: log stderr, continue
- No graceful shutdown — shell scripts handle process lifecycle via SIGTERM

## What Changes

| Before | After |
|---|---|
| `index.js` + `package.json` + `package-lock.json` + 714 `node_modules` files | `server.js` (single file) |
| express, ws, chokidar dependencies | none |
| No static file serving | `/files/*` serves from screen directory |

## What Stays the Same

- `helper.js` — no change
- `frame-template.html` — no change
- `start-server.sh` — one-line update: `index.js` to `server.js`
- `stop-server.sh` — no change
- `visual-companion.md` — no change
- All existing server behavior + external contract

## Platform Compatibility

- `server.js` uses only cross-platform Node built-ins
- `fs.watch` reliable for single flat dirs on macOS, Linux, Windows
- Shell scripts need bash (Git Bash on Windows, required for Claude Code)

## Testing

**Unit tests** (`ws-protocol.test.js`): Test WebSocket frame encode/decode, handshake compute, protocol edge cases direct via `server.js` exports.

**Integration tests** (`server.test.js`): Test full server — HTTP serving, WebSocket comm, file watching, brainstorm workflow. Uses `ws` npm package as test-only client dep (not shipped to end users).