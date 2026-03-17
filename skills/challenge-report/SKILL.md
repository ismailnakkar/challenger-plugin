---
description: Saves the results of the most recent /challenge session to a persistent markdown report. Invoke after a challenge session to create an audit trail.
---

# Challenge Report Generator

Saves the results of the most recent `/challenge` session to a markdown file.

## Pre-Check

If there is no `/challenge` session in the current conversation, respond:

> "No challenge session found in this conversation. Run `/challenge` first, then use `/challenge-report` to save the results."

Do not proceed further.

## Generate Report

If a challenge session exists, extract the following from the conversation and write a markdown report:

### Report Template

```markdown
# Challenge Report: [Topic]

**Date:** [YYYY-MM-DD]
**Intensity:** [quick/deep/brutal]
**Rounds:** [N]
**Agents:** [list]
**Composite Confidence:** [N]/10

## Final Resolution

[final resolution paragraph]

## Agent Scores

| Agent | Score | Key Concern |
|-------|-------|-------------|
| [name] | [N]/10 | [one-line summary of their main challenge] |

## Challenge Rounds

### Round 1
[condensed summary: what was challenged, what changed]

### Round 2
[condensed summary]

...

## Surviving Open Questions

- [list]

## Dismissed Challenges

| Challenge | Agent | Reason Dismissed |
|-----------|-------|-----------------|
| [description] | [agent] | [reason] |

## Evidence Trail

- [file:line — what was found and why it matters]
```

### Save Location

Save to: `docs/challenge-reports/YYYY-MM-DD-<topic-slug>.md`

- Create the `docs/challenge-reports/` directory if it doesn't exist
- Convert the topic to a URL-safe slug for the filename (lowercase, hyphens, no special chars)
- If a file with that name already exists, append a number: `-2`, `-3`, etc.

After saving, confirm:

> "Challenge report saved to `docs/challenge-reports/[filename].md`"
