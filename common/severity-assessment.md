# Severity Assessment

## Purpose

Use this prompt to objectively assess the severity of a finding using a standardized framework, avoiding the common mistake of over- or under-rating issues.

## Prompt

```
You are a smart contract security auditor assessing the severity of a vulnerability. Apply the following framework:

**Finding:**
- **Description:** [what is the bug]
- **Affected contract/function:** [where]
- **Impact:** [what can go wrong]
- **Preconditions:** [what must be true]
- **Attacker requirements:** [privileges, capital, timing needed]

**Severity Matrix:**

| | High Impact | Medium Impact | Low Impact |
|---|---|---|---|
| **High Likelihood** | Critical | High | Medium |
| **Medium Likelihood** | High | Medium | Low |
| **Low Likelihood** | Medium | Low | Informational |

**Impact Assessment (choose one):**
- **High:** Direct theft/loss of user funds, permanent freezing of funds >1%, protocol insolvency
- **Medium:** Temporary freezing of funds, griefing with material cost to users, protocol disruption
- **Low:** Informational leakage, minor inconvenience, theoretical concern

**Likelihood Assessment (choose one):**
- **High:** No special conditions required, anyone can exploit, low cost to attack
- **Medium:** Requires specific protocol state, moderate capital, or timing
- **Low:** Requires unlikely conditions, admin error, specific token behavior, or extremely high cost

**Provide:**
1. Impact level with justification
2. Likelihood level with justification
3. Final severity from the matrix
4. Comparison to similar historical findings and how they were rated
5. Counterarguments — why might someone rate this differently?
```

## Usage Tips

- Always consider the "so what?" — a bug that looks scary but has no realistic exploitation path is not High severity.
- Cross-reference with platform-specific severity guidelines (Immunefi, Code4rena, Sherlock all differ slightly).
- When in doubt, argue both sides and let the facts decide.
