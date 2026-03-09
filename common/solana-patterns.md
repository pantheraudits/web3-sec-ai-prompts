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

### 11. Oracle Validation

- **Missing confidence interval check** — Does the program validate that `conf / price` is below a reasonable threshold (e.g., 2–5%)? Wide confidence means the price is unreliable — acting on it enables oracle manipulation.
- **Stale oracle price** — Is the price timestamp checked against a configurable max age? Using a stale feed lets attackers profit from lagging prices.
- **Hardcoded staleness/confidence thresholds** — Are oracle thresholds admin-configurable? Hardcoded values can't adapt to market volatility changes.
- **Retroactive oracle pricing** — Does the program use the current oracle price for positions that were opened at a different price? Store the reference price at action time and use it at settlement — never the live price.

### 12. Fee Completeness

- **Fee bypass on edge-case routes** — Are fees applied to EVERY code path (redemption, withdrawal, single-asset, multi-asset)? Fee bypasses on edge-case routes are a consistent source of protocol drain.
- **Non-atomic fee deduction** — Are fees deducted from tracked totals atomically with the principal deduction? A separate step can be skipped or reordered.
- **Pre/post-fee amount mixing** — Is a consistent amount (pre-fee or post-fee) used for both capacity checks and execution? Mixing them causes overfills or incorrect limit-order behavior.
- **Wrong fee side** — Are fee calculations applied to the input token? Unless the protocol explicitly specifies output-side fees, input-side is safer.

### 13. Token Dust & Time-Limited Account DoS

- **Dust deposit blocking close** — Can an attacker deposit a dust amount to make token account `close` permanently fail? Before closing any token account, sweep or burn the residual balance.
- **Missing post-transfer balance check** — After any transfer, is the account balance reloaded and verified to detect unexpected deposits?
- **No dust threshold** — Is there a defined dust threshold? Dust should be swept to treasury or rejected. Never let dust block settlement or close.
- **Expired accounts left open** — Are time-limited accounts (offers, escrows, locks) closed at expiry? Leaving expired accounts open leaks rent and enables griefing. Allow anyone to trigger closure after expiry.
- **`init_if_needed` on adversary-controlled accounts** — Can an adversary pre-initialize an account with harmful state via `init_if_needed`? Use `init` for one-time initialization instead.

### 14. State Management — Coupled Fields & Counters

- **Partial field reset on close** — Are ALL logically coupled fields reset atomically in completion and close paths? Leaving a derived field (e.g., `shares_pending`, `rewards_owed`) non-zero after its parent is zeroed breaks protocol invariants permanently.
- **Merged locked/unlocked balances during migration** — When migrating positions, are pending (locked) and withdrawable (matured) balances transferred as separate quantities? Merging them or reapplying lockup to unlocked amounts is a critical bug.
- **Counter drift** — Are all counters and statistics updated atomically with the operation that triggers them (fill count, volume, total supply)? A drifting counter is a protocol invariant violation.

### 15. Shared Position & Pool Logic

- **Missing destination preprocessing** — Before transferring shares or liquidity between positions, are BOTH source and destination preprocessed (settle pending fees, snapshot reward accumulators)? Skipping destination lets a user claim fees they never earned.
- **Self-transfer fee inflation** — Can a no-op or self-transfer pattern inflate fee claims? Verify `source != destination` before any share movement.
- **Unintended fee asymmetry** — If directional fee asymmetry (buy vs. sell) exists, is it intentional and documented? If symmetry is required, apply fees on input side for both directions.

### 16. Clock & Timing

- **Mixed time units** — Does the program mix slots and seconds in time-dependent logic? A vesting window in seconds compared to raw slots can unlock 4× earlier than intended. Use a single canonical time unit throughout.
- **Missing scale factor** — When comparing durations across unit boundaries, is the correct scale factor applied (e.g., multiply slot count by `SLOTS_PER_SECOND`)?
- **Unannotated time fields** — Are time fields annotated with their unit in code (`vesting_end_slot: u64`, `unlock_timestamp_secs: i64`)? Unnamed fields are silently misused as code evolves.

### 17. Token / Mint Integrity

- **Mint close authority set** — Does the program assert that the mint close authority is `None` during initialization? A mint with close authority can be closed and re-initialized at the same address with different decimals, breaking all downstream accounting.
- **Mutable mint properties assumed immutable** — Are immutable mint properties (decimals, supply cap, authorities) stored at account creation and re-validated on every instruction? Never assume they can't change between calls.
- **Recycled address state inheritance** — Can a reinitialized account at a recycled address inherit state from its previous lifetime? Validate all fields as if the account is fresh.

### 18. Protocol-Level Input Validation

