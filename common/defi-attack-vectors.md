# DeFi Attack Vectors

## Purpose

Use this prompt to analyze a DeFi protocol against known attack vectors, informed by historical exploits.

## Prompt

```
You are a DeFi security researcher. Analyze [Protocol Name], a [protocol type], for the following known attack vectors:

**Price Manipulation**
- Oracle manipulation (TWAP, Chainlink, custom)
- Spot price manipulation via flash loans
- Sandwich attacks on user transactions

**Flash Loan Attacks**
- Governance vote manipulation
- Collateral ratio manipulation
- LP share price inflation/deflation
- Reward distribution gaming

**Economic Exploits**
- First depositor / inflation attack on vaults (ERC4626)
- Donation attacks to manipulate share price
- Bad debt creation through market manipulation
- Liquidation cascades

**Bridge / Cross-chain**
- Message replay across chains
- Incomplete message validation
- Race conditions between chains

**Governance**
- Flash loan governance attacks
- Timelock bypass
- Proposal manipulation

**Token-specific**
- Fee-on-transfer breaking accounting
- Rebasing tokens breaking share math
- ERC777 reentrancy via hooks
- Weird decimals (non-18) causing precision issues
- USDT approval race condition

**Composability Risks**
- Assumptions about external protocol behavior
- Hardcoded addresses that can change
- Missing interface validation

For each applicable vector:
1. Is this protocol potentially vulnerable?
2. What specific contract/function would be the entry point?
3. What would the exploit flow look like?
4. How can it be mitigated?
```

## Usage Tips

- Reference https://rekt.news and https://solodit.xyz for historical exploits matching these patterns.
- Focus on vectors that match the protocol type — not every vector applies everywhere.
