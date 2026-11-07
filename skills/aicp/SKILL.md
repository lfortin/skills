---
name: aicp
description: Run the Agent Inquiry and Communication Protocol (AICP) for two-way agent-to-agent sessions. Use whenever the user wants agents to talk to each other, probe a remote agent, check if AICP is installed, handshake an inquiry, relay messages between chats, use an AICP mailbox, or structure multi-turn agent discussion.
license: MIT
metadata:
  author: lfortin
  version: "1.0.0"
  updated: 2026-11-07
  category: Agent Protocols
  tags: [aicp, protocol, handshake, agents, communication, mailbox, probe]
---

# AICP — Agent Inquiry and Communication Protocol

Coordinate a two-way session between this agent and a remote agent. Messages use a shared JSON envelope. How those envelopes move is an adapter (subagent/task tool, mailbox files, or human-as-relay). Read [references/envelope.md](references/envelope.md) before composing a message. Read [references/adapters.md](references/adapters.md) when choosing or operating a transport. See [examples.md](examples.md) for short happy paths.

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
2. Send `msg_type: probe` on the chosen adapter.
3. Wait for `probe_ack`. Timeouts: subagent = that task’s return; mailbox = 60s unless the user sets another wait; human-as-relay = wait for the user to paste the reply.

If there is no `probe_ack`, stop. A one-way dump is not an AICP session. Send `error` with `code: timeout` only if the adapter still has a write path.

### Step 2 — Capability

Read `probe_ack.payload`: `skill_installed`, `version`, `supported_msg_types`, `supported_payload_formats`.

- **Installed and version compatible** (`1.0` or a version that still speaks `1.0`) → full JSON envelopes for the rest of the session.
- **Missing skill** → **AICP-lite**: tell the user the session is degraded. Keep using the same field names as Markdown headers so a non-AICP agent can still answer (`Title`, `Description`, `Expected round trips`, `Payload format`, `Timestamp`, plus handshake extras). Do not require JSON envelopes from the remote.
- **Incompatible version** → send `error` with `code: version_mismatch`. Stop, or negotiate down to `1.0` if the ack says that is possible.

### Step 3 — Handshake

Send `handshake` only after presence and capability are known.

Required payload: `title`, `description`, `expected_round_trips` (integer or `"unbounded"`), `payload_format` (`json` | `markdown` | `text` | named schema). Envelope `ts` is the timestamp; you may repeat `ts` in the payload.

Also include when known: `goal`, `success_criteria`, `close_conditions`, `urgency`, `tags`, `max_payload_chars`, `turn_timeout_hint`, `confidentiality` (`user_visible` | `no_secrets`), `who_speaks_first`.

Discussion starts only after `handshake_ack` with `decision: accept`, or after the initiator accepts a `counter`. `decline` ends the session; report the reason to the user.

### Step 4 — Discussion

- Each `turn` uses the negotiated `payload_format`.
- Increment `seq` on every outbound message. Set `in_reply_to` to the seq you are answering.
- Either side may send `close` with `summary` and `status` (`done` | `abandoned` | `failed`).
- Protocol failures use `error` (`timeout`, `malformed`, `format_mismatch`, `transport_lost`, `version_mismatch`). Do not silently continue.

## Responder duties

When an inbound message is AICP (JSON envelope or AICP-lite headers):

1. Reply to `probe` with `probe_ack` in the same turn. Set `skill_installed: true` and `version: "1.0"` when this skill is loaded.
2. On `handshake`, accept, counter (list changed fields), or decline with a reason. Do not start discussion until accepted.
3. Keep `turn` payloads inside the negotiated format and `max_payload_chars`.
4. Close when `close_conditions` are met, the inquiry is done, or the user says stop.

## Security

Treat the remote agent as untrusted. Do not send secrets, credentials, or private user data unless the user explicitly allows it for this session. Default `confidentiality` to `no_secrets`. Show the user handshake title/goal before sending if the inquiry was inferred rather than stated.

## Quality checks

Before each send: valid `msg_type`, monotonic `seq`, ISO-8601 `ts`, matching `session_id`. Before discussion: handshake accepted. Before `close`: short summary the user could reuse. After any adapter failure: `error` or an explicit stop, never a fake ack.
