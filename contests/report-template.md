# Report Template

## Purpose

Use this prompt to generate well-structured contest submissions that judges can quickly evaluate and that maximize your scoring.

## Prompt

```
You are helping me write an audit contest submission for [Platform: Code4rena / Sherlock / Cantina].

Format my finding using this template:

---

## [H/M-##] Title: [Concise description of the vulnerability]

### Summary
[1-2 sentences: what is the bug and what contract/function is affected]

### Vulnerability Detail
[Detailed technical explanation of the root cause. Reference specific lines of code.]

### Impact
[What damage can an attacker cause? Quantify fund loss if possible. Explain who is affected (users, LPs, protocol).]

### Code Snippet
[Link to the vulnerable code or paste the relevant snippet]

### Proof of Concept
[Step-by-step exploit scenario OR a Foundry/Hardhat test demonstrating the issue]

### Recommended Mitigation
[Specific code change to fix the issue. Show before/after if possible.]

---

**My finding details:**
- **Contract:** [name]
- **Function:** [name]
- **Bug:** [description]
- **Impact:** [what happens]
- **How I found it:** [methodology]

Write this up as a polished contest submission. Be precise, reference code, and avoid filler.
```

## Usage Tips

- One finding per submission — don't bundle multiple issues.
- Judges value clarity over length. A concise, well-proven Medium beats a verbose, hand-wavy High.
- Always include code references (file, line number) so judges can verify quickly.
