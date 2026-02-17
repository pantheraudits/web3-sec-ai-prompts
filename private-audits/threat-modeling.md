# Threat Modeling

## Purpose

Use this prompt to build a threat model for a protocol before diving into code, identifying the most likely attack paths, high-value targets, and economic attack vectors. A good threat model tells you where to spend 80% of your review time.

## Prompt

```
You are a Web3 security researcher building a threat model for [Protocol Name], a [protocol type, e.g., lending protocol, DEX, bridge] deployed on [Chain].

Based on the following information, build a comprehensive threat model:

- **Architecture:** [brief description or link to docs]
- **Key contracts:** [list main contracts and their roles]
- **External integrations:** [oracles, AMMs, bridges, tokens]
- **Privileged roles:** [owner, admin, keeper, guardian]
- **User flows:** [deposit, borrow, swap, stake, etc.]
- **TVL / fund distribution:** [approximate TVL and where funds are held]

---

## 1. Assets at Risk

What has value and where does it live?

- User deposited funds (which contracts hold them?)
- LP tokens / share tokens (can they be manipulated?)
- Governance power / voting tokens
- Oracle data and price feeds
- Protocol-owned liquidity
- Fee revenue / treasury
- NFTs, positions, or other non-fungible value

## 2. Threat Actors

Map every type of attacker and their capabilities:

| Actor | Capabilities | Motivation | Example Attack |
|-------|-------------|------------|----------------|
| **External attacker** | Public function access, flash loans, MEV | Direct profit via fund theft | Flash loan → manipulate oracle → drain lending pool |
| **Malicious user** | Normal user privileges, multiple accounts | Gaming incentives, rewards, airdrops | Sybil attack on reward distribution |
| **MEV bot** | Transaction ordering, sandwich, front-run | Extract value from pending transactions | Sandwich LP deposits for price manipulation |
| **Flash loan attacker** | Unlimited single-tx capital, no upfront cost | Single-transaction profit | Inflate collateral → borrow max → default |
| **Governance attacker** | Voting power (purchased or flash-loaned) | Protocol parameter manipulation | Flash loan governance tokens → pass malicious proposal |
| **Compromised admin** | Admin/owner key access | Rug or parameter manipulation | Upgrade to malicious implementation, drain treasury |
| **Compromised oracle** | Feed false price data | Enable downstream exploits | Report inflated price → attacker borrows against it |
| **Keeper / relayer** | Transaction submission, ordering | Griefing, front-running, censorship | Delay liquidation to create bad debt |

For each actor, assess: How much capital do they need? What's their expected profit? Is this economically rational?

## 3. Attack Surfaces

Where can each actor interact with the protocol?

- **Public/external functions** — Every function callable without privileges
- **Callback hooks** — ERC721/1155 receivers, flash loan callbacks, Uniswap callbacks
- **Token interactions** — Fee-on-transfer, rebasing, weird decimals, ERC777 hooks
- **Oracle dependencies** — Every point where external price data enters the system
- **Cross-contract calls** — Every point where the protocol calls external contracts
- **Governance actions** — Proposal creation, voting, execution, parameter changes
- **Upgrade mechanisms** — Proxy upgrades, implementation changes, migration functions
- **Emergency functions** — Pause, emergency withdraw, rescue, sweep

## 4. Attack Scenarios

For each threat actor × attack surface combination that applies, describe the concrete attack:

**Scenario format:**
- Actor: [who]
- Surface: [where they interact]
- Attack flow: [Step 1 → Step 2 → Step 3 → Profit/Damage]
- Capital required: [how much]
- Preconditions: [what must be true]
- Expected profit/damage: [quantify]

## 5. Trust Boundaries

Where does the protocol trust external input? Every trust boundary is a potential exploit point.

- **User-supplied parameters** — amounts, addresses, deadlines, slippage tolerances
- **Oracle prices** — assumed accurate, fresh, and manipulation-resistant
- **Token behavior** — assumed standard ERC20 (no fees, no rebasing, no hooks)
- **External protocol state** — assumed solvent, unpaused, and behaving as documented
- **Admin actions** — assumed honest and timely
- **Cross-chain messages** — assumed authentic and delivered exactly once

For each boundary: What happens if the trust is violated? What's the worst-case outcome?

## 6. Economic Attack Modeling

For the top 5 most likely attack scenarios:

| Scenario | Attack Cost | Expected Profit | Profitable? | Flash Loanable? |
|----------|------------|-----------------|-------------|-----------------|
| [Scenario 1] | [gas + capital + slippage] | [stolen funds] | [Yes/No] | [Yes/No] |
| ... | ... | ... | ... | ... |

If an attack is flash-loanable, the capital requirement drops to near-zero — only gas cost remains. This dramatically increases likelihood.

## 7. Priority Matrix

Rank all scenarios by Likelihood × Impact:

| Priority | Scenario | Likelihood | Impact | Focus Level |
|----------|----------|------------|--------|-------------|
| P0 (Critical) | [scenario] | High | High | Review first, PoC immediately |
| P1 (High) | [scenario] | High/Med | High/Med | Deep review |
| P2 (Medium) | [scenario] | Med | Med | Standard review |
| P3 (Low) | [scenario] | Low | Low | Quick check |

The P0 and P1 scenarios should consume 80% of your audit time.
```

## Usage Tips

- Do this BEFORE code review to focus your audit time on what matters most.
- Pair with `common/protocol-detection.md` — run detection first to identify the protocol type, then feed the result into this threat model for type-specific scenarios.
- Update the threat model as you discover new information during the audit.
- The economic attack modeling section is critical — it separates theoretical vulnerabilities from real exploits.
- For bug bounties, the priority matrix directly maps to where you should spend your time for maximum payout.
