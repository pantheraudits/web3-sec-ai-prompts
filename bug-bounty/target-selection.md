# Target Selection

## Purpose

Use this prompt to evaluate and prioritize bug bounty targets based on protocol characteristics, bounty size, code complexity, and attack surface.

## Prompt

```
You are a Web3 security researcher evaluating bug bounty targets. Analyze the following program and help me decide whether it's worth hunting on:

- **Program:** [Protocol Name]
- **Platform:** [Immunefi / Code4rena / Sherlock / etc.]
- **Bounty range:** [e.g., $1k – $100k]
- **In-scope contracts:** [list or GitHub link]
- **Chain(s):** [e.g., Ethereum, Arbitrum]
- **Protocol type:** [e.g., DEX, lending, bridge, vault]

Please evaluate:
1. Attack surface size (number of contracts, LOC, external integrations)
2. Complexity vs. bounty ratio — is the payout worth the effort?
3. Common vulnerability patterns for this protocol type
4. What areas of the codebase are most likely to contain bugs?
5. Any red flags or signs the code is well-hardened?
6. Recommended hunting strategy (what to focus on first)
```

## Usage Tips

- Fill in as much detail as possible for better analysis.
- Pair this with `recon-checklist.md` for a full target onboarding workflow.
