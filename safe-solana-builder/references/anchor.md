# Anchor Framework — Specific Patterns, Constraints & Pitfalls
## Frank Castle — Safe Solana Builder

Read this AFTER shared-base.md when the user selects Anchor.
Every rule here is in addition to — not instead of — shared-base.md.

---

## 1. ACCOUNT TYPES — USE THE RIGHT WRAPPER

Anchor provides typed account wrappers. Using the wrong one skips critical checks.

### 1.1 Account Type Selection Rules
- **`Account<'info, T>`** — Use whenever you expect typed, owned data. Anchor automatically verifies:
  - The account's owner matches the program that defined `T`
  - The discriminator (first 8 bytes) matches `T`
  - Never use `AccountInfo` or `UncheckedAccount` where `Account<T>` is possible.

- **`Signer<'info>`** — Use for accounts that must sign. Anchor verifies `is_signer` automatically.

- **`SystemAccount<'info>`** — Use for accounts that must be owned by the System Program.

- **`Program<'info, T>`** — Use for program accounts (e.g., `Program<'info, Token>`). Verifies the account is executable and matches the program ID.

- **`UncheckedAccount<'info>`** — Use only when you have a deliberate, documented reason. **Always add a `/// CHECK: <reason>` safety comment** explaining exactly why it's safe. Anchor requires this comment — treat it as a serious obligation, not boilerplate.

- **`Interface<'info, T>`** / **`InterfaceAccount<'info, T>`** — Use for Token-2022 compatible programs. Supports both legacy Token and Token-2022.

### 1.2 Never Use AccountInfo for Typed Data
- `AccountInfo` gives you raw bytes — no discriminator check, no owner check, no type safety.
- If you find yourself using `AccountInfo` and manually deserializing, switch to `Account<T>` unless you have a specific reason not to.

---

## 2. CONSTRAINTS — DECLARE, DON'T IMPEACH

Anchor's `#[account(...)]` constraints are your first line of defense. Push as much validation as possible into constraints, not function bodies.

### 2.1 Core Constraint Patterns

```rust
#[account(
    mut,                                          // must be writable
    has_one = authority,                          // vault.authority == ctx.accounts.authority.key()
    has_one = token_account,                      // vault.token_account == ctx.accounts.token_account.key()
    constraint = vault.amount >= amount @ ErrorCode::InsufficientFunds,
    constraint = vault.key() != destination.key() @ ErrorCode::SameAccount,
)]
pub vault: Account<'info, Vault>,
```

### 2.2 `has_one` — Enforce Cross-Account Relationships
- Use `has_one = field` to verify that a field in the deserialized account matches another account in the context.
- This replaces manual pubkey comparison inside your instruction logic.
- **Every account that "belongs to" another must have this enforced.**

### 2.3 `seeds` and `bump` — PDA Derivation in Constraints
- Use Anchor's built-in PDA verification:
  ```rust
  #[account(
      seeds = [b"vault", user.key().as_ref()],
      bump = vault.bump,
  )]
  pub vault: Account<'info, Vault>,
  ```
- **Always store the canonical bump in your account struct** and pass it back in the constraint.
- Do not use `bump` without `seeds` — it does nothing alone.
- Do not let users provide the bump — store it at init time and reuse it.

### 2.4 `init` vs `init_if_needed` — Critical Distinction
- **`init`**: Creates the account. Fails if the account already exists. Use this for one-time initialization. It sets discriminator + owner, preventing reinitialization.
- **`init_if_needed`**: Creates if not exists, skips if already exists. **This is a footgun.**
  - If you use `init_if_needed`, you MUST manually verify that the existing account's state is valid for your instruction — an attacker can pre-create the account with malicious state.
  - Add explicit checks: `require!(!account.initialized || account.authority == ctx.accounts.user.key(), ...)`.
  - Prefer `init` unless you have a documented reason to use `init_if_needed`.

### 2.5 `close = recipient` — Secure Account Closing
- Use Anchor's `close` constraint to properly close accounts:
  ```rust
  #[account(mut, close = user)]
  pub escrow: Account<'info, Escrow>,
  ```
- This zeroes the data, transfers lamports, and reassigns ownership to the System Program in one safe operation.
- **Always close to a trusted recipient** — never to a user-provided arbitrary account for admin-only closures.
- Never manually drain lamports without using `close` or following the manual sequence in shared-base.md §6.3.

