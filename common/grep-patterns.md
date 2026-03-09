# Grep Patterns

Quick-reference search patterns for systematically scanning Solidity codebases. Run these before deep review to map the attack surface and know where to focus.

## How to Use

Run these patterns against the codebase using `grep`, `rg` (ripgrep), or your IDE's search. The results give you a map of where dangerous operations live — then prioritize your manual review around those locations.

For AI-assisted reviews, paste relevant results into your prompts as additional context.

## Patterns

### External Calls & Reentrancy Vectors

```bash
# Low-level calls (reentrancy, unchecked return values)
rg "\.call\{" --glob "*.sol"
rg "\.call\(" --glob "*.sol"
rg "\.delegatecall\(" --glob "*.sol"
rg "\.staticcall\(" --glob "*.sol"

# Transfer/send (2300 gas limit, but still external calls)
rg "\.transfer\(|\.send\(" --glob "*.sol"

# Token transfers (check for return value handling, fee-on-transfer)
rg "\.safeTransfer\(|\.safeTransferFrom\(" --glob "*.sol"
rg "\.transfer\(|\.transferFrom\(" --glob "*.sol" 

# Cross-contract calls via interfaces
rg "I[A-Z]\w+\(" --glob "*.sol"
```

### Access Control

```bash
# Modifiers and role checks
rg "onlyOwner|onlyAdmin|onlyRole|onlyGuardian|onlyKeeper" --glob "*.sol"
rg "require\(msg\.sender" --glob "*.sol"
rg "if\s*\(msg\.sender" --glob "*.sol"

# OpenZeppelin access control
rg "hasRole\(|grantRole\(|revokeRole\(|renounceRole\(" --glob "*.sol"
rg "Ownable|AccessControl|AccessManaged" --glob "*.sol"

# Missing access control — external/public state-changing functions without modifiers
# (manual review required — search for external functions, then check which lack access control)
rg "function\s+\w+\s*\([^)]*\)\s+external" --glob "*.sol"
rg "function\s+\w+\s*\([^)]*\)\s+public" --glob "*.sol"
```

### Dangerous Operations

```bash
# Self-destruct (fund drain, proxy bricking)
rg "selfdestruct|SELFDESTRUCT" --glob "*.sol"

# Inline assembly (memory corruption, storage manipulation)
rg "assembly\s*\{" --glob "*.sol"

# Unchecked arithmetic (intentional overflow risk)
rg "unchecked\s*\{" --glob "*.sol"

# Delegatecall (storage collision, upgrade attacks)
rg "delegatecall" --glob "*.sol"

# tx.origin (phishing attacks)
rg "tx\.origin" --glob "*.sol"
```

### Arithmetic & Precision

```bash
# Decimal assumptions
rg "1e18|1e6|1e8|10\*\*18|10\*\*6|decimals" --glob "*.sol"

# Division (precision loss, division before multiplication)
rg "\s/\s" --glob "*.sol"

# Type casting (truncation risks)
rg "uint8\(|uint16\(|uint32\(|uint64\(|uint128\(|int8\(|int16\(|int32\(" --glob "*.sol"

# Unchecked blocks near user input
rg "unchecked" --glob "*.sol"
```

### Oracle & Price Dependencies

```bash
# Chainlink
rg "latestRoundData|latestAnswer|AggregatorV3" --glob "*.sol"

# TWAP / Uniswap oracles
rg "observe\(|consult\(|TWAP|twap" --glob "*.sol"

# Generic price references
rg "getPrice|price\(\)|pricePerShare|exchangeRate|getRate" --glob "*.sol"

# Stale price indicators
rg "updatedAt|answeredInRound|roundId|heartbeat" --glob "*.sol"
```

### Token Standards & Hooks

```bash
# ERC standards implemented
rg "ERC20|ERC721|ERC1155|ERC4626|ERC777|ERC2981" --glob "*.sol"

# EIP references
rg "EIP-|EIP_|eip-" --glob "*.sol"

# Callback hooks (reentrancy entry points)
rg "onERC721Received|onERC1155Received|tokensReceived|onFlashLoan" --glob "*.sol"
rg "fallback\(\)|receive\(\)" --glob "*.sol"

# Approval patterns
rg "approve\(|permit\(|allowance\(" --glob "*.sol"
```

### Upgradability & Proxy

```bash
# Proxy patterns
rg "Proxy|proxy|IMPLEMENTATION|implementation|upgradeTo" --glob "*.sol"
rg "initializer|initialized|__init|reinitializer" --glob "*.sol"

# Storage slots (EIP-1967)
rg "0x360894|0xb53127|0xa3f0ad74" --glob "*.sol"

# Storage gaps
rg "__gap|uint256\[" --glob "*.sol"
```

### State & Control Flow

```bash
# Pause mechanisms
rg "pause|unpause|whenNotPaused|whenPaused|Pausable" --glob "*.sol"

# Reentrancy guards
rg "nonReentrant|ReentrancyGuard|_status|_locked" --glob "*.sol"

# Emergency functions
rg "emergency|emergencyWithdraw|rescue|recover|sweep" --glob "*.sol"

# Timelocks and delays
rg "timelock|TimeLock|delay|cooldown|DELAY|MIN_DELAY" --glob "*.sol"
```

### Constants & Magic Numbers

```bash
# Hardcoded values that need verification
rg "constant\s+\w+\s*=" --glob "*.sol"
rg "immutable\s+\w+" --glob "*.sol"

# Basis points / percentages
rg "10000|10_000|BPS|bps|BASIS|PERCENT|FEE" --glob "*.sol"

# Time constants
rg "days|hours|minutes|weeks|365|86400|3600" --glob "*.sol"
```

