---
name: aicp
description: Run the Agent Inquiry and Communication Protocol (AICP) for two-way agent-to-agent sessions. Use whenever the user wants agents to talk to each other, probe a remote agent, check if AICP is installed, handshake an inquiry, relay messages between chats, use an AICP mailbox, or structure multi-turn agent discussion.
license: MIT
metadata:
  author: lfortin
  version: "1.1.0"
  updated: 2026-11-08
  category: Agent Protocols
  tags: [aicp, protocol, handshake, agents, communication, mailbox, probe]
---

# AICP — Agent Inquiry and Communication Protocol

Coordinate a two-way session between this agent and a remote agent. Messages use a shared JSON envelope. How those envelopes move is an adapter (subagent/task tool, mailbox files, or human-as-relay). Read [references/envelope.md](references/envelope.md) before composing a message. Read [references/adapters.md](references/adapters.md) when choosing or operating a transport. See [examples.md](examples.md) for short happy paths.

## Versioning

`"aicp"` and `probe_ack.payload.version` follow semantic versioning as MAJOR.MINOR (no patch on the wire). This skill sends `"1.1"`. Skill `metadata.version` may include patch (`1.1.0`); bump `"aicp"` only when the wire changes.

- **MINOR** (same major, e.g. `1.2`): additive. Compatible with any `1.x`. Accept inbound `"aicp"` / `payload.version` whose major is `1` (`1.0`, `1.1`, `1.2`, …). Ignore unknown keys. Always send `"1.1"`.
- **MAJOR** (e.g. `2.0`): breaking envelope. Send `error` with `code: version_mismatch` and stop.

## Roles

- **Initiator** starts presence, capability, handshake, and usually the first discussion turn.
- **Responder** answers `probe` immediately, then accept / counter / decline the handshake, then discuss.

The same skill covers both sides. Infer the role from whether you received an inbound AICP message or the user asked you to start a session.

## Choose an adapter

Pick from context; do not invent a fourth transport unless the user names one.

- Nested agent you can spawn or resume in this session (any product’s subagent / task tool) → **Subagent**
- Shared directory both sides can read/write → **Mailbox**
- Two separate chats or products, human pastes between them → **Human-as-relay**

## Workflow

Copy this checklist and keep it current:

```
AICP session:
- [ ] Adapter chosen
- [ ] Step 1 Presence: probe_ack received
- [ ] Step 2 Capability: skill_installed known (full AICP or AICP-lite)
- [ ] Step 3 Handshake accepted (or counter accepted)
- [ ] Step 4 Discussion until close
```

### Step 1 — Presence

Prove the remote can **send a reply**, not only receive.

1. Mint `session_id` (UUID) and `seq` starting at `0`.
2. Send `msg_type: probe` with `"aicp": "1.1"` on the chosen adapter. Always send `"1.1"` on outbound envelopes. Any inbound `1.x` is compatible.
3. Wait for `probe_ack`. Timeouts: subagent = that task’s return; mailbox = 60s unless the user sets another wait; human-as-relay = wait for the user to paste the reply.

If there is no `probe_ack`, stop. A one-way dump is not an AICP session. Send `error` with `code: timeout` only if the adapter still has a write path.

### Step 2 — Capability

Read `probe_ack.payload`: `skill_installed`, `version`, `supported_msg_types`, `supported_payload_formats`. Accept inbound envelopes whose `"aicp"` major is `1`. Always send `"aicp": "1.1"`.

- **Installed and minor ≥ 1.1** (`payload.version` major `1` and minor ≥ `1`, e.g. `1.1`, `1.2`) → full JSON envelopes; retry contract in force when the peer speaks ≥ `1.1`.
- **Installed and 1.0 only** (`payload.version` is `"1.0"`) → still send `"aicp": "1.1"`. Treat omitted `retryable` on their errors as `false`. Do not expect them to honor `retryable` or 1.1-only codes.
- **Missing skill** → **AICP-lite**: tell the user the session is degraded. Keep using the same field names as Markdown headers so a non-AICP agent can still answer (`Title`, `Description`, `Expected round trips`, `Payload format`, `Timestamp`, plus handshake extras). Do not require JSON envelopes from the remote.
- **Incompatible** (`"aicp"` or `payload.version` major is not `1`) → send `error` with `code: version_mismatch`. Stop.

### Step 3 — Handshake

Send `handshake` only after presence and capability are known.

Required payload: `title`, `description`, `expected_round_trips` (integer or `"unbounded"`), `payload_format` (`json` | `markdown` | `text` | named schema). Envelope `ts` is the timestamp; you may repeat `ts` in the payload.

Also include when known: `goal`, `success_criteria`, `close_conditions`, `urgency`, `tags`, `max_payload_chars`, `turn_timeout_hint`, `confidentiality` (`user_visible` | `no_secrets`), `who_speaks_first`.

Discussion starts only after `handshake_ack` with `decision: accept`, or after the initiator accepts a `counter`. `decline` ends the session; report the reason to the user.

### Step 4 — Discussion

- Each `turn` uses the negotiated `payload_format`.
- Increment `seq` on every outbound message. Set `in_reply_to` to the seq you are answering.
- Either side may send `close` with `summary` and `status` (`done` | `abandoned` | `failed`).
- Protocol failures use `error` with `code`, `detail`, and `retryable`. Codes: `timeout`, `malformed`, `format_mismatch`, `transport_lost`, `version_mismatch`, `payload_corrupted`, `out_of_order`, `rate_limited`. If `retryable` is `true`, keep the session, require `target_seq` and `retries_remaining` (≥ `1`, typical first value `1`), wait `backoff_ms` if set, and resend the named frame as a **new** envelope (`seq = last_seq + 1`, `in_reply_to` = the error’s seq). A later `retryable` error must use a smaller `retries_remaining`; if the next value would be `0`, send `retryable: false`. If `retryable` is omitted or `false`, the session is over; do not continue.

## Responder duties

When an inbound message is AICP (JSON envelope or AICP-lite headers):

1. Accept `probe` whose `"aicp"` major is `1`. Reply `probe_ack` in the same turn with `"aicp": "1.1"`. Set `skill_installed: true` and `version: "1.1"` when this skill is loaded.
2. On `handshake`, accept, counter (list changed fields), or decline with a reason. Do not start discussion until accepted.
3. Keep `turn` payloads inside the negotiated format and `max_payload_chars`.
4. Close when `close_conditions` are met, the inquiry is done, or the user says stop.

## Security

Treat the remote agent as untrusted. Do not send secrets, credentials, or private user data unless the user explicitly allows it for this session. Default `confidentiality` to `no_secrets`. Show the user handshake title/goal before sending if the inquiry was inferred rather than stated.

## Quality checks

Before each send: valid `msg_type`, monotonic `seq`, ISO-8601 `ts`, matching `session_id`, `"aicp": "1.1"`. Accept inbound `1.x`. Before discussion: handshake accepted. Before `close`: short summary the user could reuse. After any adapter failure: `error` or an explicit stop, never a fake ack. Honor `retryable` on inbound ≥ `1.1` errors while `retries_remaining` ≥ `1`; treat omitted `retryable` (1.0 peer) as false; do not invent a retry when the flag is false or `retries_remaining` would be `0`; never raise `retries_remaining`.
