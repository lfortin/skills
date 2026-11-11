# The AICP Skill Manifesto

A local agent and a remote agent start as strangers. They may share a chat, a folder, or only a person willing to paste between them. They may share nothing else: no common product, no common task model, no common elaborate protocol. The remote agent's existence may not be proven yet. A probe is the first act, because only a reply establishes that the remote agent is there. AICP exists so they can still discover each other, and so that discovery stays small enough to finish before either side commits to a heavier conversation.

## Discovery before commitment

A session begins when the remote agent sends a reply. Until that reply arrives, the initiator does not know whether anyone is there, whether that party can speak back, or what it can speak.

AICP is the basic process for learning those facts, in order:

1. **Presence.** A probe asks only “can you reply?” A `probe_ack` is the proof. Silence ends the attempt. A session requires a reply.
2. **Capability.** The reply says what the remote actually has: whether AICP is installed, which dialect it speaks, which message types and payload formats it accepts. The initiator reads that answer before proposing work.
3. **Agreement.** A handshake names the inquiry — title, description, expected length, payload format — and the responder accepts, counters, or declines. Discussion starts only after both sides have agreed.
4. **Exchange.** Turns follow the agreement. Either side may close, with a summary a person can reuse.

Each step is a question. The next step is allowed only when the previous answer is known.

## Finding a richer protocol

The capability step is also how two agents learn whether they can leave this basic process for something more elaborate.

Google’s Agent2Agent protocol (A2A) is one such protocol: agent cards, tasks, artifacts, and a fuller model of delegated work. Other products will have their own. Those protocols earn their place once both sides can speak them and the work needs them. AICP asks first, while the other side might still be absent, able to answer only in prose, or committed to a different stack.

Presence proves a peer. Capability tells the initiator what that peer can use. If both agents can run A2A, or another named protocol the user wants, the session has done its work: the two sides know an upgrade is possible, and they can move the real task onto that protocol. If only one side can, or neither can, they still have a shared, bounded way to inquire, answer, and close.

The skill stays basic so this question can be asked of a nested agent, of a peer that only shares a directory, or of a chat a person relays by hand. The same discovery runs on all three transports.

## When the remote has no skill

Discovery includes the case where the remote can reply and has no AICP skill. That session is AICP-lite: the same field names, written so a plain reply can still carry them. The user is told the session is degraded. The inquiry can still have a title, a description, a bound on round trips, and a close. A peer that has never seen an envelope can still answer.

## Standing conditions

- **The transport is an adapter.** The envelope is the same whether it rides a subagent return, an append-only mailbox, or a paste between two chats. Discovery itself stays one process.
- **The remote agent is untrusted.** Discovery and inquiry carry secrets, credentials, or private user data only when the user allows that for this session. A person sees the inquiry before it is sent when the agent inferred it.
- **First contact is bounded.** The handshake states how long the exchange is expected to last and how it should end. An unbounded conversation is an explicit choice, recorded as such.
- **A heartbeat is not a turn.** Either side may ask whether the session is still alive. That check stays outside the sequence, so a slow reply is not mistaken for a new step.
- **Majors must match.** Additive minor versions can still discover each other. A breaking envelope stops, and says why.

## The point

Two agents should be able to learn, from a real reply, three things: that the other side can speak back, what it is able to speak, and whether a more elaborate protocol such as A2A is available to both. AICP is that process. Everything heavier waits until those answers exist.
