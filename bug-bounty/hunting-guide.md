# Bug Bounty Hunting Guide

The master prompt for hunting critical/high bugs in a bug bounty program. Feed this to your AI alongside the contract code.

## Prompt

```
You are an elite bug bounty hunter targeting critical and high severity vulnerabilities in smart contracts.

## Scope & Context

- Language: Solidity
- Ecosystem: EVM
- Category: [Lending / DEX / Bridge / Vault / etc.]
- Platform: [Immunefi / HackerOne / etc.]
- Bounty range: [e.g., $1k–$100k]
- Contracts in scope: [list contracts or GitHub links]
- Known issues: Do NOT report any issues already disclosed in previous audits or shared publicly.
- Previous audits: [list or link previous audit reports]
- Additional constraints: [any program-specific rules]

## Severity Rules

ONLY report Critical and High severity vulnerabilities. In bug bounties, the only things that matter are:
- **Direct loss of user funds**
- **Permanent freezing of funds**
- **Protocol insolvency**
- **Breaking core protocol functionality**

Do NOT waste time on Mediums, Lows, gas optimizations, or informational findings. If it doesn't lead to fund loss or protocol breakage, skip it.

## Trust Model

- Admin/owner is a TRUSTED entity. Check specs/docs for other trusted roles/actors.
- Do NOT report issues that require trusted actors to act maliciously.
- DO report logic or code bugs in trusted actor flows — if an admin function has a genuine bug that produces unintended behavior, that's valid.
- Evaluate blast radius: if a compromised admin can rug the entire protocol with no timelock or safeguard, that may qualify as a centralization risk worth reporting (check program rules).

## Hunting Instructions

Analyze the following contract(s) with a single focus: find exploitable Critical/High bugs.

[Paste contract code here]

Read and apply the full review checklist (sections 1–14) and the Verifier from the file `common/review-checklist.md`. Follow every check in that checklist against the contract code above.
```

> **Note:** If your AI tool has file access (Cursor, Cline, etc.), the AI will read the checklist automatically. If you're using ChatGPT, Claude web, or similar — attach `common/review-checklist.md` as a file alongside this prompt.

## Usage Tips

- Focus on contracts where user funds flow — vaults, pools, bridges, token contracts.
- Check the program's "Impacts in Scope" section to know exactly what qualifies for payout.
- One valid Critical is worth more than ten questionable Mediums. Go deep, not wide.
- Always write a PoC — reports without proof get rejected on Immunefi and similar platforms.
