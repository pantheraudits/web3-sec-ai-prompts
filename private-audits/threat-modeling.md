# Threat Modeling

## Purpose

Use this prompt to build a threat model for a protocol before diving into code, identifying the most likely attack paths and high-value targets.

## Prompt

```
You are a Web3 security researcher building a threat model for [Protocol Name], a [protocol type, e.g., lending protocol, DEX, bridge] deployed on [Chain].

Based on the following information, build a comprehensive threat model:

- **Architecture:** [brief description or link to docs]
- **Key contracts:** [list main contracts and their roles]
- **External integrations:** [oracles, AMMs, bridges, tokens]
- **Privileged roles:** [owner, admin, keeper, guardian]
- **User flows:** [deposit, borrow, swap, stake, etc.]

Structure the threat model as:

1. **Assets at Risk** — What has value? (user funds, LP tokens, governance power, oracle data)
2. **Threat Actors** — Who might attack? (external attacker, malicious admin, MEV bot, compromised oracle)
3. **Attack Surfaces** — Where can they attack? (public functions, callbacks, token interactions, governance)
4. **Attack Scenarios** — For each surface, what are the realistic attack paths?
5. **Trust Boundaries** — Where does the protocol trust external input? (user params, oracle prices, token behavior)
6. **Priority Matrix** — Rank attack scenarios by likelihood × impact
```

## Usage Tips

- Do this BEFORE code review to focus your audit time on what matters most.
- Update the threat model as you discover new information during the audit.
