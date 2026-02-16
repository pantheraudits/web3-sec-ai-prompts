# Code Review

## Purpose

Use this prompt to perform a systematic, line-by-line security review of a smart contract during a private audit engagement.

## Prompt

```
You are a senior smart contract security auditor performing a private audit. Review the following Solidity contract for security vulnerabilities:

[Paste contract code or provide file path]

Perform the review in these phases:

**Phase 1 — Architecture Overview**
- What does this contract do?
- What are the trust assumptions?
- Who are the privileged roles and what can they do?
- What external contracts/tokens does it interact with?

**Phase 2 — Line-by-Line Analysis**
For each function, check for:
- Reentrancy (state changes after external calls)
- Integer overflow/underflow (even with Solidity 0.8+, check unchecked blocks)
- Access control gaps
- Input validation (zero address, zero amount, array length mismatches)
- Oracle manipulation or stale data
- Front-running / MEV opportunities
- Precision loss in math operations
- Incorrect use of msg.value in loops
- Storage vs. memory misuse
- Delegatecall safety
- ERC20 transfer return value handling

**Phase 3 — Cross-Contract Risks**
- Can external tokens/contracts cause unexpected behavior?
- Are there composability risks with DeFi protocols?
- flash loan attack vectors?

**Phase 4 — Findings Summary**
List all findings with severity (Critical / High / Medium / Low / Informational) and recommended fixes.
```

## Usage Tips

- Feed contracts one at a time for focused analysis.
- For large codebases, start with the highest-value contracts (vaults, pools, bridges).
