# Review Checklist

The shared review checklist used by `bug-bounty/hunting-guide.md`, `private-audits/audit-guide.md`, and any other engagement type. Update here, it applies everywhere.

## Checklist

```
### 1. Constants & Immutables
- Check every constant defined in the contract. Is the value correct?
- Trace its usage across the codebase. Is it logically correct in every formula?
- Verify the math and formulas where constants are used — check for precision errors, wrong units, or incorrect scaling.

### 2. State Variables
- Enumerate all state variables. For each one:
  - Where is it read? Where is it written?
  - Is it updated correctly in every path that should update it?
  - Can an attacker manipulate or exploit any state through unexpected ordering, reentrancy, or missing updates?

### 3. Access Control
- Identify ALL privileged functions.
- Don't just check "modifier is present." Verify the logic actually ties the caller to the specific resource (e.g., is msg.sender the admin of THIS fund, not just any admin?).
- Principle of Least Privilege: no actor should have more power than needed. If an API exists to fetch a value, a bot shouldn't be able to pass it as an arbitrary parameter.
- Defense in Depth: if a privileged account is compromised, what's the worst-case damage?

### 4. Asymmetry Detection
- Compare matching function pairs side-by-side:
  - deposit vs withdraw
  - buy vs sell
  - mint vs burn
  - borrow vs repay
  - user version vs admin version of the same action
- Look for functions that use an oracle/API in one path but accept an arbitrary parameter in the other.
- Admin functions are underrepresented in testing and frequently contain critical bugs — scrutinize them harder.

### 5. Bad Symmetry
- A safety check duplicated from a "prepare" function into a "redeem/claim" function can cause permanent DoS if the prepare step already decremented a counter to zero.
- Defensive code can itself be a critical vulnerability — overly restrictive checks can brick functionality.

### 6. Input Validation
- Check every external/public function for:
  - Identical inputs (same address passed for from/to)
  - Empty lists (loop skipped → returns default true/false)
  - Zero values, 1 wei values, max uint values
  - Unvalidated addresses (can a user pass a malicious contract that implements the expected interface?)
- Loop bypass: if a for-loop iterates over a user-supplied list and returns true/false, what happens with an empty list?

### 7. Setters & Update Functions
- For every setter/update: does changing this state variable retroactively impact any live position, loan, stake, or pending operation?
- If a function allows changing a token address, does the new token have different decimals? Will that corrupt internal accounting?
- If a function updates an external contract address, does it first reclaim tokens/allowances from the old address?
- When adding items to lists, does it check for duplicates? Duplicates can break downstream logic.

### 8. Unchecked Return Values
- Check ALL external calls for unchecked return values.
- Especially: OpenZeppelin's EnumerableSet (returns bool, doesn't revert), low-level calls, ERC20 approve/transfer.

### 9. Arithmetic & Overflow
- Multiplying two uint32s stores the result in uint32 even if assigned to uint256 — must cast before multiplying.
- Even uint256 multiplication can overflow with large numbers.
- Division before multiplication causes precision loss.
- Check rounding direction — does it favor the attacker or the protocol?

### 10. Storage vs Memory
- When copying storage to memory before deleting: verify the code resets storage (not memory).
- After resetting storage, verify subsequent reads use memory (not the now-zeroed storage).

### 11. Precision & Decimals Mismatch
- Trace every variable's decimal precision through the full flow.
- Look for addition/subtraction between variables of different precision.
- Common pattern: protocol uses 18 decimals internally but interacts with USDC (6 decimals) — check for missing or incorrect conversion at every boundary.

### 12. Copy-Paste Errors
- When you see similar constants, hashes, or storage slots — verify they're actually different.
- A coding pattern present everywhere except one place is an asymmetry — the missing instance is likely a bug.

### 13. General Heuristics
- **Many small ops vs one large op:** Do many small deposits/withdrawals produce the same end state as one large one? If not, there's a rounding, fee, or state corruption bug.
- **Off-by-one:** `<` vs `<=`, especially with slightly different checks in different places.
- **Minimum amounts:** 1-wei attack vectors — can an attacker grief or drain the protocol with dust amounts?
- **Sandwich attacks:** Any function that sets ticks, adds/removes liquidity, or interacts with AMM pools without manipulation checks (especially admin functions like unpause, setPositionWidth).
- **Spending allowances / resource recovery:** If a function updates an external contract address, does it first reclaim tokens/allowances from the old address?
- **Token address swaps:** If a function allows changing a token address, does the new token have different decimals? Will that corrupt internal accounting?
- **Duplicate values in lists:** When adding new tokens/addresses, does it check for duplicates?
- **Missing logic:** Look for what's MISSING, not just what's wrong. Missing checks are harder to spot than incorrect ones.

### 14. Forked Protocols (skip if not a fork)
If this protocol is a fork of an existing protocol, this section is critical:
- **Identify all custom changes.** Diff the fork against the original — every modification is a sensitive area where bugs are likely introduced.
- **Understand why each change was made.** Does the change introduce new state, new roles, new token interactions, or altered math? Has it introduced any new issues?
- **Check the original protocol's security considerations.** Are there security recommendations or invariants from the original that this fork has NOT implemented?
- **Check the original protocol's known issues.** Are there any disclosed bugs or unresolved issues in the original that also affect this fork?
- **Inherited vulnerabilities.** A fork inherits not just security from an established protocol, but also its vulnerabilities. "Minor" modifications — custom proxies, oracle mechanisms, admin controls, fee logic — are frequently the source of new critical exploits.
- **Upgraded dependencies.** Did the fork change compiler version, OpenZeppelin version, or other dependencies? Check for breaking changes or deprecated patterns.

### 15. What's NOT Listed Above
The above areas are critical, but don't stop there. Use your own experience and imagination to find bugs in areas not mentioned. Explore all possible paths, check every single line, and trace all flows end-to-end.
```

## Verifier

Append this after the review checklist in your prompt:

```
## Verifier

After identifying a potential bug, switch to verifier mode:

1. Can this actually be exploited, or is it based on a wrong assumption?
2. Who are the actors involved? What are the constraints?
3. Are there upstream or downstream checks in the flow that invalidate this finding?
4. Is the precondition realistic on mainnet, or purely theoretical?
5. Does the exploit require trusted-actor malice (if so, it's invalid per trust model)?
6. Does the exploit require conditions excluded by the program's scope/rules?

Only output findings that survive verification. For each finding, state clearly why it's valid and not blocked by other checks in the system.
```
