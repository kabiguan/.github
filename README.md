# Kabiguan (KabiAI)

Kabiguan, or **KabiAI** for short, is a self-hosted personal AI assistant: a companion that lives in a messaging app, remembers your goals and context over time, checks in on you proactively, and can call external tools when it needs to. It runs on local models, so your data stays on your own hardware.

## About the Name

*Kabiguan* is a Tagalog word [meaning](https://tagalog.pinoydictionary.com/word/kabiguan/) failure or disappointment, and also a change to bad fortune. The project takes the name as a challenge rather than a warning: setbacks are part of pursuing any goal, and what matters is how quickly they are noticed and recovered from. KabiAI is built as a personal assistant and accountability partner for exactly that purpose. It remembers what its user has committed to, checks in when things go quiet, and helps turn a bad stretch around before it becomes a pattern.

> **Status: under restructuring.** The project started as a single repository (`kabi-ai`). It is being split into separate, containerised services under this organisation. See [The Migration](#the-migration) below.

---

## What KabiAI Does

- **Conversational assistant** with a consistent persona, reachable through a messaging channel (Discord for now)
- **Long-term memory**: conversation history, extracted facts and semantic recall via vector search
- **Proactive check-ins**: the assistant starts conversations about your goals instead of only replying
- **Tool use** through the Model Context Protocol (MCP): controlled database access and calendar integration, with more planned
- **Fully self-hosted**: local inference with Ollama, data in your own Postgres

## Core Stack

| Concern | Technology |
|---|---|
| Language | Python 3.12+ (managed with `uv`) |
| LLM inference | Ollama |
| Tool integration | MCP Python SDK |
| Messaging | discord.py |
| Storage and memory | PostgreSQL 16 + pgvector |
| Packaging and deployment | Docker / Docker Compose |

---

## The Migration

### Current state

Until now everything lived in one repository: the orchestrator, database layer, MCP client, CLI, utilities and infrastructure config, all in a single Python package (`kabi_ai`) with one `compose.yml`. That was the right call for getting a first end-to-end version working quickly.

### Target state

Each major component moves into its own repository in this organisation. Each is built as its own container image, and the services are wired together over a shared Compose network (with the option of moving to an orchestrator later).

### Why the split is happening

- **Independent deployment.** Tool servers change often, while the orchestrator should stay stable. Separate repos and images let each ship on its own schedule.
- **Isolation.** Some services need different privileges and dependencies. Tool servers run in their own containers with only the access they need, away from the core that holds user data.
- **Swappable edges.** The messaging adapter is designed to be modular. If the Discord bot lives in its own service, replacing it with SMS means a new service, not a change to the core.
- **Independent dependencies.** Each service pins only what it needs, rather than the whole system sharing one dependency set.
- **Reusability.** A well-built MCP tool server (calendar integration, for example) is useful beyond this project and can be published or reused on its own.
- **Clear service boundaries.** Services talk through defined interfaces, which makes the system easier to reason about and test.

### Accepted trade-offs

Splitting adds real overhead: cross-repo changes need coordinated PRs, shared schemas need a home, and local development means running several services together. The project takes this on deliberately and will mitigate it with a central infra repo, versioned interfaces and one-command local startup.

---

## Proposed Repositories

> Names and boundaries are provisional and may change as the split progresses. Repositories are named after the organ whose role they play.

| Repo | Purpose |
|---|---|
| `kabi-brain` | Core agent loop: context assembly, memory, persona, Ollama calls, MCP routing |
| `kabi-hippocampus` | Schema, Database Documentation |
| `kabi-voice-discord` | Discord messaging adapter (thin translation layer to the orchestrator) |
| `kabi-heart` | Proactive check-in jobs and quiet-hours logic |
| `kabi-limbs` | General-purpose MCP tool server (controlled database access, calendar integration). Specialised tool servers may be added later |
| `kabi-skeleton` | Docker Compose files, env templates, init scripts and deployment docs |

## Architecture at a Glance

```
   Messaging adapter (Mouth (Discord))
              │  (user_id, text)
              ▼
        Orchestrator (Brain) ──────► Ollama (LLM + embeddings)
         │        │
         │        └────────► MCP client  ──► MCP tool servers (Limbs)
         ▼
   Postgres + pgvector  (Hippocampus)      Scheduler ──► Orchestrator (proactive check-ins)
```

The orchestrator is channel-agnostic. It exposes one entrypoint, roughly `handle_turn(user_id, text) -> response`, and never imports anything adapter-specific.

## Roadmap

- [x] Postgres + pgvector
- [x] Orchestrator, memory and CLI harness (monorepo)
- [ ] MCP tool servers (controlled database access, calendar integration)
- [ ] Split services into separate repositories
- [ ] Containerise each service and connect them via Compose
- [ ] Proactive check-in scheduler

## License

To be decided.
