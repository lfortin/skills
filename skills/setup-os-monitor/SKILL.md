---
name: setup-os-monitor
description: Install and configure the Node.js `os-monitor` package, including legacy-version fallback, threshold tuning, event handlers, stream mode, and validation. Use this skill whenever the user asks to set up OS/system monitoring in Node.js, wire alerts for memory/load/disk/uptime, or integrate `os-monitor` into an app, even if they do not explicitly mention the package name.
license: MIT
metadata:
  author: lfortin
  version: "1.2.0"
  updated: 2026-04-27
  category: Monitoring
  tags: [monitor, monitoring, os, system, loadavg, freemem, memory, uptime, disk, df, diskfree]
---

# Setup os-monitor

Use this skill to reliably install and configure [`os-monitor`](https://www.npmjs.com/package/os-monitor) in a Node.js project.

## What to do

1. Confirm runtime and package-manager context.
2. Install the correct package version (`latest` vs `legacy`).
3. Add a minimal monitor setup.
4. Add production-oriented config (thresholds, events, throttling, stream mode when needed).
5. Verify behavior with a quick run and explain tuning knobs.

## Inputs to gather first

Ask for missing details before changing files:

- Project package manager (`npm`, `pnpm`, or `yarn`).
- Node.js version in the target environment.
- Whether the user needs `CommonJS` (`require`) or `ESM` (`import`) style.
- Which conditions should trigger alerts:
  - low free memory
  - high load average (1/5/15 min)
  - low disk space on specific paths
  - uptime threshold
- Desired monitor interval (`delay`).
- Whether output should be event callbacks only or readable stream mode.

If the user does not provide preferences, choose sensible defaults and call them out clearly.

## Installation logic

Use this decision tree:

- If Node.js is `>= 18.15.x`, install:
  - `npm install os-monitor --ignore-scripts`
- If Node.js is `< 18.15.x` or explicitly very old, install:
  - `npm install os-monitor@legacy --ignore-scripts`

When in doubt, ask the user to confirm target Node version before proceeding.

## Baseline implementation

Start from this baseline (CommonJS):

```javascript
const { createMonitor } = require("os-monitor");

const monitor = createMonitor();

monitor.on("monitor", (event) => {
  console.log("[monitor]", event);
});

monitor.on("freemem", (event) => {
  console.warn("[freemem]", event.freemem, "bytes free");
});

monitor.on("loadavg1", (event) => {
  console.warn("[loadavg1]", event.loadavg[0]);
});

monitor.start({
  delay: 3000,
  freemem: 0, // replace with threshold later
  critical1: require("os").cpus().length,
  critical5: require("os").cpus().length,
  critical15: require("os").cpus().length,
  silent: false,
  stream: false,
  immediate: false,
});
```

For ESM, adapt imports and keep behavior equivalent.

## Configuration guidance

Use these rules while tuning:

- `delay`: default `3000` ms; increase for lower overhead.
- `freemem`: allow bytes or fraction of total memory (for example `0.2` for 20%).
- `critical1`, `critical5`, `critical15`: default to `os.cpus().length`; lower values make alerts more sensitive.
- `diskfree`: only use where `fs.statfs` support is available (`Node >= 18.15.x`).
- `silent: true` disables recurring `monitor` events when logs are too noisy.
- `stream: true` when piping output to stdout/files/processors.
- `immediate: true` when an initial sample is needed right after `start()`.

If throttling is useful, add:

```javascript
monitor.throttle("loadavg5", (event) => {
  console.warn("[loadavg5 throttled]", event.loadavg[1]);
}, monitor.minutes(5));
```

## Disk monitoring pattern

When user asks for disk alerts, convert byte thresholds to blocks:

```javascript
monitor.start({
  diskfree: {
    "/": monitor.blocks(100000000, 4096),
  },
});
```

Explain that block size depends on the filesystem and should match the environment.

## Validation checklist

After setup, verify all of the following:

1. Package appears in `dependencies`.
2. Monitor starts without runtime errors.
3. At least one event handler logs expected payload structure.
4. `monitor.isRunning()` reports `true` after start and `false` after stop.
5. If stream mode is enabled, output is piped and visible.
6. If `diskfree` was configured, user confirms runtime supports `statfs`.

Note: Whenever you start a server for validation, ensure you stop the process once the check is complete. Do not leave background processes hanging in the user's environment.

## Troubleshooting

- No events firing:
  - Check thresholds are realistic and not too strict.
  - Enable `immediate: true` for instant first cycle.
- Too many logs:
  - Increase `delay`.
  - Set `silent: true` and keep only targeted event handlers.
  - Add throttled handlers for noisy events.
- Diskfree not working:
  - Confirm Node version is new enough for `fs.statfs`.
  - Re-check block threshold conversion.

## Output format

When completing a setup task, return:

1. What you installed and why (`latest` vs `legacy`).
2. Files changed.
3. Final monitor config used.
4. How to run and validate.
5. Suggested next tuning steps for production workloads.
