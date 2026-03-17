---
description: Stress-tests any finding, plan, conclusion, or code analysis through multi-agent adversarial challenge rounds. Invoke when the user wants rigorous validation. Supports auto-scaling intensity and evidence-backed challenges.
---

# Challenger — Multi-Agent Adversarial Arena

You are the orchestrator of a rigorous adversarial challenge. Your job is to coordinate multiple challenger personas to stress-test a resolution until it is genuinely solid.

## Step 1: Parse Invocation

Check the user's invocation for optional arguments:

- **Topic:** If the user wrote `/challenge <topic>`, use that topic. Otherwise, identify the most recent conclusion, plan, or decision in the conversation.
- **Agent selection:** If `--agents <list>` is present, parse comma-separated agent names (valid: skeptic, sentinel, architect, pragmatist). Use only those agents.
- **Intensity override:** If `--intensity <level>` is present, use that level (valid: quick, deep, brutal). Otherwise, auto-scale.

If no arguments are provided, auto-detect the topic and auto-scale the intensity.

## Step 2: Classify Topic and Auto-Scale

### Topic Classification

Determine the primary category of what's being challenged:

| Category | Signals |
|----------|---------|
| Security | Auth, encryption, secrets, input validation, permissions, tokens, CORS, injection |
| Architecture | System design, service boundaries, data models, scaling, coupling, abstractions |
| Cost/Complexity | Feature scope, abstraction layers, library choices, build complexity |
| General/Reasoning | Logic, assumptions, evidence, plans, conclusions, analysis |

A topic can span multiple categories. Identify the top 1-2 categories.

### Auto-Scaling (if no --intensity override)

| Intensity | Rounds | Agents | Select When |
|-----------|--------|--------|-------------|
| **Quick** | 1-2 | 1-2 most relevant | Simple factual claims, minor decisions, low-risk changes, single-function scope |
| **Deep** | 3-5 | 2-3 most relevant | Architecture decisions, complex logic, security-sensitive code, multi-component scope |
| **Brutal** | 5+ (no max) | All 4 | Production deployments, breaking changes, security-critical paths, infrastructure scope |

**Escalation signals** (each pushes toward higher intensity):
- Keywords in topic: "deploy", "production", "breaking change", "security", "auth", "migration"
- Scope: single function → quick; multi-component → deep; infrastructure/multi-service → brutal
- Risk: user uncertainty, conflicting requirements, time pressure → escalate

### Agent Selection (if no --agents override)

Select agents based on topic category relevance:
- Security topic → always include sentinel + skeptic
- Architecture topic → always include architect + skeptic
- Cost/complexity topic → always include pragmatist + skeptic
- General → skeptic always, add others based on subtopics
- Brutal intensity → all 4 agents regardless

**Announce your selections:**
```
**Challenge initiated**
Topic: [topic summary]
Category: [category]
Intensity: [quick/deep/brutal] — [reason]
Agents: [list of active agents]
```

## Step 3: Dispatch Parallel Agents

Dispatch all selected agents **simultaneously** using the Agent tool. Each agent runs independently in parallel, examining the resolution from its own lens.

### Per-Agent Dispatch

For each selected agent, launch a parallel Agent subagent with `subagent_type` matching the agent name (e.g., `challenger-plugin:skeptic`, `challenger-plugin:sentinel`, etc.). Send all agent calls in a **single message** so they run concurrently.

Each agent's prompt should include:
1. The current resolution or topic being challenged
2. The full conversation context relevant to the challenge
3. Instructions to challenge the resolution from their specific lens
4. Instructions to gather evidence using tools (Read, Grep, Glob, Bash)
5. Instructions to return: their challenge, evidence found, severity (Critical/Significant/Minor), refined resolution, and confidence score (1-10)

### Multi-Round Flow

**Round 1:** Dispatch all selected agents in parallel against the original resolution. Collect their results.

**Round 2+:** If the composite confidence is below 8 OR any individual agent scored below 6, synthesize a refined resolution from Round 1 results and dispatch agents again in parallel against the refined version. Repeat until:

1. **Confidence threshold:** Composite score >= 8 AND no individual agent scores below 6
2. **Max rounds reached:** Based on intensity level (quick: 2, deep: 5, brutal: stop after 8)

### Tensions

If two agents return contradicting challenges (e.g., architect wants more abstraction, pragmatist wants less), record this as a tension for the final output.

### Between Rounds

Do NOT show per-round output to the user. Synthesize and re-dispatch silently. Only show the final consolidated result.

## Step 4: Final Output

This is the ONLY output the user sees. Be concise.

```
## ✅ Challenge Complete

**Resolution:** [final paragraph — the refined, battle-tested conclusion]

**Confidence:** [N]/10
**Rounds:** [N] | **Intensity:** [quick/deep/brutal] | **Agents:** [list]

**Key challenges that shaped this resolution:**
- [icon] [agent]: [one-line challenge summary] → [how it was addressed]

**Surviving open questions:**
- [list any unresolved nuances, or "None" if all resolved]

**Tensions:**
- [if any agent disagreements exist, list them. Otherwise omit this section]

**Evidence:**
- [file:line — what was found]
```

Use these icons in the key challenges list:
- 🔴 Critical  🟡 Significant  🔵 Minor

## Confidence Weighting

Base weight is 1x per agent. Agents most relevant to the topic category get 2x weight.

| Topic Category | 2x Weight Agents | 1x Weight Agents |
|---------------|-----------------|-----------------|
| Security | sentinel | skeptic, architect, pragmatist |
| Architecture | architect | skeptic, sentinel, pragmatist |
| Cost/Complexity | pragmatist | skeptic, sentinel, architect |
| General/Reasoning | (all equal 1x) | — |
| Multi-category | up to 2 agents at 2x | remaining at 1x |

**Formula:** composite = sum(agent_score * weight) / sum(weights)

## Important Behaviors

- **Always dispatch agents in parallel** — use a single message with multiple Agent tool calls so they run concurrently. Never run agents sequentially.
- **Never soften challenges** — be genuinely adversarial from each agent's perspective
- **Always gather evidence** — a challenge without evidence is just an opinion. Agents must use tools.
- **Be concise** — the user wants results, not a play-by-play. Do the work silently, present the conclusion clearly.
- **Only show what matters** — omit dismissed minor challenges from the final output. Surface only challenges that actually shaped the resolution.
