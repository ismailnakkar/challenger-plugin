# Challenger Plugin v2 — Multi-Agent Arena Design

## Overview

The challenger-plugin is a Claude Code plugin that provides rigorous adversarial validation of code, reasoning, and decisions. v2 evolves it from a single-agent rhetorical challenger into a multi-agent, evidence-backed adversarial arena with auto-scaling intensity and multiple trigger modes.

## Goals

- General-purpose adversarial tool that works equally well for code review and reasoning/decision validation
- Evidence-backed challenges that verify claims against actual code, git history, and test results
- Multiple specialized challenger personas that cover distinct adversarial angles
- Auto-scaling intensity that adapts to the stakes of what's being challenged
- Three trigger modes: manual invocation, auto-suggest, and hook-based
- Optional persistent artifacts (challenge reports)

## Agent Roster

Four specialized challenger agents plus one monitoring agent:

### skeptic (refined from v1)
- **Role:** Logic & reasoning challenger
- **Lens:** Assumptions, logical fallacies, weak evidence, circular reasoning
- **Tool use:** Reads code to verify claims made in resolutions
- **Personality:** Intellectually rigorous, never sycophantic, focused on finding the weakest point in any argument. Doesn't enjoy being wrong but doesn't enjoy false certainty either.

### sentinel
- **Role:** Security & risk challenger
- **Lens:** Threat models, attack vectors, data exposure, authentication gaps, injection risks
- **Tool use:** Reads code, checks for known vulnerability patterns, examines input validation
- **Personality:** Paranoid by design. Assumes every input is hostile, every boundary is permeable, every secret will leak. Calm and methodical, not alarmist.

### architect
- **Role:** Design & scalability challenger
- **Lens:** Over-engineering, tight coupling, missing abstractions, scaling bottlenecks, violation of SOLID principles
- **Tool use:** Reads code, checks git history for patterns and evolution, examines dependency graphs
- **Personality:** Thinks in systems. Challenges whether today's decision creates tomorrow's tech debt. Respects simplicity but demands intentionality.

### pragmatist
- **Role:** Cost & complexity challenger
- **Lens:** YAGNI violations, premature optimization, unnecessary abstractions, simpler alternatives
- **Tool use:** Reads code, can run tests to prove simpler approaches work
- **Personality:** The engineer who asks "do we actually need this?" Allergic to complexity theater. Will propose the 10-line solution that replaces 200 lines.

### challenger-watch (monitoring agent)
- **Role:** Background conversation monitor that suggests challenges
- **Lens:** Detects high-stakes decisions, unexamined assumptions, security-sensitive changes, architecture decisions with long-term implications
- **Behavior:** Does NOT challenge directly. Nudges the user: "This looks like a good candidate for `/challenge` — want me to stress-test it?"
- **Triggers on:** Decisions made quickly without deliberation, assumptions stated as facts, security-sensitive code being modified, breaking changes

## Skills

### /challenge (main skill, manual invoke)

The orchestrator skill that coordinates agents through challenge rounds.

**Invocation variants:**
- `/challenge` — challenges the last conclusion/plan in conversation context
- `/challenge <topic>` — challenges a specific named topic
- `/challenge --agents skeptic,sentinel` — manually select which agents participate
- `/challenge --intensity quick|deep|brutal` — override auto-scaling

**Auto-scaling logic:**
The skill analyzes what's being challenged and selects intensity:

| Intensity | Rounds | Agents | When |
|-----------|--------|--------|------|
| Quick | 1-2 | 1-2 | Simple factual claims, minor decisions, low-risk changes |
| Deep | 3-5 | 2-3 | Architecture decisions, complex logic, security-sensitive code |
| Brutal | 5+ | All 4 | Production deployments, breaking changes, security-critical paths |

**Auto-scaling signals:**
- Keywords: "deploy", "production", "breaking change", "security", "auth" → escalate
- Scope: single function → quick; multi-service → deep; infrastructure → brutal
- Risk indicators: user uncertainty, conflicting requirements, time pressure → escalate

### /challenge-report (artifact generation)

Saves the results of the most recent challenge session to a markdown file.

**Output location:** `docs/challenge-reports/YYYY-MM-DD-<topic>.md`

