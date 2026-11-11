# AICP examples

Short happy paths. Envelopes abbreviated where the shape is already in [references/envelope.md](references/envelope.md).

## Subagent — one round trip

User: “Ask a nested agent whether this README’s install steps are complete.”

1. Adapter: Subagent (whatever task/subagent tool the host provides). `session_id` minted. `seq` 0 `probe` with `"aicp": "1.2"` in the first spawn prompt.
2. Nested agent returns `probe_ack` with `"aicp": "1.2"`, `skill_installed: true`, `version: "1.2"`.
3. Resume the same nested agent with `handshake`: title “README install completeness”, description “Confirm missing install steps”, `expected_round_trips`: `1`, `payload_format`: `markdown`.
4. `handshake_ack` `accept`.
5. `turn` with the README excerpt in `payload.body`.
6. Nested agent `turn` lists gaps. Initiator `close` `status: done` with a one-line summary for the user.

## Mailbox — ~10 round trips

User: “Work with the other agent via `./aicp-shared` on the rate-limit design.”

1. Adapter: Mailbox. Session dir `aicp/<session_id>/` under that shared root (or `./aicp/<session_id>/` if they did not name a root).
2. Append `probe` (`"aicp": "1.2"`) to `to-responder.jsonl`. After `probe_ack` appears on `to-initiator.jsonl`, continue. Always send `"aicp": "1.2"`; accept inbound any `1.x`.
3. Handshake: `expected_round_trips`: `10`, `payload_format`: `markdown`, `who_speaks_first`: `initiator`.
4. Ten `turn` pairs, each a new JSONL line, `seq` increasing, `in_reply_to` set.
5. If a turn runs long and the peer advertised `1.2` or newer, either side may append `ping` with `seq: null` and optional `payload.info`. The other side appends `pong` with `seq: null` and optional `payload.info`. Neither line changes `seq`.
6. Either side appends `close` when success criteria are met.

## Human-as-relay — degraded AICP-lite

User: “I have another chat in a different product. Get it to review this error message.”

1. Adapter: human-as-relay. Show a fenced `aicp` `probe`. User pastes it over; pastes back a prose “yes I’m here” with no JSON.
2. Treat as `skill_installed: false`. Tell the user the session is AICP-lite.
3. Send Markdown handshake headers (Title, Description, Expected round trips: `1`, Payload format: `text`, Timestamp).
4. After the other side agrees, send the error message as the first turn. User pastes the review back. `close` locally with summary; offer a `close` block to paste if the other side can take it.
