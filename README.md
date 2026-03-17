# Challenger Plugin

A Claude Code plugin that stress-tests your code, reasoning, and decisions through multi-agent adversarial challenge rounds. Instead of accepting conclusions at face value, Challenger assembles a panel of specialized adversaries that poke holes, gather evidence, and force refinement until your resolution is bulletproof.

## How It Works

When you invoke `/challenge`, the plugin:

1. **Classifies** what you're challenging (security, architecture, cost/complexity, or general reasoning)
2. **Auto-scales** intensity based on stakes — quick sanity check, deep review, or brutal stress test
3. **Assembles** the right panel of challenger agents for the topic
4. **Runs rounds** where each agent challenges the resolution from their unique angle, gathering real evidence from your codebase
5. **Refines** the resolution until it reaches high confidence or you accept it

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
| Quick | 1-2 | 1-2 | Minor decisions, simple claims |
| Deep | 3-5 | 2-3 | Architecture decisions, security-sensitive code |
| Brutal | 5+ | All 4 | Production deploys, breaking changes, critical paths |

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

## Round 1 — Skeptic challenges: JWT authentication
...
## Round 1 — Sentinel challenges: JWT authentication
...
## Round 1 — Architect challenges: JWT authentication
...

Round 1 Complete
Composite Confidence: 6/10

> Accept this resolution, or shall I keep challenging?
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
