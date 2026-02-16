# Solidity Patterns

## Purpose

Use this prompt to check a contract against common Solidity vulnerability patterns — a rapid-fire checklist for any security review.

## Prompt

```
You are a Solidity security expert. Review the following contract and check for these common vulnerability patterns:

[Paste contract code]

**Checklist:**

1. **Reentrancy**
   - Are there external calls before state updates?
   - Is nonReentrant modifier used where needed?
   - Cross-function reentrancy via shared state?
   - Read-only reentrancy via view functions?

2. **Access Control**
   - Are admin functions properly protected?
   - Can initializers be called more than once?
   - Are there missing zero-address checks on critical setters?

3. **Integer / Math**
   - Unchecked blocks with user-controlled values?
   - Division before multiplication (precision loss)?
   - Rounding direction favoring attacker?
   - Phantom overflow in intermediate calculations?

4. **Token Handling**
   - SafeERC20 used for transfers?
   - Fee-on-transfer token compatibility?
   - Rebasing token compatibility?
   - ERC777 callback risks?
   - Return value of approve not checked?

5. **Oracle / Price**
   - Stale price checks?
   - Manipulable spot prices used (e.g., balanceOf-based)?
   - Missing sequencer uptime checks (L2)?

6. **Flash Loan Vectors**
   - Can any calculation be manipulated within a single transaction?
   - Share price / exchange rate manipulation?

7. **MEV / Front-running**
   - Slippage protection on swaps?
   - Deadline parameters on time-sensitive operations?

8. **Upgradability**
   - Storage layout collisions?
   - Missing gap variables in inherited contracts?
   - Initializer vs constructor confusion?

9. **Gas / DoS**
   - Unbounded loops over user-controlled arrays?
   - Block gas limit reachable?
   - External call failures causing full revert?

10. **Logic**
    - Off-by-one errors in loops or comparisons?
    - Incorrect operator (> vs >=, && vs ||)?
    - Missing return statements?

For each pattern found, indicate severity and the specific code location.
```

## Usage Tips

- Use this as a first-pass scan before deep analysis.
- Not every pattern applies to every contract — focus on what's relevant.
