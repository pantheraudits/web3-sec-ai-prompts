# Safe Solana Builder

A Claude/Cursor skill for generating production-grade, secure Solana programs. Based on [Frank Castle's Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder).

Rather than a simple prompt, this is a layered reference architecture that forces the AI to evaluate risk levels, load framework-specific security rules, and apply 20 security rule categories before generating any code.

## What It Produces

Every invocation delivers four artifacts:

1. **Full project scaffold** — Ready-to-build directory structure with Cargo/Anchor config
2. **Program code** (`lib.rs` + supporting files) — Compilable, with inline security annotations
3. **Test skeleton** — Happy-path tests implemented, security edge-case tests scaffolded with TODOs
4. **Security checklist** — Documents every applied rule, assumptions, and known limitations

## Frameworks Supported

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Anchor** | Most programs | Auto-validates accounts, built-in constraints |
| **Native Rust** | Full control, custom runtimes | Manual validation — every check is your responsibility |
| **Pinocchio** | High-throughput (DEXs, orderbooks) | 88–95% CU reduction vs Anchor. Unaudited — flagged in checklist |

## Security Coverage

The skill enforces 20 categories of security rules from `references/shared-base.md`:

| # | Category | What it catches |
|---|----------|----------------|
| 1 | Account & Identity Validation | Missing signer/owner/discriminator checks |
| 2 | PDA Security | Non-canonical bumps, seed collisions, sharing |
| 3 | Arithmetic & Logic Safety | Overflow, multiply-before-divide, slippage |
| 4 | Duplicate Mutable Accounts | Same account passed for two roles |
| 5 | CPI Safety | Arbitrary CPI, stale state, signer escalation |
| 6 | Account Storage & Lifecycle | Insecure close, revival attacks, rent |
| 7 | Token-2022 Compatibility | Legacy transfer on Token-2022 mints |
| 8 | Transaction Model Safety | Compute budget, ALTs, durable nonces |
| 9 | Safe Rust Patterns | vec! footgun, unsafe blocks, remaining_accounts |
| 10 | Curiosity Principle | Adversarial mindset questions |
| 11 | Oracle Validation | Confidence intervals, staleness, retroactive pricing |
| 12 | Fee Completeness | Fee bypass on edge-case code paths |
| 13 | Token Dust & Account DoS | Dust deposit blocking close, expired accounts |
| 14 | State Management | Coupled field resets, counter drift |
| 15 | Shared Position & Pool Logic | LP preprocessing gaps, self-transfer inflation |
| 16 | Clock & Timing | Slot vs seconds mixing |
| 17 | Token/Mint Integrity | Mint close authority, address recycling |
| 18 | Protocol-Level Input Validation | Mint allowlists, variable-length bounds |
| 19 | Type Narrowing & Integer Safety | Silent narrowing casts, type consistency |
| 20 | Event Logging | Log truncation, structured events |

Plus framework-specific rules from `references/anchor.md` or `references/native-rust.md`.

## Installation

### Cursor

Copy the skill directory to your Cursor skills folder:

```bash
cp -r safe-solana-builder/ ~/.cursor/skills/safe-solana-builder/
```

Or symlink it from this repo:

```bash
ln -s $(pwd)/safe-solana-builder ~/.cursor/skills/safe-solana-builder
```

### Claude Code

Reference SKILL.md in your project's `.claude/settings.json` or use it directly via slash commands.

### Claude.ai (Projects)

1. Create a new Claude Project
2. Upload `SKILL.md` and all files from `references/` to the project's knowledge base
3. Start a conversation with: "Write a Solana program that..."

## Usage

Trigger phrases:
- "Write a Solana program that..."
- "Build a Solana smart contract for..."
- "Scaffold an Anchor program that..."
- "Create a native Rust Solana program for..."

The skill will:
1. Ask which framework (Anchor / Native Rust / Pinocchio)
2. Load the relevant security reference files
3. Assess risk level (Low / Medium / Critical)
4. Gather requirements
5. Write the program with full security annotations
6. Deliver scaffold + code + tests + checklist

## Relationship to Audit Pipeline

This builder skill is **complementary** to the Panther Audit skill (`claude-skill/`):

- **Safe Solana Builder** = writes new secure code (proactive)
- **Panther Audit** = reviews existing code for vulnerabilities (reactive)
- Both share security knowledge from `common/solana-patterns.md`

## Credits

- [Frank Castle](https://github.com/Frankcastleauditor/safe-solana-builder) — Original Safe Solana Builder skill and security rules
- [Trail of Bits](https://blog.trailofbits.com/) — Solana security research
