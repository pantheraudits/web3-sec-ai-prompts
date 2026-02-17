# Protocol Detection

The first step in any review pipeline. Identify the blockchain, protocol type, and architecture before reading a single line of code — then apply type-specific attack patterns.

## Why This Matters

Different protocol types have fundamentally different attack surfaces. A lending protocol's biggest risk is oracle manipulation and liquidation logic. A bridge's biggest risk is message replay and validation. Running the same generic review on both wastes time on irrelevant checks and misses type-specific gotchas.

Run this prompt **before** any other review prompt to generate a protocol profile that sharpens everything downstream.

## Prompt

```
You are a Web3 security researcher performing initial reconnaissance on a codebase. Before any vulnerability analysis, identify the protocol's architecture.

## Contract Code

[Paste contract code OR provide repository link]

## Step 1: Identify Blockchain & Language

- What blockchain is this targeting? (EVM, Solana, Cosmos, NEAR, Sui, Aptos, StarkNet, etc.)
- What language is used? (Solidity, Vyper, Rust/Anchor, Move, Cairo, CosmWasm, etc.)
- What compiler/framework version? Flag if outdated.
- What chain-specific features are used? (EVM precompiles, Solana CPIs, Cosmos IBC, etc.)

## Step 2: Identify Protocol Type

Determine the primary protocol category and any secondary categories:

- **AMM / DEX** — Token swaps, liquidity pools, concentrated liquidity
- **Lending / Borrowing** — Collateral, debt positions, liquidations, interest rates
- **Vault / Yield** — Deposit/withdraw, share-based accounting, strategy routing
- **Bridge / Cross-chain** — Message passing, lock-and-mint, validator sets
- **Oracle** — Price feeds, data aggregation, update mechanisms
- **Governance** — Voting, proposals, timelocks, delegation
- **NFT / Marketplace** — Minting, trading, royalties, auctions
- **Staking / Restaking** — Delegation, slashing, reward distribution
- **Derivatives / Perps** — Positions, funding rates, settlement, margin
- **Stablecoin** — Peg mechanism, collateral management, liquidation

## Step 3: Map the Architecture

- What are the core contracts and their roles?
- What is the upgrade pattern? (Proxy, Diamond, immutable, etc.)
- What external dependencies exist? (Oracles, other protocols, token standards)
- What are the privileged roles and what can they do?
- Where do user funds live? (Which contract holds tokens?)

## Step 4: Apply Type-Specific Attack Checklist

Based on the identified protocol type, list the TOP attack vectors that specifically apply:

**AMM / DEX:**
- MEV / sandwich attacks on swaps
- LP share price manipulation
- Tick/price range manipulation
- Concentrated liquidity edge cases (JIT, range orders)
- Fee accounting errors on fee-on-transfer tokens
- Slippage and deadline parameter validation

**Lending / Borrowing:**
- Oracle price manipulation → bad liquidations or bad debt
- Interest rate model overflow at edge rates
- Liquidation logic (bonus calculation, underwater positions, cascading liquidations)
- Collateral factor manipulation
- Flash loan borrow → manipulate → repay in one tx
- Interest accrual rounding (borrow vs supply rate)

**Vault / Yield:**
- First depositor / inflation attack (ERC4626)
- Share price manipulation via donation
- Strategy migration losing funds in transit
- Withdrawal queue manipulation / griefing
- Harvest sandwich (front-run compound/harvest)
- Decimal mismatch between vault and underlying

**Bridge / Cross-chain:**
- Message replay across chains or after reorg
- Incomplete message validation (sender, chain ID, nonce)
- Race conditions between source and destination chain
- Relayer manipulation or censorship
- Token mapping errors (wrong decimals, wrong address)
- Stuck funds when destination chain reverts

**Oracle:**
- Stale price acceptance (missing freshness check)
- Manipulable sources (spot price, low-liquidity pools)
- L2 sequencer downtime → stale prices accepted
- Price deviation bounds too wide or missing
- Decimal/precision mismatch between oracle and consumer

**Governance:**
- Flash loan voting (borrow → vote → return)
- Proposal manipulation (dust proposals, quorum gaming)
- Timelock bypass via emergency functions
- Delegation double-counting
- Snapshot timing attacks

**Staking / Restaking:**
- Reward distribution rounding (dust accumulation)
- Slashing accounting errors
- Unbonding period bypass
- Reward rate manipulation via flash stake
- Validator set manipulation

**Derivatives / Perps:**
- Funding rate manipulation
- Position liquidation sandwich
- Settlement price manipulation
- Margin calculation precision loss
- Open interest limits bypass

## Step 5: Output Protocol Profile

Produce a structured profile:

Protocol: [Name]
Chain: [Chain]
Language: [Language + version]
Type: [Primary] + [Secondary if any]
Core contracts: [List with one-line descriptions]
Fund-holding contracts: [Which contracts hold user funds]
External deps: [Oracles, protocols, tokens]
Privileged roles: [Who can do what]
Upgrade pattern: [Type]
Top 5 attack vectors for this type: [Ranked list]

This profile should be attached to all subsequent review prompts as context.
```

## Usage Tips

- Run this once per protocol, not per contract. The output is a protocol-level profile.
- Attach the generated profile to every subsequent prompt you run (`audit-guide.md`, `hunting-guide.md`, `multi-expert-review.md`, etc.) for sharper results.
- If the protocol spans multiple categories (e.g., a lending protocol with an integrated DEX), apply attack checklists for ALL identified types.
- Update the profile if you discover new architecture details during the audit.
