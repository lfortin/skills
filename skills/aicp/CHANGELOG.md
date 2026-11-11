# Changelog

AICP versions. The wire dialect (`"aicp"`, and `probe_ack.payload.version`) is MAJOR.MINOR. The skill package (`SKILL.md` `metadata.version`) is the same minor with a patch. Newest release first.

## 1.2 — 2026-11-11

Skill package `1.2.0`. Wire dialect `"1.2"`. Additive. Compatible with any `1.x`.

### Added

- `ping` and `pong`, a heartbeat that confirms the session is still alive without advancing discussion.
- Optional `payload.info` on either frame. On `ping` it can say why the check was sent (for example, the remote is taking unexpectedly long). On `pong` it can add a note. An empty payload is valid.
- Heartbeats use `seq: null` and `in_reply_to: null`. They do not change `last_seq`, and they are not sequence drift or a `target_seq`.
- Mailbox reads pick up `seq: null` lines without moving the sequence cursor.
- `supported_msg_types` advertises `ping` and `pong`.

### Changed

- This skill sends `"aicp": "1.2"` and advertises `version: "1.2"`.
- `ping` is sent only to a peer at `1.2` or newer. A `1.0` or `1.1` peer is not expected to understand these types.
- A missing `pong` does not by itself end the session.
- On a subagent that cannot accept a second message until the current task returns, wait for that return instead of pinging.

## 1.1 — 2026-11-08

Skill package `1.1.0`. Wire dialect `"1.1"`. Additive. Compatible with `1.0`.

### Added

- Semantic versioning for the wire dialect: MAJOR.MINOR, no patch. Same major is compatible. A different major is `version_mismatch` and stops the session.
- `error` fields `retryable`, `target_seq`, `retries_remaining`, and optional `backoff_ms`.
- Error codes `payload_corrupted`, `out_of_order`, and `rate_limited`.
- Retransmission: a retryable error names the frame to resend; the peer sends that content again as a new envelope (`seq = last_seq + 1`). The retry budget only decreases.

### Changed

- Outbound envelopes send `"aicp": "1.1"`. Inbound `1.x` is accepted. Unknown keys on a later `1.x` envelope are ignored.
- Every `error` carries `code`, `detail`, and `retryable`. Omitting `retryable` means `false`.
- A `1.0` peer omits `retryable` and `retries_remaining`. Treat that as a terminal error. Do not expect that peer to honor the retry contract.
- An incompatible major no longer tries to negotiate down to `1.0`.
- A mailbox lock or a spawn retry is adapter I/O. It is not an envelope `retryable` resend.

## 1.0 — 2026-11-07

Skill package `1.0.0`. Wire dialect `"1.0"`. First release.

### Added

- Two-way session between an initiator and a responder: presence (`probe` / `probe_ack`), capability, handshake (`handshake` / `handshake_ack`), discussion (`turn`), and `close`.
- One JSON envelope: `aicp`, `msg_type`, `session_id`, `seq`, `ts`, `from`, `to`, `in_reply_to`, `payload`.
- Message types `probe`, `probe_ack`, `handshake`, `handshake_ack`, `turn`, `close`, and `error`.
- One combined `seq` for the session, starting at `0`.
- Handshake fields for title, description, expected round trips, payload format, and the optional goal, limits, and confidentiality fields.
- `handshake_ack` decisions `accept`, `counter`, and `decline`.
- AICP-lite: the same field names as Markdown headers when the remote can reply and has no skill.
- Three transports: subagent, append-only mailbox, and human-as-relay.
- Error codes `timeout`, `malformed`, `format_mismatch`, `transport_lost`, and `version_mismatch`. An error ends the session.
- The remote agent is untrusted. Default confidentiality is `no_secrets`.
