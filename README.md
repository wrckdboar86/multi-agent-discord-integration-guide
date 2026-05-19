# Multi-Agent Discord Integration Guide

> A practical guide to wiring multiple AI agents into a shared Discord channel for real-time ops communication.

---

## What This Is

This guide documents a **peer-agent architecture** for connecting multiple AI agents to a single Discord channel. Each agent can read and post independently — no central router, no shared codebase, no single point of failure.

The pattern is **platform-agnostic**. It works for any agent that can make HTTP calls and read from cloud storage — Claude, Gemini, GPT, or a custom agent runtime.

---

## The Pattern

```pattern
Human Operator
      │
      │ directs
      ▼
 ┌─────────────────────────────┐
 │       Shared Channel        │
 │      (Discord #ops-room)    │
 └────────────┬────────────────┘
              │
   ┌──────────┼──────────┐
   ▼          ▼          ▼
Agent A    Agent B    Agent C
Briefings  Analysis   Code
```

Each agent has:

- Its own **Discord bot** (registered in the Developer Portal)
- Its own **bot token** stored in a private cloud doc
- Its own **webhook** for posting (write path)
- Direct **Bot API access** for reading (read path)

The human operator decides which agent handles which task. Agents may read each other's messages for context but do not act on them autonomously.

---

## What's in the Guide

The PDF covers 12 sections end-to-end:

| §  | Section                                 |
|--- |---                                      |
| 01 | Overview & Architecture                 |
| 02 | Prerequisites                           |
| 03 | Step 1 — Channel Setup                  |
| 04 | Step 2 — Bot Provisioning               |
| 05 | Step 3 — Write Path (Webhook)           |
| 06 | Step 4 — Read Path (Bot API)            |
| 07 | Step 5 — Token Storage                  |
| 08 | Step 6 — Channel Permissions            |
| 09 | Step 7 — Full Duplex Verification       |
| 10 | Multi-Agent Coordination Model          |
| 11 | Lessons Learned (10 field observations) |
| 12 | Security Hardening Checklist            |

---

## Key Concepts

**Write path — Webhook**
Each agent gets a Discord webhook for posting. Simple HTTP POST, no auth header required beyond the URL. Write-only — webhooks cannot read messages.

**Read path — Bot API**
Each agent uses a Discord Bot token to call the REST API directly:

```example
GET https://discord.com/api/v10/channels/{channel_id}/messages
Authorization: Bot {token}
```

**Token storage**
Bot tokens live in a dedicated private cloud doc per agent — never hardcoded in instructions, skills, or code. Always fetched fresh per session, never cached.

**Permission layers**
Four layers must all pass for a bot to operate in a channel:

1. Valid token (Layer 1 — 401 if broken)
2. Server membership via OAuth2 invite (Layer 2 — 403 if broken)
3. Channel-level permissions (Layer 3 — 403 if broken)
4. Privileged intents enabled in Developer Portal (Layer 4 — silent fail)

---

## Lessons Learned Highlights

A few things that burned the most time during the actual build:

- **"Active" status lies** — a platform dashboard showing "Connected" does not mean the token works. Always test with an actual API call.
- **Managed bot connections are brittle** — if a third-party platform manages your Discord bot token, you have no recovery path when it breaks. Use your own bot from the Developer Portal.
- **Peer model beats router model** — building a central dispatcher adds complexity for minimal gain. Human-in-the-loop routing is faster to build and easier to audit at this scale.
- **Token hygiene is non-negotiable** — a leaked bot token gives full write access to every server the bot is in. Rotate first, investigate second.

Full lessons in the guide.

---

## Who This Is For

- Developers exploring multi-agent AI architectures
- Anyone connecting LLM-based agents to shared communication channels
- Teams looking to build auditable, peer-based agent workflows
- Anyone who wants a real-world pattern that was built, debugged, and documented — not a theoretical diagram

---

## What This Is Not

- A tutorial for any specific AI platform
- A guide that assumes you have a DevOps background
- A production-grade enterprise deployment blueprint

This is a practical pattern from a real build. Adapt it to your stack.

---

## Download

📄 [Multi-Agent Discord Integration Guide v1.pdf](./Multi-Agent%20Discord%20Integration%20Guide%20v1.pdf)

---

## License

MIT — use it, share it, build on it.

---

*Built by Anthony Ocasio — [LinkedIn](https://www.linkedin.com/in/anthonyocasio)*
