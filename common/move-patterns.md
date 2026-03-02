# Move Vulnerability Patterns (Sui / Aptos / Move)

## Purpose

Use this prompt to check a Move contract against the most common vulnerability patterns found across 200+ public Move audit reports (1141 findings). Covers Sui, Aptos, Supra, and other Move-based chains.

Based on data from the [Move Vulnerability Database](https://movemaverick.github.io/move-vulnerability-database/) by MoveMaverick.

## Prompt

```
You are a Move smart contract security expert. Review the following contract and check for these vulnerability patterns, derived from 1141 real findings across 200+ audited Move protocols.

[Paste contract code or reference file path]

**The top 5 vulnerability classes account for 70%+ of all Critical/High findings in Move. Check them first.**

### 1. Business Logic (296 findings, 21 Critical, 58 High)

The #1 source of bugs in Move protocols. Check:

- **Reward/staking timing exploits** — Can a new user deposit and immediately claim rewards accumulated before they joined? Is there a timelock or snapshot mechanism? (Found in Dola, Thala Labs, Walrus)
- **Flash loan reward manipulation** — Can an attacker flash-loan to inflate their stake, claim disproportionate rewards, then unstake in the same transaction? Are accumulators updated BEFORE stake amount changes? (Found in Thala Labs — both 2022 and 2023)
- **Liquidation logic flaws** — Can an attacker liquidate themselves for profit? Can valid positions be liquidated using incorrect configurations? Are missing asserts allowing wrong config objects? (Found in Kai Finance, Thala Labs)
- **Partial close/withdrawal bypasses** — Does partial position closing use full collateral instead of proportional collateral for cap calculations? Can profit caps be bypassed by splitting actions? (Found in Dexlyn Perp DEX)
- **Pool creation validation** — Can anyone create a pool with identical token pairs (X/X)? Can pools be created with fake tokens that map to legitimate addresses? (Found in Pools Finance, Dola)
- **Constant product invariant breaks** — After fees or flash loans, does the AMM invariant (x*y=k) still hold? Can repeated swaps with fee deductions progressively drain the pool? (Found in Baptswap)
- **State reset on update** — Do admin update functions accidentally reset all fields to zero/default instead of only updating specified parameters? (Found in Superposition)
- **Queue/tree data structure bugs** — If the protocol uses custom data structures (splay trees, linked lists, queues), are insert/remove/enqueue operations correct in all edge cases? Especially after deletions? (Found in Laminar Markets — 3 critical bugs in data structures)

### 2. Input Validation (170 findings, 16 Critical, 29 High)

- **Missing generic type checks** — Does the function verify that the generic type parameter matches the expected type stored in the resource? Can an attacker pass CoinType X when the pool expects CoinType Y? This is Move-specific and extremely common. (Found in Econia, Navi, AquaSwap, Dexlyn — multiple criticals)
- **Missing UID/object validation** — Are object IDs validated before use? Can an attacker forge or substitute objects (BankV2, receipts, configs) with invalid ones? (Found in Bluefin)
- **Flash loan receipt manipulation** — Does the repay function verify that the receipt's order_id matches the original order? Can an attacker use a receipt from one loan to repay a different one? (Found in Cetus)
- **Zero-value inputs** — Can zero-value bids/deposits/amounts cause division by zero, DoS, or bypass checks? Does `split(0)` revert and block downstream operations? (Found in MoviePass Exchange)
- **Arbitrary asset repayment** — Do repay/claim functions validate that the provided token matches the pool's configured asset? Can an attacker repay a flash loan with worthless tokens? (Found in Dexlyn — 3 critical findings for flash swap, liquidity, and rewards)
- **Signature validation** — Are signature bytes length-checked? Are the correct parameters used for digest generation (nonce vs merkle_index)? Can extra bytes alter the computed hash? (Found in Bluefin, Dexlyn Hyperlane)
- **Uncallable functions** — Can required functions actually be called? Are there parameters (like shared objects) that cannot be passed by users, making functions permanently inaccessible? (Found in SuiPad)

### 3. Calculation Errors (148 findings, 13 Critical, 28 High)

- **Precision/decimal mismatches** — Are token amounts converted correctly between different decimal precisions? Is the collateral amount adjusted for the token's actual decimals during conversions? (Found in Bucket, Navi)
- **Scaled vs unscaled mixing** — Are scaled balances (supply/borrow) mixed with unscaled amounts in the same calculation? This causes massive inaccuracies. (Found in Navi)
- **Time constant errors** — Is `SECONDS_PER_YEAR` calculated correctly? Using milliseconds instead of seconds makes the value 1000x too large. (Found in Navi)
- **Double scaling** — Is a value upscaled twice in the same flow? Especially in flash loan repayment paths where meta-stable pool values are multiplied by oracle-derived rates. (Found in Thala Swap)
- **Share price manipulation** — Can the first depositor inflate share price via rounding? Are token-to-share conversions safe against donation/inflation attacks? (Found in Bluefin Spot)
- **Arithmetic overflow** — Can reserve manipulation (via flash loans) cause intermediate calculations to overflow when converting to the protocol's fixed-point format? (Found in Pools Finance)
- **Formula errors** — Are AMM formulas (add_liquidity, stable curve invariant) implemented correctly? Is the order of parameters correct? (Found in KriyaDEX, Cetus, Pontem/Liquidswap)
- **Missing rewarder updates** — Does removing liquidity call `update_rewarder` first? If not, reward accounting becomes permanently corrupted. (Found in Cetus Concentrated Liquidity)
- **Refund precision** — After repayment, is the excess amount converted back to the correct decimal precision before returning to the user? (Found in Navi)

### 4. Access Control (73 findings, 13 Critical, 20 High)

- **Public function visibility** — Are sensitive functions marked `public` when they should be `public(package)` or `public(friend)`? In Move, function visibility is the primary access control mechanism. Check EVERY public function. (Found in Creek Finance, Navi, Superposition, Aftermath)
- **Missing capability checks** — Are capability objects (AdminCap, MinterCap, etc.) required and verified before privileged operations? Can capabilities be forged by creating overlapping IDs? (Found in Argo — MeterCap forgery)
- **Resource signer exposure** — Is the `resource_signer` function (used to generate resource accounts for token storage) properly restricted? If any module can call it, they can access stored tokens. (Found in Eternal Finance, Project Z)
- **Liquidation access control** — Is the liquidation flow's capability requirement consistently enforced across all entry points? (Found in Argo — liquidate_repay bypassed access control)
- **Test code in production** — Is test code properly gated with `#[test_only]`? Ungated test functions can give anyone admin privileges. (Found in SuiPad)
- **Pool creation permissions** — Can anyone create pools, or is it restricted to admins? Unrestricted pool creation can enable fake deposit/withdraw messages via bridges. (Found in Dola)
- **Front-running via public minting** — Can attackers front-run minting transactions to manipulate ordering and capture rewards? (Found in Lombard Finance)

### 5. State Management (64 findings, 7 Critical, 14 High)

- **Stale state dependencies** — Are epoch checks complete? (e.g., checking `withdraw_epoch` but not `activation_epoch` causes reward calculation errors) (Found in Walrus)
- **Incorrect index tracking** — Do removal functions correctly swap-and-pop, or do they accidentally overwrite the target index with itself (removing the wrong element)? (Found in Aptos Labs Securitize)
- **Tail pointer corruption** — When removing nodes from linked structures, is the tail pointer updated? Missing tail updates corrupt all subsequent operations. (Found in Laminar Markets)
- **Accumulator ordering** — Are reward accumulators updated BEFORE modifying stake amounts? Wrong ordering allows flash loan exploitation. (Found in Thala Labs)
- **Timestamp manipulation** — Can anyone set arbitrary timestamps on critical state variables via public setter functions? (Found in Superposition)
- **Recording zero values** — Do creation functions accidentally hardcode zero instead of using the provided parameter? This silently corrupts all stored records. (Found in Aptos Labs Securitize)

### 6. Oracle Issues (27 findings, 3 Critical, 5 High)

- Stale price acceptance — missing freshness/staleness checks
- Price manipulation via low-liquidity sources
- Incorrect decimal scaling between oracle and consumer
- Missing circuit breaker / deviation bounds

### 7. Denial of Service (40 findings, 2 Critical, 4 High)

- Unbounded loops over dynamic collections
- Single bad entry blocking batch operations (one zero-value bid blocks all withdrawals)
- Arithmetic overflow causing function-level DoS

### 8. Data Inconsistency (31 findings, 2 Critical, 10 High)

- State updates not atomic across related variables
- Events emitting incorrect/stale values
- Cross-module state assumptions breaking after independent updates

### 9. Constant Definition (21 findings, 3 Critical, 2 High)

- Wrong constant values (SECONDS_PER_YEAR in milliseconds, wrong precision factors)
- Constants not matching documentation/specs
- Hardcoded values that should be configurable

### 10. Front-Running (7 findings, 0 Critical, 3 High)

- Transaction ordering manipulation on Sui/Aptos
- Payload front-running to block other users' operations
- Missing commit-reveal for sensitive operations

For each pattern found:
1. State the specific vulnerability class from the list above
2. Indicate severity (Critical/High/Medium/Low) with justification
3. Point to the exact code location
4. Describe the exploit scenario
5. Reference similar historical findings from Move audits if applicable
```

## Usage Tips

- The **top 3 patterns** (business logic, input validation, calculation errors) account for 53% of all findings and 62% of all Critical/High findings. Spend most of your time there.
- **Generic type validation** is the single most Move-specific pattern. Solidity doesn't have this class of bug. Always check every function that accepts a generic type parameter.
- **Function visibility** in Move is fundamentally different from Solidity. `public` means any module can call it — there's no `msg.sender` equivalent by default. Review every `public` function.
- Use alongside `common/review-checklist.md` — the generic checklist covers cross-language patterns (arithmetic, access control, asymmetry), while this file covers Move-specific gotchas.
- Reference the full [Move Vulnerability Database](https://movemaverick.github.io/move-vulnerability-database/) for detailed examples of each pattern.
