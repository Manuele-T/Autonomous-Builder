# Autonomous Builder

> A Discord-controlled autonomous app builder powered by [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research.

You describe what you want built. The agent asks clarifying questions, writes a plan, waits for your approval, then builds it locally — one folder per project, zero manual coding.

---

## How It Works

Send a message to a Discord bot. Get a working application back.

```
Discord message
      ↓
Clarification       up to 3 rounds of questions
      ↓
Planning            writes PLAN.md inside a new project folder
      ↓
Approval Gate       stops and waits for your YES
      ↓
Build               creates all files, runs npm install
      ↓
Validation          runs build + tests, reports pass/fail
      ↓
Report              tells you where files are and what was built
```

The agent **cannot skip phases** and **cannot write a single line of code before you approve the plan**. Human-in-the-loop by design, not as an afterthought.

---

## Stack

| | |
|---|---|
| Agent framework | Hermes Agent v0.14.0 |
| Model | MiniMax M2.7 via OpenRouter |
| Cost per build | ~$0.05 |
| Interface | Discord |
| Runtime | WSL2 (Ubuntu) on Windows |

---

## Project Layout

Each app gets its own isolated folder, for example:

```
~/projects/
├── todo-app/
│   ├── PLAN.md       ← generated before any code is written
│   └── src/
├── restaurant-website/
│   ├── PLAN.md
│   └── ...
```

---

## Key Files

| File | Purpose |
|---|---|
| `SOUL.md` | Defines the agent's identity, pipeline phases, and constraints |
| `config.yaml` | Hermes profile config (secrets removed — add your own keys) |

---

## Phase 2 — Roadmap

- [ ] GitHub push after each build
- [ ] Vercel deployment + live URL reported to Discord
- [ ] Docker containerisation
- [ ] Stricter coding guardrails in SOUL.md (TypeScript, tests, input validation)

---

## Why This Matters

The approval gate pattern — where an autonomous agent cannot proceed without explicit human sign-off — mirrors the governance frameworks now recommended for agentic AI in regulated environments. This is Phase 1 of an ongoing exploration of human-in-the-loop autonomous pipelines.

---

Built by Manuele · [LinkedIn](https://www.linkedin.com/in/manueletacchetti/)
