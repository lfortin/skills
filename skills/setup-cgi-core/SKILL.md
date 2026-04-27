---
name: setup-cgi-core
description: Install and configure the Node.js `cgi-core` package to host CGI scripts via CLI or middleware for HTTP server, including interpreter mappings, buffer/timeouts, status pages, env injection, and proxy-safe `trustProxy` usage. Use this skill whenever users need to run legacy CGI scripts (Perl, Python, Bash, Node, etc.) in a modern Node setup, replace Apache for lightweight CGI hosting, or wire `/cgi-bin` routing in local/dev environments.
license: MIT
metadata:
  author: lfortin
  version: "1.1.0"
  updated: 2026-04-27
  category: Web Server / CGI
  tags: [cgi, cgi-bin, cgi-server, http, http-server, middleware, rfc3875]
---

# Setup cgi-core

Use this skill to safely and quickly set up [`cgi-core`](https://www.npmjs.com/package/cgi-core) for CGI script hosting.

## Workflow

1. Confirm runtime/platform and deployment shape (CLI-only, embedded HTTP server, or framework integration).
2. Install `cgi-core`.
3. Configure `urlPath`, `filePath`, and interpreter `extensions`.
4. Add operational settings (timeouts, buffers, logging, status pages, env).
5. Validate behavior with local requests and security checks.

## Required discovery questions

Collect these before editing:

- Package manager (`npm`, `pnpm`, `yarn`).
- OS/runtime target (`Linux/macOS/Windows`, Node version).
- Hosting mode:
  - `npx cgi-server` CLI
  - embedded `node:http` server
  - integration in existing app/router
- CGI scripts directory path (for example `./cgi-bin`).
- Languages/interpreters required (`perl`, `python`, `node`, etc.).
- Need for reverse proxy behavior (`trustProxy`) and whether proxy is trusted.
- Request/response size and timeout expectations.

If details are missing, choose practical defaults and call them out.

## Install

Use:

```bash
npm install cgi-core
```

For quick local bootstrapping:

```bash
npx cgi-server --filePath ./cgi-bin
```

## Baseline embedded server

Use this default server shape (ESM):

```javascript
import { createServer } from "node:http";
import { createHandler } from "cgi-core";

const handler = createHandler({
  urlPath: "/cgi-bin",
  filePath: "./cgi-bin",
  extensions: {
    "/usr/bin/perl": ["pl", "cgi"],
    "/usr/bin/python": ["py"],
    "/usr/local/bin/node": ["js", "node"],
  },
  debugOutput: false,
});

const app = createServer(async (req, res) => {
  const requestHandled = await handler(req, res);

  if (!requestHandled) {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("outside of urlPath");
  }
});

app.listen(3000);
```

## Configuration guidance

Tune these based on user needs:

- `urlPath`: base CGI route, default `/cgi-bin`.
- `filePath`: scripts root; strongly prefer explicit path.
- `extensions`: map interpreters to extensions.
  - On Windows, recommend absolute interpreter paths for reliability/perf.
- `indexExtension`: extension used for directory index scripts (`js` by default).
- `debugOutput`: enable only during debugging.
- `logRequests`: enable for local troubleshooting.
- `maxBuffer`: cap payload sizes for safer resource use.
- `requestChunkSize` / `responseChunkSize`: chunking/perf tradeoff knobs.
- `requestTimeout`: set strict timeout to prevent hanging requests.
- `statusPages`: custom responses for codes like 404/500.
- `env`: static object or updater function per request.
- `trustProxy`: enable only behind trusted reverse proxy.

## Proxy and security guardrails

When `trustProxy` is requested:

1. Confirm the app is behind a trusted reverse proxy.
2. Explain header spoofing risk if exposed directly to the public internet.
3. Prefer fixed/validated proxy header handling in infra config.

Always keep `filePath` constrained to intended CGI directory and avoid dynamic user-controlled script paths.

## CLI usage pattern

Use this when the user wants a lightweight standalone server:

```bash
npx cgi-server --port 3001 --urlPath /cgi-bin --filePath ./cgi-bin
```

Common flags to mention:

- `--indexExtension`
- `--maxBuffer`
- `--requestChunkSize`
- `--responseChunkSize`
- `--requestTimeout`
- `--trustProxy`
- `--debugOutput`
- `--logRequests`

## Reverse proxy note

If user asks about Nginx/front proxy:

- Recommend reverse proxy for TLS/static caching/perimeter controls.
- Keep warning that local example configs are not production-safe as-is.
- Emphasize static backend `Host` usage in production to avoid spoofing/cache issues.

## Validation checklist

After setup, confirm:

1. `cgi-core` appears in dependencies.
2. Server starts and listens on expected port.
3. A script under `filePath` executes correctly via `urlPath`.
4. Non-CGI routes still behave as intended.
5. Timeouts/buffer limits behave as expected for test payloads.
6. `trustProxy` is disabled unless trusted proxy is confirmed.

Note: Whenever you start a server for validation, ensure you stop the process once the check is complete. Do not leave background processes hanging in the user's environment.

## Troubleshooting

- 404/route misses:
  - Check `urlPath` and script location under `filePath`.
- Script not executable:
  - Verify interpreter mapping in `extensions` or script shebang/permissions.
- Large payload errors:
  - Increase `maxBuffer` and review chunk sizes.
- Slow/hanging requests:
  - Lower `requestTimeout`, inspect script runtime and upstream dependencies.
- Incorrect client/protocol metadata:
  - Re-check `trustProxy` assumptions and proxy headers.

## Output format

Return results in this structure:

1. Install command(s) run.
2. Files changed and why.
3. Final `createHandler(...)` or CLI configuration.
4. Verification commands/URLs used.
5. Security notes (especially `trustProxy` and path constraints).