### 2.6 `realloc` — Safe Account Resizing
- When resizing an account:
  ```rust
  #[account(
      mut,
      realloc = new_size,
      realloc::payer = user,
      realloc::zero_init = true,
  )]
  ```
- **Set `zero_init: true`** when increasing account size after a prior decrease in the same transaction. Prevents reading stale "dirty" memory that was previously used.
- Without `zero_init`, leftover bytes from a previously-shrunk account could be misread as valid data.

---

## 3. STATE MANAGEMENT

### 3.1 `reload()` After CPI — Non-Negotiable
- Anchor caches deserialized account data in memory. After any CPI that modifies an account, the in-memory struct is stale.
- **Always call `ctx.accounts.account_name.reload()?`** after a CPI that modifies that account before using its data.
- This applies even if the CPI is to your own program. Treat every CPI as a black box that may modify state.

```rust
// Transfer tokens via CPI
token::transfer(cpi_ctx, amount)?;

// Reload before using updated balance
ctx.accounts.token_account.reload()?;
let new_balance = ctx.accounts.token_account.amount;
```

### 3.2 Secure Initialization
- Your `initialize` instruction must be callable **exactly once**.
- Anchor's `init` constraint enforces this automatically (fails if account already exists).
- If you implement a custom initialization pattern, add an `initialized: bool` flag and `require!(!state.initialized, ...)` as the first check.

### 3.3 `#[access_control(...)]` — Pre-Condition Checks
- Use `#[access_control(check_fn(&ctx))]` for pre-condition checks that apply to an entire instruction.
- Keeps business logic clean by separating access checks from core logic.
- Ideal for: admin-only gates, protocol pause checks, time-lock validation.

---

## 4. TOKEN OPERATIONS — ANCHOR SPL

### 4.1 Token-2022 Compatible Transfers
- **Never use `anchor_spl::token::transfer`** for generic programs that may encounter Token-2022 mints.
- **Always use `anchor_spl::token_interface::transfer_checked`**:
  ```rust
  use anchor_spl::token_interface::{self, TransferChecked};

  token_interface::transfer_checked(
      CpiContext::new(
          ctx.accounts.token_program.to_account_info(),
          TransferChecked {
              from: ctx.accounts.from_ata.to_account_info(),
              mint: ctx.accounts.mint.to_account_info(),  // required
              to: ctx.accounts.to_ata.to_account_info(),
              authority: ctx.accounts.authority.to_account_info(),
          },
      ),
      amount,
      ctx.accounts.mint.decimals,  // required
  )?;
  ```
- Use `InterfaceAccount<'info, Mint>` and `InterfaceAccount<'info, TokenAccount>` for Token-2022 compatibility.
- Use `Interface<'info, TokenInterface>` instead of `Program<'info, Token>` for the token program account.

### 4.2 Mint and Decimal Validation
- Always validate the mint's `decimals` field matches your expected value before using it in calculations.
- Verify `mint.is_initialized` before operating on any mint account.

---

## 5. ANCHOR CPI PATTERNS

### 5.1 Program ID Validation in CPI
- For static, well-known programs, use `Program<'info, T>` — Anchor validates the ID automatically.
- For dynamic programs (e.g., user-provided callback programs):
  ```rust
  require_keys_eq!(
      ctx.accounts.external_program.key(),
      expected_program_id,
      ErrorCode::InvalidProgram
  );
  ```

### 5.2 CpiContext Construction
- Always construct `CpiContext` with only the accounts needed for that specific CPI call.
- Never pass your entire context to a CPI wrapper — it may expose accounts with unintended signer privileges.

### 5.3 Signer Seeds for PDA CPIs
- When a PDA must sign in a CPI:
  ```rust
  let seeds = &[b"vault", user.key().as_ref(), &[vault.bump]];
  let signer_seeds = &[&seeds[..]];
  CpiContext::new_with_signer(program, accounts, signer_seeds)
  ```
- Never hardcode bump values — always use the stored canonical bump.

---

## 6. ERROR HANDLING

### 6.1 Custom Error Codes
- **Always define program-specific error codes.** Never return generic errors or panic.
- Use Anchor's `#[error_code]` derive:
  ```rust
  #[error_code]
  pub enum ErrorCode {
      #[msg("Insufficient funds in vault")]
      InsufficientFunds,
      #[msg("Authority mismatch — provided authority does not own this account")]
      AuthorityMismatch,
      #[msg("Source and destination accounts must differ")]
      SameAccount,
  }
  ```
