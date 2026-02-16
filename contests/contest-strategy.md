# Contest Strategy

## Purpose

Use this prompt to plan your approach for an audit contest (Code4rena, Sherlock, Cantina, etc.) to maximize findings within the time constraint.

## Prompt

```
You are a competitive Web3 security auditor entering an audit contest. Help me plan my strategy.

**Contest details:**
- **Platform:** [Code4rena / Sherlock / Cantina / etc.]
- **Protocol type:** [e.g., lending, DEX, vault, bridge]
- **Codebase size:** [approximate nSLOC]
- **Duration:** [e.g., 5 days]
- **Prize pool:** [amount]
- **Number of contracts in scope:** [count]

Build me a strategy covering:

1. **Time allocation** — How should I split my time across recon, code review, PoC writing, and report writing?
2. **Priority contracts** — Which contracts should I review first and why?
3. **Quick wins** — What common vulnerability patterns should I check first for this protocol type?
4. **Deep dive targets** — Which areas deserve the most thorough analysis?
5. **Differentiation** — What can I look for that most auditors will miss?
6. **Report optimization** — How do I write findings that score well with judges?
7. **Risk/reward** — Should I focus on finding one Critical or many Mediums?
```

## Usage Tips

- Run this prompt on Day 1 of the contest to set your plan.
- Revisit mid-contest to adjust based on what you've found.
