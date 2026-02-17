# Multi-Expert Review

A 3-pass adversarial review system that forces independent analysis from different angles, then cross-validates to kill false positives. Use this instead of (or alongside) the standard single-pass audit prompts for higher-confidence findings.

## Why Multiple Passes?

A single-pass review inherits the biases of whatever angle the AI starts from. If it begins with reentrancy, it tends to see reentrancy everywhere. Three independent passes with different focuses produce:
- Findings that survive adversarial scrutiny (higher confidence)
- Coverage across technical, economic, and integration attack surfaces
- A built-in false-positive filter (the triager in Pass 3)

## Prompt

```
You are performing a multi-expert security review of a smart contract. Execute THREE independent analysis passes, then synthesize.

## Context

- Language: [Solidity / Vyper / Rust-Anchor / etc.]
- Ecosystem: [EVM / Solana / Cosmos / etc.]
- Protocol type: [Lending / DEX / Bridge / Vault / etc.]
- Contracts in scope: [list contracts]
- Known issues: [list any disclosed issues to skip]

## Contract Code

[Paste contract code here]

---

## Pass 1: Systematic Auditor

You are a methodical security auditor. Work through the code systematically.

**Focus areas:**
- Start with highest-risk functions: payable, external calls, admin/privileged functions
- Map ALL fund flow paths — where do tokens enter, move, and exit?
- Map ALL state changes — which functions modify which storage variables?
- Check every external call for reentrancy, return value handling, and gas griefing
- Check arithmetic for overflow, underflow, precision loss, rounding direction
- Check access control on every state-changing function
- Check every constant and hardcoded value against docs/comments

**Output format for each finding:**
- Location (contract:function:line)
- What's wrong
- Who can exploit it and how
- Severity estimate

State "--- END PASS 1 ---" when complete.

---

## Pass 2: Economic & Integration Attacker

You are an attacker looking for economic exploits and composability issues. DO NOT reference Pass 1 — start completely fresh from the code.

**Focus areas:**
- Single-transaction profit opportunities (flash loans, price manipulation, sandwich)
- Multi-step economic attacks (governance manipulation, liquidation cascades, bad debt creation)
- Cross-contract interaction risks (callbacks, reentrancy via integrations, trust assumptions about external protocols)
- Token economics misalignment (share price manipulation, donation attacks, first-depositor inflation)
- MEV extraction opportunities (front-running, back-running, JIT liquidity)
- Oracle dependency risks (stale prices, manipulation, L2 sequencer downtime)

**For each attack scenario, tell TWO stories:**
1. INNOCENT USER: What does the normal user expect to happen? (e.g., deposits 100 USDC → expects proportional LP tokens)
2. ATTACKER: What can the attacker do to exploit this? (e.g., flash loan → inflate share price → innocent user gets fewer shares → attacker profits)

**After your independent review:**
- List your own findings first
- Then review Pass 1 findings — state what you AGREE with, what you DISAGREE with, and why
- Flag any Pass 1 findings you believe are false positives

State "--- END PASS 2 ---" when complete.

---

## Pass 3: Skeptical Triager

You are a skeptic whose job is to DISPROVE findings. You protect the protocol team from wasting time on invalid reports.

**For EVERY finding from Pass 1 and Pass 2, answer:**
1. Can you construct a concrete proof that this is NOT exploitable?
2. Are there upstream or downstream checks that prevent exploitation?
3. Is the attack economically rational? (cost to attack vs profit, including gas, slippage, oracle costs)
4. Does the exploit require unrealistic preconditions? (e.g., specific token behavior, admin malice, exact timing)
5. Would existing protections (reentrancy guards, timelocks, slippage checks) block it?

**Label each finding:**
- **VALID** — Exploitable with realistic conditions. Include in final report.
- **QUESTIONABLE** — Possible but uncertain. Needs manual verification or PoC.
- **DISMISSED** — Not exploitable, blocked by existing protections, or economically irrational. Explain why.
- **OVERCLASSIFIED** — Real issue but severity is too high. Suggest correct severity with reasoning.

State "--- END PASS 3 ---" when complete.

---

## Final Consolidated Report

After all three passes, produce the final report containing ONLY findings labeled VALID or QUESTIONABLE by the Triager.

**Format each finding as:**

### [SEVERITY]-[NUMBER]: [Title]

**Status:** VALID / QUESTIONABLE
**Found by:** Pass 1 / Pass 2 / Both
**Triager assessment:** [Why this survived scrutiny]

**Impact:** [What goes wrong]
**Attack flow:**
1. [Step 1 — setup]
2. [Step 2 — trigger]
3. [Step 3 — exploit]
4. [Step 4 — profit/damage]

**Location:** [contract.sol:function:lines]
**Severity:** [Critical/High/Medium/Low] — Impact × Likelihood × Exploitability
**Recommended fix:** [Specific code-level remediation]

Sort findings: Critical → High → Medium → Low. Within each severity, VALID before QUESTIONABLE.
```

## Usage Tips

- This produces significantly better results than a single-pass review, but uses more tokens. Use it on your highest-value contracts (where funds live).
- For large contracts, run this on one contract at a time — don't feed multiple contracts into a single multi-expert run.
- The Pass 2 "innocent user vs attacker" story format is powerful even outside this prompt — use it whenever you need to articulate an attack scenario.
- If you've already built a custom primer (`common/custom-primer.md`), feed it as additional context after the contract code. The primer entries will sharpen all three passes.
- After the AI finishes, any finding marked QUESTIONABLE is your priority for manual PoC work — the AI couldn't disprove it but also couldn't fully confirm it.
