# Time Management

## Purpose

Use this prompt to create a structured schedule for an audit contest, avoiding the common trap of spending too long on recon and running out of time for deep analysis.

## Prompt

```
I am participating in a [duration]-day audit contest for a [protocol type] with approximately [nSLOC] lines of Solidity code across [N] contracts.

Help me create a day-by-day schedule:

**Day-by-Day Breakdown:**
For each day, specify:
- Primary focus area
- Time blocks (morning / afternoon / evening)
- Deliverables by end of day
- Go/no-go checkpoints

**General Framework:**
- Day 1: Recon, docs, architecture understanding, initial threat model
- Day 2-3: Deep code review of priority contracts
- Day 4: PoC development for findings, edge case hunting
- Day 5: Report writing, polish, submission

**Rules to enforce:**
1. Stop recon by end of Day 1 — no matter what
2. Start writing reports for confirmed findings on Day 3, not Day 5
3. If a rabbit hole takes >2 hours without progress, move on
4. Reserve the last 20% of time purely for report quality
5. Track findings in a simple log as you go (severity, contract, one-liner)

Adapt this schedule to my specific contest parameters and suggest any modifications.
```

## Usage Tips

- Print this schedule and check off milestones as you go.
- The biggest mistake in contests is spending 80% of time reading and 20% writing — invert that ratio for findings quality.
