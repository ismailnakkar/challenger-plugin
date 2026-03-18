# Challenger Plugin

A Claude Code plugin that stress-tests your code, reasoning, and decisions through multi-agent adversarial challenge rounds. Instead of accepting conclusions at face value, Challenger assembles a panel of specialized adversaries that poke holes, gather evidence, and force refinement until your resolution is bulletproof.

## How It Works

When you invoke `/challenge`, the plugin:

1. **Classifies** what you're challenging (security, architecture, cost/complexity, or general reasoning)
2. **Auto-scales** intensity based on stakes — quick sanity check, deep review, or brutal stress test
3. **Dispatches parallel agents** — real subagents running simultaneously, each with isolated context and independent perspectives
4. **Runs multi-round refinement** silently — agents challenge, evidence is gathered, resolution is refined
5. **Delivers one concise result** — the battle-tested resolution with key challenges, evidence, and confidence score

## Agents

| Agent | Lens | What They Challenge |
|-------|------|-------------------|
| **Skeptic** | Logic & reasoning | Weak assumptions, logical fallacies, unverified claims |
| **Sentinel** | Security & risk | Threat models, attack vectors, data exposure, auth gaps |
| **Architect** | Design & scalability | Over-engineering, coupling, missing abstractions, scaling limits |
| **Pragmatist** | Cost & complexity | YAGNI violations, premature optimization, simpler alternatives |
| **Challenger Watch** | Advisory | Passively suggests `/challenge` during high-stakes moments |

Every challenge is **evidence-backed** — agents read your code, search for patterns, check git history, and run existing tests to support or dismiss their challenges. No hand-waving.

## Skills

### `/challenge`

The main skill. Runs adversarial challenge rounds against a resolution.

```
/challenge                              # Challenge the last conclusion in conversation
/challenge <topic>                      # Challenge a specific topic
/challenge --agents skeptic,sentinel    # Pick which agents to use
/challenge --intensity brutal           # Override auto-scaling
```

**Intensity levels:**

| Level | Rounds | Agents | When |
|-------|--------|--------|------|
| Quick | 1 | 1-2 | Minor decisions, simple claims |
| Deep | up to 3 | 2-3 | Architecture decisions, security-sensitive code |
| Brutal | no cap | All 4 | Production deploys, breaking changes, critical paths |

### `/challenge-report`

Saves the results of the last challenge session to a markdown file at `docs/challenge-reports/`.

```
/challenge-report
```

Generates a report with: final resolution, per-agent scores, round summaries, dismissed challenges, and evidence trail.

## Installation

### From GitHub

```bash
# Inside a Claude Code session:
/plugin marketplace add ismailnakkar/challenger-plugin
/plugin install challenger-plugin@challenger
```

### Local Development

```bash
claude --plugin-dir /path/to/challenger-plugin
```

## Example

```
You: /challenge Our API should use JWT tokens for authentication

Challenge initiated
Topic: JWT token authentication for API
Category: Security
Intensity: Deep — security-sensitive authentication decision
Agents: skeptic, sentinel, architect

[agents dispatched in parallel, running silently...]

## ✅ Challenge Complete

Resolution: Use JWT tokens with short-lived access tokens (15min),
HTTP-only refresh tokens, and server-side revocation via a token
blocklist. Store secrets in environment variables, not code.

Confidence: 8/10
Rounds: 2 | Intensity: Deep | Agents: skeptic, sentinel, architect

Key challenges that shaped this resolution:
- 🔴 sentinel: JWT in localStorage is XSS-vulnerable → moved to HTTP-only cookies
- 🟡 skeptic: No revocation strategy for compromised tokens → added token blocklist
- 🔵 architect: Refresh token rotation missing → added rotation on each refresh

Surviving open questions:
- Token blocklist storage strategy (Redis vs DB) not yet decided

Evidence:
- src/auth/middleware.ts:42 — current token validation has no expiry check
```

## Project Structure

```
challenger-plugin/
├── .claude-plugin/
│   ├── plugin.json           # Plugin manifest
│   └── marketplace.json      # Marketplace config
├── agents/
│   ├── skeptic.md            # Logic & reasoning challenger
│   ├── sentinel.md           # Security & risk challenger
│   ├── architect.md          # Design & scalability challenger
│   ├── pragmatist.md         # Cost & complexity challenger
│   └── challenger-watch.md   # Passive auto-suggest advisor
└── skills/
    ├── challenge/
    │   └── SKILL.md          # Multi-agent orchestrator
    └── challenge-report/
        └── SKILL.md          # Report generator
```

## License

MIT