- Descriptive error messages are crucial: they're the first thing an auditor and a debugging developer will see.

### 6.2 `require!` Over Manual if/return
- Use `require!(condition, ErrorCode::Variant)` for all validation checks.
- It's cleaner, shorter, and produces better error messages than manual `if !condition { return Err(...) }`.
- Use `require_keys_eq!`, `require_eq!`, `require_gt!`, etc. for typed comparisons.

---

## 7. ANCHOR-SPECIFIC FOOTGUNS SUMMARY

| Pattern | Safe Version | Unsafe Version |
|---|---|---|
| Account wrapping | `Account<'info, T>` | `AccountInfo` without `/// CHECK:` |
| Initialization | `init` constraint | `init_if_needed` without reinitialization guard |
| Cross-account link | `has_one = field` | Manual pubkey compare inside function body |
| Closing accounts | `close = recipient` constraint | Manually zeroing + draining without ownership transfer |
| After-CPI data use | `.reload()?` | Using cached struct values |
| Token transfers | `transfer_checked` via `token_interface` | `token::transfer` (legacy only) |
| PDA derivation | `seeds + bump` constraint with stored canonical bump | User-supplied bump |
| Memory resize | `realloc` with `zero_init = true` | Raw realloc without zeroing |
| Error reporting | `#[error_code]` with descriptive messages | `ProgramError::Custom(0)` or panics |

---

## 8. COMMON ANCHOR BUILD & TOOLING ERRORS

### GLIBC Version Too Old (`GLIBC_2.38` / `GLIBC_2.39` not found)
Anchor 0.31+ requires GLIBC >= 2.38; Anchor 0.32+ requires >= 2.39. Ubuntu 24.04+ ships 2.39.
**Fix:** Upgrade OS, or build Anchor CLI from source: `cargo install --git https://github.com/solana-foundation/anchor --tag v0.31.1 anchor-cli`

### `proc_macro_span_shrink` / Rust 1.80 Incompatibility
Anchor 0.30.x uses a `time` crate incompatible with Rust >= 1.80.
**Fix:** Use AVM (auto-pins rustc 1.79 for Anchor <0.31), or upgrade to Anchor 0.31+.

### `unexpected_cfg` Warnings
Newer Rust versions are stricter about `cfg` conditions. Add to `Cargo.toml`:
```toml
[lints.rust]
unexpected_cfgs = { level = "allow" }
```
Or upgrade to Anchor 0.31+.

### IDL Build Fails (`anchor build` or `anchor idl build`)
Ensure `idl-build` feature is enabled (required since 0.30.0):
```toml
[features]
idl-build = ["anchor-lang/idl-build", "anchor-spl/idl-build"]
```
Debug with: `ANCHOR_LOG=1 anchor build`. Skip IDL with: `anchor build --no-idl`.

### `module inner is private`
Version mismatch between `anchor-lang` crate and Anchor CLI. Match versions in `Cargo.toml` and `Anchor.toml`.

### `overflow-checks` Not Specified (Anchor 0.30+)
```toml
[profile.release]
overflow-checks = true
```

### Anchor Version Migration Quick Reference

**0.29 → 0.30:** Change `.accounts({...})` to `.accountsPartial({...})`. Add `idl-build` feature.

**0.30 → 0.31:** Remove direct `solana-program`/`solana-sdk` deps; use `anchor_lang::prelude::*` instead.

**0.31 → 0.32:** `solana-program` fully removed. Use `solana_pubkey::Pubkey` or `anchor_lang::prelude::*`. Duplicate mutable accounts now error — use `dup` constraint.

### `Connection refused` / IPv6 in Tests
Node.js 17+ resolves `localhost` to `::1` but `solana-test-validator` binds to `127.0.0.1`.
**Fix:** Set `cluster = "http://127.0.0.1:8899"` in `Anchor.toml`, or use `NODE_OPTIONS="--dns-result-order=ipv4first"`.

### `declare_program!` IDL Not Found
Place IDL JSON in `idls/<program_name>.json` at workspace root (snake_case filename matching program name).

### CLI / Crate Version Mismatch Warnings
Warnings like `anchor-lang version(0.32.1) and CLI(0.30.1) don't match` are cosmetic — builds succeed. Match versions in `Anchor.toml [toolchain]` and install with `avm install <version>` to eliminate them.