- **Unconstrained mint acceptance** — Are token mints validated against a protocol allowlist or framework constraints (`mint::authority`, `mint::decimals`)? An unconstrained mint allows arbitrary tokens to be injected into protocol flows.
- **Same-asset operations** — Where distinct assets are required, is `input_mint != output_mint` enforced? Same-token operations can exploit fee accounting or pool invariants.
- **Unbounded variable-length inputs** — Are maximum sizes enforced on variable-length inputs (messages, payloads, URIs) before encoding? Unbounded inputs cause compute overruns and silent log truncation.
- **Unconstrained protocol address updates** — Are protocol-owned addresses (fee recipients, config accounts) verified as expected before updating? An unconstrained update enables fee redirection to attacker-controlled accounts.

### 19. Type Narrowing & Integer Safety

- **Silent integer narrowing** — Are numeric types consistent across instruction params, on-chain state, and emitted events? Silently narrowing (e.g., `u64 → u32`) causes on-chain state and events to diverge, breaking auditability.
- **Missing upper-bound assertion before cast** — Before any narrowing cast, is an explicit upper-bound asserted (`require!(val <= u32::MAX as u64, ...)`)?
- **Late input validation** — Are all amounts validated at instruction entry (`> 0`, within protocol min/max bounds) before being passed into math helpers? Deep validation catches bugs late and produces confusing error codes.

### 20. Event Logging

- **Oversized log messages** — Are individual log messages concise? Solana truncates transaction logs at ~10 KB per transaction — long strings are silently dropped.
- **Free-form string events** — Are critical state changes (amounts, authorities, timestamps, balances) emitted as structured, fixed-size on-chain events rather than free-form strings?
- **Log-only auditability** — Does the program rely solely on logs for auditability? Logs are ephemeral and truncatable — persist critical state in on-chain accounts.

---

### Framework-Specific Audit Checks

#### Anchor Programs — Check For:

- **`AccountInfo` or `UncheckedAccount` used where `Account<T>` should be** — Missing automatic owner and discriminator checks. Look for missing `/// CHECK:` comments.
- **`init_if_needed` without reinitialization guard** — Attacker can pre-create the account with malicious state. Prefer `init`.
- **Missing `reload()` after CPI** — Anchor caches deserialized account data. After any CPI that modifies an account, `reload()` must be called before using the data.
- **`token::transfer` instead of `token_interface::transfer_checked`** — Legacy transfer hardcodes Token Program ID, fails silently with Token-2022 mints.
- **Manual pubkey comparison instead of `has_one`** — Constraint-based validation is more reliable and harder to bypass than function-body checks.
- **User-supplied bump instead of stored canonical bump** — PDAs should store their canonical bump at init and reuse via `bump = account.bump` constraint.
- **`realloc` without `zero_init = true`** — After a prior decrease in the same transaction, leftover bytes from the shrunk account could be misread as valid data.
- **Missing `overflow-checks = true` in `[profile.release]`** — Anchor 0.30+ requires this. Without it, release-mode arithmetic silently wraps.

#### Native Rust Programs — Check For:

- **Incomplete validation sequence** — Every account must pass: key check → owner check → signer check → writable check → discriminator check → data validation. Missing any step is a vulnerability.
- **Missing manual discriminator** — Without Anchor's automatic 8-byte discriminator, native programs must manage type tags manually. Missing discriminators enable type cosplay.
- **`unwrap()` / `expect()` in instruction handlers** — These panic and abort the transaction. Use `?`, `ok_or()`, or explicit match patterns.
- **Raw byte casting instead of `try_from_slice`** — Manual transmutes or pointer reads are undefined behavior. Always use Borsh deserialization.
- **Data length not verified before deserialization** — An undersized account will panic or misread during deserialization. Always check `account.data_len() >= T::LEN`.
- **Incomplete account close sequence** — Must: (1) zero all data, (2) transfer all lamports, (3) assign to System Program. Skipping any step enables revival or rent theft.
- **Hardcoded lamports for rent** — Always use `rent.minimum_balance(size)`, never a hardcoded value.
- **Stale data used after CPI** — Must manually re-borrow and re-deserialize account data after CPI. No automatic `reload()` exists.

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
- **Sections 11-20** cover protocol-level vulnerabilities from real DeFi audits: oracle manipulation, fee bypass, token dust DoS, state management bugs, timing issues, and mint integrity attacks. These are often missed by checklist-only reviews.
- **Framework-specific checks** at the end cover the most common Anchor footguns and native Rust validation gaps. Apply the relevant section based on the framework detected.
- Use alongside `common/review-checklist.md` — the generic checklist covers cross-language patterns (business logic, access control, arithmetic), while this file covers Solana-specific gotchas.
- For grep-based surface mapping of Solana codebases, see the Solana section in `common/grep-patterns.md`.
- For building secure Solana programs from scratch, see the [Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder) skill in `safe-solana-builder/`.
