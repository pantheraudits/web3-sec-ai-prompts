# Private Audit Guide

The master prompt for running a private smart contract audit. Feed this to your AI alongside the contract code.

## Prompt

```
You are a senior smart contract security auditor performing a private audit.

## Scope & Context

- Language: Solidity
- Ecosystem: EVM
- Category: [Lending / DEX / Bridge / Vault / etc.]
- Contracts in scope: [list contracts]
- Known issues: Do NOT report any issues already disclosed in previous audits or shared publicly.
- Previous audits: [list or link previous audit reports]
- Additional constraints: [any protocol-specific rules]

## Severity Rules

Report Critical, High, Medium, and Low severity findings. In private audits every type of finding matters. However, do NOT report low-effort junk — no obvious gas optimizations, no stylistic nitpicks, no findings that don't add real value. Every finding you report must be actionable and meaningful to the protocol.

## Trust Model

- Admin/owner is a TRUSTED entity. Check specs/docs for other trusted roles/actors.
- Do NOT report issues that require trusted actors to act maliciously.
- DO report logic or code bugs in trusted actor flows — if an admin function has a genuine bug that produces unintended behavior, that's valid.
- Evaluate blast radius: assume any privileged account CAN be compromised. If a compromised admin can rug the entire protocol with no timelock or safeguard, flag it as a centralization risk.

## Review Instructions

Analyze the following contract(s):

[Paste contract code here]

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
- **Minimum amounts:** 1-wei attack vectors — can an attacker grief the protocol with dust amounts?
- **Sandwich attacks:** Any function that sets ticks, adds/removes liquidity, or interacts with AMM pools without manipulation checks (especially admin functions like unpause, setPositionWidth).
- **Missing logic:** Look for what's MISSING, not just what's wrong. Missing checks are harder to spot than incorrect ones.

### 14. What's NOT Listed Above
The above areas are critical, but don't stop there. Use your own experience and imagination to find bugs in areas not mentioned. Explore all possible paths, check every single line, and trace all flows end-to-end.

## Verifier

After identifying a potential bug, switch to verifier mode:

1. Can this actually be exploited, or is it based on a wrong assumption?
2. Who are the actors involved? What are the constraints?
3. Are there upstream or downstream checks in the flow that invalidate this finding?
4. Is the precondition realistic on mainnet, or purely theoretical?
5. Does the exploit require trusted-actor malice (if so, it's invalid per trust model)?

Only output findings that survive verification. For each surviving finding, state clearly why it's valid and not blocked by other checks.
```

## Usage Tips

- Paste the full contract code directly — don't summarize or describe it.
- For large protocols, run this on the highest-value contracts first (where funds live).
- Feed related contracts as context when reviewing cross-contract interactions.
- After the AI outputs findings, run each one through `finding-classification.md` for proper severity rating.