### Solana / Rust-Anchor

```bash
# CPI calls (cross-program invocation — Solana's external call equivalent)
rg "invoke\(|invoke_signed\(" --glob "*.rs"
rg "CpiContext" --glob "*.rs"
rg "anchor_lang::solana_program::program::invoke" --glob "*.rs"

# Account validation (missing checks = Solana's #1 vulnerability class)
rg "is_signer|\.key\(\)" --glob "*.rs"
rg "\.owner\s*==" --glob "*.rs"
rg "UncheckedAccount|AccountInfo" --glob "*.rs"
rg "has_one|constraint\s*=" --glob "*.rs"

# PDA derivation and bump handling
rg "find_program_address|create_program_address" --glob "*.rs"
rg "seeds\s*=|bump\s*=" --glob "*.rs"
rg "Pubkey::find_program_address" --glob "*.rs"

# Arithmetic safety (Rust release mode DISABLES overflow checks)
rg "checked_add|checked_sub|checked_mul|checked_div" --glob "*.rs"
rg "\.unwrap\(\)|\.expect\(" --glob "*.rs"
rg "\bas\s+(u8|u16|u32|u64|u128|i8|i16|i32|i64|i128|usize)\b" --glob "*.rs"
rg "overflow-checks" --glob "Cargo.toml"

# Anchor-specific patterns
rg "init_if_needed" --glob "*.rs"
rg "remaining_accounts" --glob "*.rs"
rg "#\[account\(" --glob "*.rs"
rg "close\s*=" --glob "*.rs"

# Token operations (Token-2022 compatibility)
rg "transfer_checked|TransferChecked" --glob "*.rs"
rg "Token2022|token_2022|spl_token_2022" --glob "*.rs"
rg "spl_token|anchor_spl::token" --glob "*.rs"

# Account lifecycle (closing, rent, sysvars)
rg "close_account|CloseAccount" --glob "*.rs"
rg "rent_exempt|Rent::get" --glob "*.rs"
rg "Sysvar|Clock::get|sysvar::instructions" --glob "*.rs"

# Oracle validation (§11 — confidence, staleness, price feeds)
rg "price|oracle|pyth|switchboard|chainlink" -i --glob "*.rs"
rg "confidence|conf\s*[/\.]|stale|max_age|staleness" --glob "*.rs"
rg "get_price|price_feed|PriceFeed|OraclePrice" --glob "*.rs"

# Fee logic (§12 — fee paths, fee deduction, basis points)
rg "fee|FEE|basis_point|bps|BPS|10000|10_000" --glob "*.rs"
rg "protocol_fee|swap_fee|withdrawal_fee|redemption_fee" --glob "*.rs"

# Token dust & account DoS (§13 — dust, close, expiry)
rg "dust|sweep|burn|residual" --glob "*.rs"
rg "expire|expiry|expired|deadline" --glob "*.rs"

# State management — coupled fields & counters (§14)
rg "pending|owed|accrued|accumulated|shares_pending" --glob "*.rs"
rg "total_supply|total_staked|fill_count|volume" --glob "*.rs"

# Shared position & pool logic (§15 — LP, shares, preprocess)
rg "shares|liquidity|lp_|pool_|position" --glob "*.rs"
rg "preprocess|settle|snapshot|accrue" --glob "*.rs"

# Clock & timing (§16 — slot vs seconds, time units)
rg "Clock::get|slot|timestamp|unix_timestamp" --glob "*.rs"
rg "SLOTS_PER|seconds|duration|elapsed" --glob "*.rs"

# Mint integrity (§17 — close authority, decimals, reinit)
rg "close_authority|mint_authority|freeze_authority" --glob "*.rs"
rg "decimals|supply_cap" --glob "*.rs"

# Protocol-level input validation (§18 — allowlist, mint constraints)
rg "allowlist|whitelist|approved_mint" --glob "*.rs"
rg "input_mint.*output_mint|mint::authority|mint::decimals" --glob "*.rs"
rg "max_len|max_size|MAX_" --glob "*.rs"

# Type narrowing & integer safety (§19 — as casts, try_into)
rg "as\s+(u8|u16|u32|i32)" --glob "*.rs"
rg "try_into|try_from|TryInto|TryFrom" --glob "*.rs"

# Event logging (§20 — msg!, emit!, log)
rg "msg!|emit!|sol_log|log_" --glob "*.rs"
```

## Prompt: AI-Assisted Surface Mapping

After running grep patterns, feed the results to the AI for prioritization:

```
I ran the following search patterns against the codebase and got these results:

[Paste grep results here]

Based on these results:
1. Which external calls lack reentrancy protection?
2. Which external/public functions lack access control but modify state?
3. Are there arithmetic operations near user-controlled input that could overflow/underflow?
4. Which oracle calls are missing staleness checks?
5. Are there token transfers without SafeERC20?
6. Rank the top 10 most dangerous locations I should review first, based on what you see.
```

## Usage Tips

- Run these at the start of every engagement — takes 5 minutes, saves hours of aimless code reading.
- Pipe results to a file (`rg "\.call\{" --glob "*.sol" > surface-map.txt`) for reference throughout the audit.
- Combine with `protocol-detection.md` — after identifying the protocol type, prioritize the grep patterns most relevant to that type.
- Solidity and Solana/Rust-Anchor patterns are included above. For Vyper, Cairo, or Move, adapt the patterns to the equivalent constructs.
