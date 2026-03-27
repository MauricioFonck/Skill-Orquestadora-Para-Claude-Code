# Skill Orquestadora para Claude Code

> Master orchestrator skill for Claude Code — deploys 10 parallel agents per task category across 19 domains, with autonomous mode, cross-session memory, and automatic Obsidian vault integration.

---

## What is this?

`/orquesta` is a Claude Code skill that transforms any request into a fully coordinated multi-agent execution. Instead of Claude working alone sequentially, it automatically spawns the right team of specialized agents working in parallel — delivering production-grade results in a single pass.

Works for **any domain**: software development, research, business analysis, UI design, security audits, data pipelines, networking, and more.

---

## Features

- **10 agents per task category** organized in 3 concurrent sub-blocks:
  - **Core** (Leader + Support + Architect) — produce the main output
  - **Quality Gate** (QA + Security + Testing + Refactoring) — validate in real time
  - **Delivery** (Docs + Optimization + DevEx) — prepare the final deliverable
- **19 task categories** auto-detected from your request
- **Autonomous mode** — executes without interruptions once triggered
- **Global Obsidian vault integration** — saves session summaries, concepts, and project notes automatically on `SessionEnd`
- **Smart agent selection** via `automation:auto-agent` + `automation:smart-agents`
- **SPARC methodology** for complex multi-component features
- **3-tier model routing** (WASM booster → Haiku → Sonnet/Opus) based on task complexity

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI installed
- Claude Code v1.x or higher

---

## Installation

1. Copy `orquesta.md` to your Claude Code commands folder:

```bash
# macOS / Linux
cp orquesta.md ~/.claude/commands/orquesta.md

# Windows
copy orquesta.md %USERPROFILE%\.claude\commands\orquesta.md
```

2. Restart Claude Code or open a new session.

3. Use it:

```
/orquesta [describe what you want to do]
```

---

## Usage Examples

```
/orquesta Create a JWT authentication system in Node.js with refresh tokens
```
→ Spawns 10 agents: backend-developer, security-engineer, test-automator, docs, CI/CD...

```
/orquesta Analyze the competitive landscape for a B2B SaaS in the HR space
```
→ Spawns: research-analyst, market-researcher, competitive-analyst, business-analyst...

```
/orquesta Design a responsive dashboard UI with dark mode
```
→ Spawns: ui-designer, frontend-developer, accessibility-tester, performance-engineer...

```
/orquesta Build a Cisco Packet Tracer topology with 2 routers, 3 switches and VLANs
```
→ Uses packet-tracer MCP directly with the Cisco-specific workflow

---

## Task Categories (19)

| Category | Domain |
|----------|--------|
| A | Code & Development |
| B | Databases |
| C | Debug & Errors |
| D | Testing & QA |
| E | Infrastructure & DevOps |
| F | GitHub & Git |
| G | Research & Documentation |
| H | Security & Auditing |
| I | Code Review & Refactoring |
| J | UI/UX & Design |
| K | AI & Machine Learning |
| L | Data & Analytics |
| M | Networks & Cisco Packet Tracer |
| N | Performance & Monitoring |
| O | Web Scraping & Automation |
| P | Project Planning |
| Q | Google Stitch Data Integration |
| R | Multimodal AI with Gemini |
| S | Claude-Flow v3 Swarm Intelligence |

---

## Autonomous Mode

When you want Claude to work with minimal intervention, the skill uses:

- `automation:auto-agent` — analyzes the task and selects optimal agents
- `automation:smart-agents` — auto-spawning based on file type and complexity
- `automation:session-memory` — persists context between steps
- `automation:self-healing` — detects failures and recovers without intervention

Claude only pauses for a **critical conflict** that would break the entire execution. Otherwise, it chains all steps automatically and reports when done.

---

## Obsidian Vault Integration

A `SessionEnd` agent hook in `~/.claude/settings.json` automatically:

1. Reads the session transcript
2. Classifies content into **DIARY / CONCEPTS / PROJECTS**
3. Saves to `C:/Users/[you]/Documents/MiVault/` with wikilinks
4. Updates `STATE.md`

This runs globally — from any directory — whenever Claude Code closes.

---

## MCPs Supported

`context7` `github` `postgres` `sqlite` `filesystem` `fetch` `memory` `sequential-thinking` `playwright` `firecrawl` `desktop-commander` `supabase` `mongodb` `puppeteer` `context-mode` `claude-flow` `nanobanana` `magic` `stitch` `packet-tracer`

---

## Token Optimization

- `context-mode` runs in background for semantic search before reading files
- Each agent receives **only the context relevant to its dimension** (50-65% token reduction)
- Conflict arbitration: Security > Performance > QA > Leader
- Early termination + agent reassignment when sub-tasks complete early

---

## License

MIT — free to use, modify, and share.

---

## Author

Built by [@MauricioFonck](https://github.com/MauricioFonck) for Claude Code.
Powered by [claude-flow](https://github.com/ruvnet/claude-flow).
