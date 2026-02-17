# Finding Classification

## Purpose

Use this prompt to accurately classify, describe, and structure audit findings with proper severity, attack flows, and economic analysis — ensuring consistency and credibility across your report.

## Prompt

```
You are a smart contract security auditor classifying a finding for a private audit report.

Here is the finding:

- **Contract:** [contract name]
- **Function:** [function name]
- **Issue:** [describe the bug]
- **Impact:** [what can go wrong]
- **Likelihood:** [how likely is exploitation]
- **Preconditions:** [what must be true for the exploit to work]

Classify and structure this finding using the framework below.

---

## Severity Classification

**Severity Levels:**
- **Critical** — Direct loss of funds (>1% of TVL) or permanent protocol freeze, exploitable without special privileges
- **High** — Significant fund loss or protocol disruption, realistic exploit path exists
- **Medium** — Conditional fund loss, requires specific state or non-trivial preconditions
- **Low** — Minor issues, edge cases, or requires highly unlikely conditions
- **Informational** — Best practice violations, gas optimizations, code quality

**Quantitative check:** Apply SEVERITY = Impact(1-3) × Likelihood(1-3) × Exploitability(1-3). See `common/severity-assessment.md` for the full formula. If borderline, round DOWN.

---

## Output Format

Structure the finding exactly as follows:

### [C/H/M/L]-[Number]: [Impact] via [Weakness] in [Feature]

**Severity:** [Critical / High / Medium / Low]
**Likelihood:** [High / Medium / Low] — [one-line reasoning]
**Impact:** [High / Medium / Low] — [one-line reasoning]
**Confidence:** [High / Medium / Low] — [how certain are you this is real?]
**Location:** [contract.sol:function:lines XX-YY]

**Innocent User Story:**
[What does a normal user expect to happen?]
Example: "Alice deposits 100 USDC into the vault. She expects to receive proportional shares and later withdraw her deposit plus yield."

**Attack Story:**
[What does the attacker actually do? Step-by-step.]

**Attack Flow:**
1. [Step 1 — Setup / preconditions]
2. [Step 2 — Trigger the vulnerability]
3. [Step 3 — Exploit / extract value]
4. [Step 4 — Profit / damage quantification]

**Economic Analysis:**
- Attack cost: [gas + capital + slippage + opportunity cost]
- Attack profit: [stolen funds / damage caused]
- Profitable: [Yes/No — is profit > cost?]

**Recommended Fix:**
[Specific code-level suggestion — show the before/after or describe the exact change needed]

**Similar Historical Exploits:**
[Reference known hacks with the same root cause — link to post-mortem if available]

---

Provide:
1. The fully structured finding using the format above
2. Clear justification for the severity rating
3. Any counterarguments — why might someone rate this differently?
```

## Usage Tips

- The "Innocent User Story vs Attack Story" format forces concrete exploit articulation — if you can't write a specific attack story, the finding probably isn't real.
- The title format `[C/H/M/L]-[Number]: [Impact] via [Weakness] in [Feature]` makes findings scannable and sortable.
- Always include economic analysis — it catches overclassified findings where the attack costs more than it profits.
- Be honest about likelihood — overstating severity damages credibility.
- Reference the quantitative formula in `common/severity-assessment.md` for consistent scoring.
