# Agent Context Protocol (ACP)

**A standard for scoped, deterministic context sharing between AI agents and external platforms.**

---

## The Problem

AI agents accumulate rich context about their humans — conversations, preferences, schedules, relationships, work details. As agents begin interacting with external platforms (social networks, commerce, collaboration tools, other agents), there is no standard way to control what context travels with them.

Today it's all-or-nothing: the agent either participates with its full context (dangerous) or doesn't participate at all.

This is the primary blocker to agent interoperability. Users won't let their agents interact externally until privacy is deterministic — not aspirational, not prompt-based, but structurally enforced.

## Design Principles

1. **Privacy must be deterministic.** Guidelines ("don't share X") are insufficient — agents are language models that can deviate. The system must make leakage structurally impossible.
2. **The human defines the boundaries, not the agent.** No LLM in the trust chain for scoping decisions. Permissions are human-authored, enforced mechanically by the runtime.
3. **Agents should remain useful under constraints.** A fully sandboxed agent with zero context is just a generic chatbot. The goal is maximum usefulness with minimum exposure.
4. **Framework-agnostic.** Not tied to any specific agent platform. An open protocol that any runtime can implement.

---

## The Proposal: Two Layers

### Layer 1: Scoped Sessions (Runtime Capability)

Agent runtimes need the ability to spawn sessions with constrained context. The agent operates with its full personality but only a curated subset of knowledge.

**The consent flow:**

1. **Platform declares context scopes** — what categories of context does it need? (e.g., personality, interests, bio, memory, calendar). Platforms advertise required and optional scopes, similar to OAuth.

2. **Consent UI during onboarding** — when connecting an agent to a platform, the human sees a scope selection screen as part of the platform's onboarding flow:

```
ClawMates is requesting access to your agent's context:

 ✅ Personality (SOUL.md) — required
 ✅ Custom bio — required
 ☐ Memory (MEMORY.md) — optional
 ☐ Daily notes (memory/*.md) — optional
 ☐ Tools & config (TOOLS.md) — optional

 [Allow]  [Deny]
```

3. **Grant stored in runtime config** — the approved scopes are stored in the agent runtime's configuration, *outside the agent's workspace*. The agent cannot see or modify its own permission grants. The grant is infrastructure, not a file the agent can edit.

4. **Scoped session on each interaction** — the runtime reads the stored grant and spawns a scoped session containing only the approved context. The scoped session has restricted tools (typically only outbound HTTP). No filesystem access, no memory search, no shell. The agent physically cannot access data outside its approved scope.

5. **Main agent orchestrates mechanically** — it fetches the external conversation, and the runtime handles context injection based on the stored grant. The main agent doesn't decide what context to include.

**Example grant storage (runtime config, outside agent workspace):**

```yaml
agents:
  list:
    - id: main
      contextGrants:
        clawmates:
          allow: ["SOUL.md"]
          customFields:
            bio: "Into AI, startups, surfing..."
          deny: ["MEMORY.md", "memory/*", "TOOLS.md", "HEARTBEAT.md"]
```

The grant is persistent — the human approves once, and every subsequent interaction uses the same scoped context automatically. The human can revoke or modify grants at any time.

### Layer 2: Agent Context Protocol (Interop Standard)

Once runtimes support scoped sessions, platforms and agent runtimes need a standardized way to negotiate context requirements.

**The protocol flow:**

1. **Platform publishes a context manifest** — a machine-readable declaration of what scopes it requires and optionally requests. (Analogous to OAuth client registration.)

2. **Runtime reads the manifest** during the platform's onboarding flow and presents a consent UI. The human approves or denies each scope.

3. **The runtime stores the grant.** The platform receives confirmation of which scopes were approved (but not the content itself).

4. **On each interaction**, the runtime mechanically spawns a scoped session with only the approved context. The platform never directly accesses the agent's files — only the scoped session's output (the agent's messages).

**The OAuth analogy:**

| OAuth | ACP |
|-------|-----|
| App requests API scopes | Platform requests context scopes |
| Human approves in browser | Human approves in consent UI |
| Token grants scoped API access | Runtime spawns agent with scoped context |
| Data accessed via API | Data never leaves runtime — only agent output reaches platform |

The key difference: in OAuth, the token grants access to data on a server. In ACP, the grant controls what context is *injected into an agent session*. The data never leaves the runtime — only the agent's output (its messages) reaches the platform.

---

## Why This Matters

Every agent interaction that crosses a trust boundary needs this:

- **Agent social networks** — agents socialize with scoped personal context
- **Agent commerce** — agents book, purchase, negotiate with only relevant preferences exposed
- **Agent collaboration** — agents from different organizations work together without leaking internal strategy
- **Enterprise interop** — company agents interact with vendor agents under strict data boundaries

This is the infrastructure layer that makes agent-to-agent interaction safe enough to actually happen at scale. Without it, agents stay siloed. With it, the agent economy opens up.

---

## Implementation Path

### Phase 1: Scoped Sessions (weeks)
- Add context grant / skip-bootstrap parameter to agent session spawning
- When set, suppress auto-injection of workspace files
- Only the task/prompt and specified files become the session context
- Combined with existing tool restrictions, achieves deterministic isolation

### Phase 2: Consent Flow & Scope Convention (months)
- Define a scope manifest format (JSON) that platforms publish
- Build consent UI that renders during platform onboarding
- Grant storage in runtime config, outside agent workspace
- Document the pattern as an open convention for other frameworks

### Phase 3: Full Protocol Specification (longer term)
- Formalize as an open protocol / RFC
- Cross-framework interop testing
- Verification/attestation layer — platform can verify agent is running under declared policy
- Runtime attestation for high-stakes interactions (TEEs, verifiable compute)

---

## Open Questions

1. **Dynamic context**: Static scopes work for social interactions, but commerce might need real-time preferences. Should scopes support dynamic fields the runtime populates per-interaction from approved sources?

2. **Verification**: How does a platform trust that an agent is actually running in scoped mode? Platform attestation is centralized. Runtime attestation is complex. Reputation is slow. What's the right starting point?

3. **Scope granularity**: One set of scopes per platform? Per interaction type? Per conversation? More granularity = more useful but more complex for the human.

4. **Attention markets**: If agents have scoped context, platforms could offer enhanced experiences in exchange for broader context access. "Share your preferences for better recommendations." Context becomes a tradeable asset with human-controlled permissions.

5. **Consent UI ownership**: Should the consent screen be rendered by the platform (better UX) or by the runtime (more trustworthy, like a browser permission dialog)?

---

## Contributing

This is a living proposal. Open an issue or PR to discuss, challenge, or extend it.

## License

CC-BY-4.0
