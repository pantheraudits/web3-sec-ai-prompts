# Report Writing

## Purpose

Use this prompt to draft a clear, high-quality bug bounty report that maximizes your chances of acceptance and fair severity classification.

## Prompt

```
You are an experienced Web3 bug bounty hunter writing a vulnerability report for [Platform: Immunefi / HackerOne / etc.].

Help me write a professional report for the following finding:

- **Protocol:** [Protocol Name]
- **Contract:** [Contract name and address]
- **Function(s) affected:** [function names]
- **Vulnerability type:** [e.g., reentrancy, price manipulation, access control]
- **Severity (my assessment):** [Critical / High / Medium / Low]
- **Brief description:** [1-2 sentences on what's wrong]

Structure the report with:
1. **Title** — concise, descriptive (e.g., "Reentrancy in withdraw() allows draining of pool funds")
2. **Severity** — with justification referencing the program's impact categories
3. **Description** — clear explanation of the root cause
4. **Impact** — what damage can an attacker do? quantify if possible
5. **Proof of Concept** — step-by-step exploit flow (or Foundry test if applicable)
6. **Recommended Fix** — how should the team remediate this?

Keep the tone professional and factual. Avoid speculation — only state what you can prove.
```

## Usage Tips

- Always include a working PoC or detailed step-by-step — reports without proof get rejected.
- Match severity to the program's own impact definitions, not your personal opinion.
- Be concise. Triagers read hundreds of reports — clarity wins.
