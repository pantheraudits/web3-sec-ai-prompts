# Solana Vulnerability Patterns (Rust / Anchor / Native)

## Purpose

Use this prompt to check a Solana program against the most critical vulnerability patterns found across real exploits and audit reports. Covers both Anchor and native Rust programs.

Informed by [Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder) by Frank Castle (70+ Rust / 50+ Solana audits) and Trail of Bits Solana security research.

## Prompt

```
You are a Solana program security expert. Review the following program and check for these vulnerability patterns, derived from real Solana exploits and audit findings.

[Paste program code or reference file path]

**Solana's account model creates fundamentally different vulnerability classes than EVM. The top attack vectors are account validation failures, PDA misuse, and CPI trust boundary violations. Check them first.**

### 1. Account Validation (Most Critical — Wormhole $326M, Cashio $48M)

Every instruction must validate every account it receives. Solana does NOT enforce this automatically.

- **Missing signer check** — Does the instruction verify `account.is_signer` for all accounts that should authorize the action? In Anchor, is `Signer<'info>` or `#[account(signer)]` used? (Ref: Wormhole $326M — missing signer on guardian set)
- **Missing owner check** — Does the instruction verify `account.owner == expected_program_id`? Can an attacker pass an account owned by a different program with crafted data? In Anchor, does the `Account<'info, T>` type enforce this?
- **Type cosplay / missing discriminator** — Can an attacker pass an account of type A where type B is expected? Does the program check the 8-byte Anchor discriminator or a manual type tag? (Ref: Cashio $48M — attacker forged account type)
- **Missing reinitialization guard** — Can an already-initialized account be re-initialized to overwrite its data? Is `init` used (which checks for prior initialization) or does the code manually check an `is_initialized` flag?
- **Missing writable check** — Are accounts that should be read-only validated as non-writable? Can an attacker pass a writable version of an account that should be immutable?
- **Unchecked account substitution** — Can an attacker substitute a legitimate account with a different one that happens to pass validation? Are all relevant fields (mint, authority, program ID) cross-checked?

### 2. PDA Security

Program Derived Addresses are deterministic but easy to misuse.

- **Non-canonical bump** — Does the program use `find_program_address` (which returns the canonical bump) or manually derive with an arbitrary bump? Using non-canonical bumps allows multiple valid PDAs for the same seeds. Always store and verify the canonical bump.
- **Seed collision** — Can different logical entities produce the same PDA seeds? Are seeds sufficiently unique (e.g., including user pubkey, mint, and a purpose tag)?
- **PDA sharing across instructions** — Is a PDA used as authority for multiple unrelated operations? Can authority over one operation leak into another?
- **Missing seed components** — Are all relevant identifiers included in PDA seeds? Missing a component (e.g., user pubkey) can allow one user to access another's PDA.
- **Bump seed canonicalization** — In Anchor, is `bump` stored on the account and re-used with `seeds::program`? Using `bump` in `#[account(seeds = [...], bump)]` without specifying the stored bump recomputes it each time (extra compute but functionally safe in Anchor — risky in native).

### 3. CPI Safety (Cross-Program Invocation)

CPI is the Solana equivalent of external calls — and equally dangerous.

- **Arbitrary CPI target** — Is the target program ID hardcoded or validated? Can an attacker pass a malicious program as the CPI target? Always verify `program_id` matches the expected program.
- **Stale account state after CPI** — Does the program read account data AFTER a CPI call without reloading? CPI can modify account data — any cached state is stale post-CPI. Use `reload()` in Anchor.
- **Signer privilege escalation via CPI** — Does a CPI pass through signer privileges that the calling program shouldn't delegate? Especially dangerous with `invoke_signed` — the PDA signer seeds effectively grant the called program authority over the PDA.
- **Post-CPI ownership changes** — After a CPI, does the program verify that account ownership hasn't changed? A malicious CPI target could reassign account ownership.
- **SOL balance changes after CPI** — Does the program check lamport balances after CPI? A malicious program could drain lamports from accounts passed to it.
- **Unchecked CPI return values** — Does the program check for CPI errors? `invoke` and `invoke_signed` return `ProgramResult` — silently ignoring errors can leave the program in an inconsistent state.
- **invoke vs invoke_signed confusion** — Is `invoke_signed` used only when a PDA needs to sign? Using `invoke` when a PDA signature is required will silently fail.
- **Defense-in-depth for CPI** — Even if calling a trusted program, validate post-CPI state. Trusted programs can be upgraded.

### 4. Arithmetic & Precision (Mango Markets $115M)

Rust's release mode DISABLES overflow checks by default. This is critical.

- **Release-mode overflow** — Rust's `checked_add/sub/mul/div` and `overflow-checks = true` in Cargo.toml are the only protections. In release builds, standard `+`, `-`, `*`, `/` silently wrap on overflow. Does the program use checked arithmetic everywhere? (Ref: Mango Markets $115M — price manipulation via overflow)
- **Multiply-before-divide** — Is multiplication done before division to preserve precision? `(a / b) * c` loses precision; `(a * c) / b` is safer.
- **Missing slippage protection** — Do swap/trade instructions include minimum output amount parameters? Can an attacker sandwich the transaction?
- **Lamport invariant violations** — Does the program maintain the invariant that the sum of input lamports equals the sum of output lamports? Failing this allows lamport creation/destruction.
- **Decimal scaling errors** — Are token amounts correctly scaled for tokens with different decimals (e.g., USDC 6 vs SOL 9)? Are conversions between token amounts and internal accounting consistent?
- **Unsafe `as` casts** — Does the program use `as` for numeric type conversions (e.g., `u64 as u32`)? These silently truncate. Use `try_into()` instead.
- **Unchecked `unwrap()` / `expect()`** — Can any `unwrap()` or `expect()` call panic on unexpected input? In on-chain programs, panics abort the transaction but can be used for griefing or DoS.

