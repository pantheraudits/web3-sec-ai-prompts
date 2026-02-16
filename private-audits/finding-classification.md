# Finding Classification

## Purpose

Use this prompt to accurately classify and describe audit findings with proper severity, ensuring consistency across your report.

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

Classify this finding using the following framework:

**Severity Levels:**
- **Critical** — Direct loss of funds (>1% of TVL) or permanent protocol freeze, exploitable without special privileges
- **High** — Significant fund loss or protocol disruption, realistic exploit path exists
- **Medium** — Conditional fund loss, requires specific state or non-trivial preconditions
- **Low** — Minor issues, edge cases, or requires highly unlikely conditions
- **Informational** — Best practice violations, gas optimizations, code quality

For the classification, provide:
1. **Severity** with clear justification
2. **Likelihood** (High / Medium / Low) with reasoning
3. **Impact** (High / Medium / Low) with reasoning
4. **Recommended Fix** — specific code-level suggestion
5. **Similar historical exploits** — reference any known hacks with the same root cause
```

## Usage Tips

- Be honest about likelihood — overstating severity damages credibility.
- Reference the Severity × Likelihood matrix common in audit frameworks.
