# AICP envelope (version 1.1)

Every full-AICP message is one JSON object. AICP-lite uses the same field names as Markdown headers instead of this envelope.

## Common fields

```json
{
  "aicp": "1.1",
  "msg_type": "probe",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 0,
  "ts": "2026-11-07T14:43:00Z",
  "from": { "role": "initiator", "agent_id": "optional-stable-name" },
  "to": { "role": "responder" },
  "in_reply_to": null,
  "payload": {}
}
```

- `aicp`: wire dialect (semantic versioning, MAJOR.MINOR; no patch). This document is `"1.1"`. Always send `"1.1"`. Accept inbound any `1.x`. Major ≠ `1` → `error` `version_mismatch`. A 1.0 inbound `error` may omit `retryable`; treat that as `false`. Do not expect a 1.0 peer to honor `retryable`. Ignore unknown keys from a later `1.x`.
- `msg_type`: `probe` | `probe_ack` | `handshake` | `handshake_ack` | `turn` | `close` | `error`
- `session_id`: UUID shared for the whole session
- `seq`: integer, starts at `0`, increments by 1 on every outbound message from that sender’s stream (treat the session as one combined sequence: each new message gets `last_seq + 1`)
- `ts`: ISO-8601 timestamp
- `from` / `to`: `role` is `initiator` or `responder`; `agent_id` is optional
- `in_reply_to`: seq of the message being answered, or omit/`null` on the first `probe`
- `payload`: object shaped by `msg_type`

## probe

```json
{
  "aicp": "1.1",
  "msg_type": "probe",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 0,
  "ts": "2026-11-07T14:43:00Z",
  "from": { "role": "initiator" },
  "to": { "role": "responder" },
  "payload": {
    "intent": "aicp_presence"
  }
}
```

## probe_ack

Always send `"aicp": "1.1"`. Advertise this skill with `version: "1.1"`.

```json
{
  "aicp": "1.1",
  "msg_type": "probe_ack",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 1,
  "ts": "2026-11-07T14:43:05Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 0,
  "payload": {
    "skill_installed": true,
    "version": "1.1",
    "supported_msg_types": [
      "probe",
      "probe_ack",
      "handshake",
      "handshake_ack",
      "turn",
      "close",
      "error"
    ],
    "supported_payload_formats": ["json", "markdown", "text"]
  }
}
```

If the remote is present but does not have AICP, still send this shape when possible, with `skill_installed: false`. Otherwise fall back to AICP-lite (see SKILL.md).

## handshake

```json
{
  "aicp": "1.1",
  "msg_type": "handshake",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 2,
  "ts": "2026-11-07T14:43:20Z",
  "from": { "role": "initiator" },
  "to": { "role": "responder" },
  "in_reply_to": 1,
  "payload": {
    "title": "Review auth rate-limit design",
    "description": "Need a second agent to stress-test the proposed token-bucket rules.",
    "expected_round_trips": 10,
    "payload_format": "markdown",
    "ts": "2026-11-07T14:43:20Z",
    "goal": "Agree on limits and failure modes, then list open questions.",
    "success_criteria": "Shared list of limits plus at least three failure modes.",
    "close_conditions": "Criteria met, or either side sends close.",
    "urgency": "normal",
    "tags": ["auth", "rate-limit"],
    "max_payload_chars": 8000,
    "turn_timeout_hint": "5m",
    "confidentiality": "no_secrets",
    "who_speaks_first": "initiator"
  }
}
```

`expected_round_trips` is an integer or `"unbounded"`. `payload_format` is `json`, `markdown`, `text`, or a named schema string both sides understand. `confidentiality` is `user_visible` or `no_secrets`.

## handshake_ack

```json
{
  "aicp": "1.1",
  "msg_type": "handshake_ack",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 3,
  "ts": "2026-11-07T14:43:30Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 2,
  "payload": {
    "decision": "accept",
    "reason": null,
    "changed_fields": null
  }
}
```

`decision` is `accept` | `counter` | `decline`. On `counter`, put the replacement values in `changed_fields` (object of payload keys). On `decline`, set `reason`. Discussion must not start until `accept`, or until the initiator sends a follow-up `handshake_ack`/`turn` that accepts the counter (initiator may send another `handshake` with the countered fields, or a `handshake_ack` with `decision: accept` echoing the countered payload).

## turn

