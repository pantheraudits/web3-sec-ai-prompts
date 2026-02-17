# Severity Assessment

## Purpose

Use this prompt to objectively assess the severity of a finding using a standardized framework, avoiding the common mistake of over- or under-rating issues. The quantitative formula removes gut-feel bias — **when borderline, always round DOWN.**

## Prompt

```
You are a smart contract security auditor assessing the severity of a vulnerability. Apply the following framework:

**Finding:**
- **Description:** [what is the bug]
- **Affected contract/function:** [where]
- **Impact:** [what can go wrong]
- **Preconditions:** [what must be true]
- **Attacker requirements:** [privileges, capital, timing needed]

---

## Quantitative Severity Formula

SEVERITY SCORE = Impact × Likelihood × Exploitability

Rate each dimension 1-3, multiply, then map to severity.

**Impact (how bad is it?):**
- 3 (High): Direct theft/loss of user funds, permanent freezing of funds >1% TVL, protocol insolvency, admin takeover
- 2 (Medium): Temporary freezing of funds, griefing with material cost to users, protocol disruption, partial fund loss with conditions
- 1 (Low): Informational leakage, minor inconvenience, theoretical concern, no direct fund risk

**Likelihood (how likely is it to happen?):**
- 3 (High): Core user flows, no special conditions, anyone can trigger, low cost to attack
- 2 (Medium): Requires specific protocol state, moderate capital, timing, or uncommon but realistic conditions
- 1 (Low): Requires unlikely conditions, admin error, specific token behavior, expert knowledge + precise timing

**Exploitability (how easy is it to execute?):**
- 3 (High): Single transaction, flash loan feasible, no special tooling or privileges needed
- 2 (Medium): Multi-transaction attack, requires some setup, moderate capital, or specific ordering
- 1 (Low): Requires governance attack, multi-day setup, extremely high capital, or coordinated action

**Score → Severity mapping:**
- 18–27: **Critical** — Direct drain, admin takeover, permanent lockup >10% TVL
- 8–17: **High** — Fund loss with common conditions, privilege escalation, significant protocol disruption
- 4–7: **Medium** — Temporary DoS, conditional fund risk, unlikely but possible scenarios
- 1–3: **Low** — Gas issues, theoretical vulnerabilities, minimal real-world impact

**CRITICAL RULE: If borderline between two severities, ALWAYS round DOWN.** Over-classification wastes protocol team time, damages auditor credibility, and dilutes real findings.

---

## Severity Matrix (Quick Reference)

| | High Impact | Medium Impact | Low Impact |
|---|---|---|---|
| **High Likelihood** | Critical | High | Medium |
| **Medium Likelihood** | High | Medium | Low |
| **Low Likelihood** | Medium | Low | Informational |

---

## Provide:

1. Impact score (1-3) with justification
2. Likelihood score (1-3) with justification
3. Exploitability score (1-3) with justification
4. Calculated severity score and resulting severity level
5. Comparison to similar historical findings and how they were rated
6. Counterarguments — why might someone rate this differently? Argue both sides.
7. If the finding is borderline, explicitly state which direction you're rounding and why

## Economic Rationality Check

Before finalizing severity, answer:
- What does the attack cost the attacker? (gas, capital, opportunity cost)
- What does the attacker gain?
- Is profit > cost with reasonable certainty?
- If the answer is no, the finding is OVERCLASSIFIED — reduce severity.
```

## Usage Tips

- Always consider the "so what?" — a bug that looks scary but has no realistic exploitation path is not High severity.
- The quantitative formula prevents the common AI mistake of rating everything as Critical/High. Multiplying three dimensions forces honest assessment.
- Cross-reference with platform-specific severity guidelines (Immunefi, Code4rena, Sherlock all differ slightly).
- When in doubt, argue both sides and let the facts decide.
- For bug bounties, also consider what the platform will actually pay — a "Critical" that Immunefi downgrades to Medium hurts more than a well-argued High.