### 5. Account Lifecycle

- **Insecure account closing** — When closing an account, does the program: (1) zero out all data, (2) transfer all lamports to the recipient, (3) set the owner to the system program? Only transferring lamports leaves the account data intact and potentially reusable.
- **Account revival attack** — After an account is closed (lamports zeroed), can an attacker send lamports back to it in the same transaction to "revive" it with stale data? The runtime garbage-collects accounts with zero lamports only at transaction boundary.
- **Rent exemption issues** — Does the program ensure accounts maintain rent-exempt minimum balance? Accounts below this threshold are eventually garbage-collected, causing unexpected state loss.
- **Sysvar account spoofing** — Does the program verify sysvar accounts (Clock, Rent, etc.) by their known addresses? An attacker can pass a crafted account with fake sysvar data if the address isn't checked. In Anchor, `Sysvar<'info, T>` handles this.

### 6. Token-2022 Compatibility

The Token Extensions program introduces new attack vectors.

- **Missing transfer_checked** — Does the program use `transfer_checked` (which validates decimals) instead of `transfer`? Critical for Token-2022 mints with non-standard decimals.
- **Extension-unaware logic** — Does the program account for Token-2022 extensions (transfer fees, interest-bearing, non-transferable, permanent delegate)? Transfer fees mean received amount < sent amount.
- **Transfer hook reentrancy** — Token-2022 transfer hooks execute arbitrary CPI during transfers — effectively enabling reentrancy on Solana. Does the program assume transfers are atomic?
- **Confidential transfer handling** — If the program interacts with confidential transfer-enabled tokens, does it handle the encrypted balance model correctly?

### 7. Duplicate & Overlapping Accounts

- **Same account passed twice** — Can an attacker pass the same account for two different instruction parameters (e.g., `source` and `destination`)? Does the program check that accounts are distinct where required?
- **remaining_accounts overlap** — If the instruction uses `remaining_accounts`, can an attacker include an account that's already in the named accounts struct? This can bypass validation performed on the named account.

### 8. Input Validation & Instruction Parsing

- **Deserialization safety** — Does the program validate account data length before deserializing? Malformed data can cause panics or read out-of-bounds.
- **Zero-value inputs** — Can zero amounts, zero keys, or empty vectors cause division by zero, no-ops that bypass invariants, or other unexpected behavior?
- **remaining_accounts validation** — If the instruction accepts `remaining_accounts`, is each account validated (owner, type, signer status)? Unvalidated remaining accounts are a common source of arbitrary account injection.
- **Instruction data bounds** — Is instruction data length checked before parsing? Can oversized or undersized data cause unexpected behavior?

### 9. Transaction Model

- **Compute budget exhaustion** — Can an attacker craft inputs that cause the instruction to exceed the 200K (or 1.4M with budget increase) compute unit limit? Unbounded loops over user-controlled data are the typical vector.
- **Transaction size limits** — Does the instruction require more accounts than can fit in a single transaction (max 64 accounts, 1232 bytes)? This can make certain operations impossible.
- **Instruction introspection attacks** — If the program inspects other instructions in the transaction (via `sysvar::instructions`), can an attacker craft surrounding instructions to bypass checks?

### 10. Business Logic (Solana DeFi)

- **Flash loan reward manipulation** — Can an attacker flash-loan tokens, stake, claim accumulated rewards, and unstake in one transaction? Are reward accumulators snapshotted or time-locked?
- **Reward timing exploits** — Can a user deposit immediately before reward distribution and claim a disproportionate share? Is there a minimum stake duration?
- **Frontrunning via transaction ordering** — While Solana doesn't have a traditional mempool, validators can reorder transactions within a slot. Are time-sensitive operations (liquidations, auctions, oracle updates) protected?
- **Cross-program invariant violations** — If the program composing with other programs, are invariants maintained when external state changes between instructions in the same transaction?
- **Price oracle manipulation** — If the program reads prices from on-chain sources (Pyth, Switchboard, Raydium pools), are freshness checks, confidence intervals, and manipulation protections in place?

For each pattern found:
1. State the specific vulnerability class from the list above
2. Indicate severity (Critical/High/Medium/Low) with justification
3. Point to the exact code location (file:function:line)
4. Describe the exploit scenario step by step
5. Note whether the issue is Anchor-specific, native-specific, or both
6. Reference similar historical exploits if applicable
```

## Usage Tips

- **Account validation** (category 1) is the single most Solana-specific vulnerability class. EVM has nothing equivalent — Solana's account model requires explicit validation of every account passed to every instruction. Start here.
- **Release-mode arithmetic** (category 4) is a critical Solana/Rust gotcha. Unlike Solidity 0.8+ which has built-in overflow checks, Rust in release mode silently wraps. Always check `Cargo.toml` for `overflow-checks = true` and look for `checked_*` math.
- **CPI safety** (category 3) is the Solana equivalent of reentrancy — but the attack surface is broader because CPI can modify any account passed to it, not just the calling contract's state.
- **Token-2022** (category 6) is an emerging attack surface. Transfer hooks effectively re-enable reentrancy-like patterns on Solana. Any program interacting with Token-2022 mints needs extra scrutiny.
- Use alongside `common/review-checklist.md` — the generic checklist covers cross-language patterns (business logic, access control, arithmetic), while this file covers Solana-specific gotchas.
- For grep-based surface mapping of Solana codebases, see the Solana section in `common/grep-patterns.md`.
- Reference [Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder) for additional rules and framework-specific details.
