# AICP envelope (version 1.0)

Every full-AICP message is one JSON object. AICP-lite uses the same field names as Markdown headers instead of this envelope.

## Common fields

```json
{
  "aicp": "1.0",
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

- `aicp`: protocol version string. This document is `"1.0"`.
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
  "aicp": "1.0",
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

```json
{
  "aicp": "1.0",
  "msg_type": "probe_ack",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 1,
  "ts": "2026-11-07T14:43:05Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 0,
  "payload": {
    "skill_installed": true,
    "version": "1.0",
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
  "aicp": "1.0",
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
  "aicp": "1.0",
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
  "aicp": "1.0",
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
  "aicp": "1.0",
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

`status` is `done` | `abandoned` | `failed`.

## error

```json
{
  "aicp": "1.0",
  "msg_type": "error",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "seq": 5,
  "ts": "2026-11-07T14:45:00Z",
  "from": { "role": "responder" },
  "to": { "role": "initiator" },
  "in_reply_to": 4,
  "payload": {
    "code": "format_mismatch",
    "detail": "Negotiated markdown; received json body."
  }
}
```

`code` is `timeout` | `malformed` | `format_mismatch` | `transport_lost` | `version_mismatch`.