**Report contents:**
- Topic challenged
- Agents that participated
- Intensity level and number of rounds
- Each round's challenges and resolutions (condensed)
- Final resolution and composite confidence score
- Per-agent confidence scores
- Surviving open questions
- Dismissed challenges with reasons
- Evidence trail (links to code/files referenced)

## Challenge Protocol

### Round Structure (per agent, per round)

Each active agent takes a turn within a round. Agents challenge sequentially so each sees the refined resolution from the previous agent.

**Per-agent turn format:**

```
## Round [N] — [Agent Name] challenges: [topic]

**Current Resolution:**
[one clear paragraph]

**Challenge:**
[severity icon] [challenge type]: [description]

**Evidence:**
[code snippet, git history, test result, or logical proof backing the challenge]

**Impact:** [what happens if this challenge is ignored]

**Refined Resolution:**
[updated paragraph addressing valid challenges, or rebuttal if challenge is dismissed]

**Confidence:** [N]/10 — [reason]
```

### Multi-Agent Orchestration

1. Agents challenge sequentially within a round (order: skeptic, sentinel, architect, pragmatist)
2. Each agent sees the refined resolution from the previous agent in the same round
3. If agents disagree with each other, the disagreement is surfaced as a **tension** that the user must resolve
4. A round ends when all active agents have had their turn
5. Between rounds, the user is asked: "Accept this resolution or continue challenging?"

### Confidence Scoring

- Each agent scores independently on a 1-10 scale
- **Composite score** = weighted average, with weights based on relevance to the topic:
  - Security topic → sentinel weight 2x
  - Architecture topic → architect weight 2x
  - Cost/complexity topic → pragmatist weight 2x
  - General/reasoning → equal weights
- **Stopping conditions:**
  - Composite score >= 8 AND no individual agent scores below 6
  - User explicitly accepts ("accept", "looks good")
  - User explicitly stops ("stop challenging")
  - Maximum rounds reached (per intensity level)

### Final Output

```
## Agreed Resolution

**Resolution:** [final paragraph]
**Composite Confidence:** [N]/10
**Agent Scores:** skeptic [N], sentinel [N], architect [N], pragmatist [N]
**Surviving open questions:** [list]
**Dismissed challenges:** [with reasons]
**Evidence trail:** [links to code/files referenced]
```

## File Structure

```
challenger-plugin/
├── .claude-plugin/
│   └── plugin.json              # Updated manifest
├── agents/
│   ├── skeptic.md               # Logic & reasoning challenger
│   ├── sentinel.md              # Security & risk challenger
│   ├── architect.md             # Design & scalability challenger
│   ├── pragmatist.md            # Cost & complexity challenger
│   └── challenger-watch.md      # Background auto-suggest agent
└── skills/
    ├── challenge/
    │   └── SKILL.md             # Main orchestrator skill
    └── challenge-report/
        └── SKILL.md             # Artifact generation skill
```

## Architectural Decisions

1. **Purely declarative** — no executable code. The plugin uses markdown and JSON only, relying on Claude's built-in tool use for evidence gathering (Read, Grep, Glob, Bash for running tests).

2. **Each agent is self-contained** — personality, expertise, tool guidance, and challenge patterns defined in one markdown file per agent.

3. **Orchestration in the main skill** — `/challenge` SKILL.md handles agent selection, intensity auto-scaling, round management, multi-agent coordination, and output formatting.

4. **challenger-watch is an agent, not a skill** — it's a personality overlay that monitors conversation passively, not a user-invoked action.

5. **Reports are opt-in** — challenge sessions are ephemeral by default. Users invoke `/challenge-report` only when they want to persist the results.

## What Changed from v1

| Aspect | v1 | v2 |
|--------|----|----|
| Agents | 1 (skeptic) | 5 (skeptic, sentinel, architect, pragmatist, challenger-watch) |
| Challenge type | Rhetorical only | Evidence-backed with tool use |
| Intensity | Fixed (5 rounds) | Auto-scaling (quick/deep/brutal) |
| Triggers | Manual only (/challenge) | Manual + auto-suggest + hooks |
| Confidence | Single score | Per-agent + weighted composite |
| Artifacts | None | Optional challenge reports |
| Agent arguments | None | --agents, --intensity flags |