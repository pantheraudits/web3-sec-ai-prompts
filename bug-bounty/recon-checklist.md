# Recon Checklist

## Purpose

Use this prompt to systematically gather intelligence on a bug bounty target before diving into code.

## Prompt

```
You are helping me perform reconnaissance on a Web3 bug bounty target. The protocol is [Protocol Name] on [Chain].

Help me build a recon checklist by answering:

1. **Documentation:** Where are the official docs, whitepaper, and architecture diagrams?
2. **Previous audits:** List all known audit reports and who performed them.
3. **Known issues:** Are there any disclosed vulnerabilities or post-mortems?
4. **Deployment info:** What are the mainnet contract addresses? Are they verified on-chain?
5. **Dependencies:** What external protocols/oracles/tokens does it integrate with?
6. **Governance:** Is there a multisig, timelock, or DAO controlling upgrades?
7. **Token mechanics:** Does it have a native token? Any rebasing, fee-on-transfer, or non-standard behavior?
8. **TVL and activity:** What is the current TVL? How actively is it being used?
9. **Code freshness:** When was the last commit? Are there recent changes not covered by audits?
10. **Community signals:** Any active discussions about bugs or concerns in Discord/forums?
```

## Usage Tips

- Use this before writing a single line of exploit code.
- Cross-reference audit reports with current deployed code to find post-audit changes.