```json
{
  "aicp": "1.1",
  "msg_type": "turn",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 4,
  "ts": "2026-11-07T14:44:00Z",
  "from": { "role": "initiator" },
  "to": { "role": "responder" },
  "in_reply_to": 3,
  "payload": {
    "format": "markdown",
    "body": "## Proposed limits\n\n- 100 req/min per API key\n- burst 20\n"
  }
}
```

For `payload_format: json`, use `body` as an object. For `text` or `markdown`, `body` is a string. `format` must match the negotiated `payload_format`.

## close

```json
{
  "aicp": "1.1",
  "msg_type": "close",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 7,
  "ts": "2026-11-07T14:50:00Z",
  "from": { "role": "initiator" },
  "to": { "role": "responder" },
  "in_reply_to": 6,
  "payload": {
    "status": "done",
    "summary": "Agreed 100/min with burst 20; remaining question is lockout after 5 bursts."
  }
}
```

`status` is `done` | `abandoned` | `failed`. Examples in this file are per `msg_type`; seq numbers are not one continuous session.

## error

Every `error` payload requires `code`, `detail`, and `retryable`. Omit `retryable` only if you mean `false`.

`code` is `timeout` | `malformed` | `format_mismatch` | `transport_lost` | `version_mismatch` | `payload_corrupted` | `out_of_order` | `rate_limited`.

- `malformed`: envelope is not valid JSON or required envelope fields are missing.
- `payload_corrupted`: envelope parsed, but `payload` / `body` is unreadable or fails the negotiated schema.
- `out_of_order`: sequence drift (`seq` not `last_seq + 1`, or `in_reply_to` does not name a known frame).
- `format_mismatch`: `turn` `format` / `body` type does not match negotiated `payload_format`.
- Adapter I/O retries (mailbox lock, spawn retry) are not this frame. `retryable` means “resend an AICP message,” not “retry the write.”

Typical `retryable: true`: `payload_corrupted`, `timeout`, `out_of_order`, `rate_limited`. Typical `retryable: false`: `malformed`, `format_mismatch`, `version_mismatch`, `transport_lost`. The sender of the error sets the flag; a ≥ `1.1` peer must honor it while `retries_remaining` allows.

If `retryable` is `true`: keep session state. `target_seq` is required (seq of the **payload to resend**, not this error). `retries_remaining` is required: integer ≥ `1`, the number of retransmits still allowed **including** the one this error requests. Typical first value is `1`. `backoff_ms` is optional. The peer waits `backoff_ms` if set, then resends that content in a **new** envelope: `seq = last_seq + 1`, `in_reply_to` = this error’s seq. Do not reuse the original seq.

If the retransmit is still bad, the error sender may send another `retryable: true` only with a **strictly smaller** `retries_remaining` (previous minus 1). Never raise the budget. If the next value would be `0`, send `retryable: false` instead. A missing `retries_remaining` on `retryable: true` is treated as `1`.

If `retryable` is omitted or `false`: the error is terminal; destroy session state and stop (send `close` only if the adapter still has a write path). A 1.0 peer will omit `retryable` and `retries_remaining`; treat that as `false`.

Non-retryable example (bad turn at seq 4):

```json
{
  "aicp": "1.1",
  "msg_type": "error",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 5,
  "ts": "2026-11-07T14:45:00Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 4,
  "payload": {
    "code": "format_mismatch",
    "detail": "Negotiated markdown; received json body.",
    "retryable": false
  }
}
```

### Retransmission (`retryable: true`)

Turn seq 4 is unreadable → error seq 5 asks to resend that turn → peer resends as seq 6.

```json
{
  "aicp": "1.1",
  "msg_type": "error",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 5,
  "ts": "2026-11-07T14:45:00Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 4,
  "payload": {
    "code": "payload_corrupted",
    "detail": "JSON body failed schema validation.",
    "retryable": true,
    "target_seq": 4,
    "retries_remaining": 1,
    "backoff_ms": 1000
  }
}
```

Retransmitted turn:

```json
{
  "aicp": "1.1",
  "msg_type": "turn",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 6,
  "ts": "2026-11-07T14:45:02Z",
  "from": { "role": "initiator" },
  "to": { "role": "responder" },
  "in_reply_to": 5,
  "payload": {
    "format": "markdown",
    "body": "## Proposed limits\n\n- 100 req/min per API key\n- burst 20\n"
  }
}
```
