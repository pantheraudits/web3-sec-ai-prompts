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
- These patterns are Solidity-focused. For Rust/Anchor, Vyper, Cairo, or Move, adapt the patterns to the equivalent constructs.
