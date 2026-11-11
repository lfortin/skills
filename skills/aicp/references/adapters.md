# AICP transport adapters

The envelope is the same on every adapter. Pick one from session context. Do not invent a fourth transport unless the user names one.

## Subagent (task tool)

**When:** The host can spawn or resume a nested agent in this session and you get its reply as a return value. Product-agnostic: Cursor Task, Claude Code subagents/tasks, ChatGPT agent/task tools, or any similar spawn-and-wait API.

If the other agent lives in a **separate** chat or product and the user must copy messages, use **Human-as-relay** instead — even if that other product is Cursor, Claude Code, ChatGPT, etc.

**How messages move:** Each spawn or resume payload contains **one** AICP envelope as a fenced JSON block. No session files required. Use whatever the host calls the tool (`Task`, `subagent`, `agent`, resume, follow-up).

**Presence:** Spawn or resume the nested agent with the `probe` envelope and an instruction to reply with only a `probe_ack` envelope first. That tool’s return **is** the remote’s send path. If the return has no `probe_ack`, presence failed.

**Later turns:** Resume the **same** nested agent (same thread/id if the host provides one) with the next envelope (`handshake`, `turn`, `close`, `error`). Require the return to contain the next envelope. A `ping` checks liveness and is not the next discussion envelope. If this tool cannot accept a second message until the current task returns, wait for that return instead of pinging.

**Instruction to include with the first spawn:**

```
You are the AICP responder. Read the aicp skill if you have it.
Reply with a single AICP JSON envelope. First message you received is probe;
respond with probe_ack (skill_installed true if AICP is loaded).
```

## Mailbox

**When:** Both agents can read and write a shared directory.

**Layout** (append-only JSONL, one envelope per line):

```
aicp/<session_id>/to-responder.jsonl
aicp/<session_id>/to-initiator.jsonl
```

Initiator **appends** to `to-responder.jsonl`. Responder **appends** to `to-initiator.jsonl`. Never rewrite earlier lines. Optional `aicp/<session_id>/session.json` may store `session_id`, adapter, and handshake snapshot after accept.

**Concurrent writes:** Two agents can hit a brief OS lock if they write at the same time (especially to `session.json`, or if a side ever appends while another process touches the same file). Append atomically (one complete JSONL line per write, e.g. open-append-write-close, or write a temp file and rename into place for `session.json`). If a write fails with a lock or busy error, retry a few times with a short backoff (tens to a few hundred milliseconds) instead of treating it as `transport_lost` on the first failure. That is adapter I/O retry, not envelope `retryable` (resend of an AICP frame).

**Presence:** Append `probe` to `to-responder.jsonl`. Poll `to-initiator.jsonl` until a `probe_ack` with the same `session_id` appears. Default wait **60 seconds** unless the user sets another timeout. If the file never gains that line, presence failed.

**Polling:** Read new lines after the last sequenced seq you have seen, and also new lines with `seq: null` (`ping`, `pong`). A null seq does not move the seq cursor. Ignore envelopes with a different `session_id`.

**Create the directory** if missing: `aicp/<session_id>/`.

## Human-as-relay

**When:** Two chats or products; the user pastes messages between them.

**How messages move:** Show the user one fenced block labeled `aicp`. Ask them to paste it to the other agent and paste the reply back here.

````markdown
Paste this entire block to the other agent:

```aicp
{ ...envelope... }
```

Then paste their reply here.
````

**Presence:** The pasted `probe_ack` (JSON or AICP-lite headers) is the proof of two-way communication. Wait for the user; do not invent a reply.

**AICP-lite on this adapter:** If the other chat has no AICP skill, send handshake as Markdown headers instead of JSON after you know capability is missing:

```markdown
# AICP-lite handshake
- Title: ...
- Description: ...
- Expected round trips: 1
- Payload format: markdown
- Timestamp: 2026-11-07T14:43:20Z
```
