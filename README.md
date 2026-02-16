# web3-sec-ai-prompts

A curated collection of AI prompts for Web3 security researchers, auditors, and bug bounty hunters.

## What is this?

Battle-tested prompts designed to accelerate your Web3 security workflow — from bug bounty target selection to private audit code reviews, contest strategies, and common vulnerability patterns.

## Structure

| Directory | Description |
|-----------|-------------|
| `bug-bounty/` | Prompts for bug bounty hunting workflows |
| `private-audits/` | Prompts and full audit guide with rules, heuristics, and verifier |
| `contests/` | Prompts for audit contest strategies |
| `zk-audits/` | ZK circuit audit guide — soundness, completeness, privacy, DSL-specific checks |
| `common/` | Shared prompts for patterns, attack vectors, and severity |

## How to Use

1. **Pick the directory** that matches your engagement type (`bug-bounty/`, `private-audits/`, or `contests/`). Use `common/` prompts alongside any of them.
2. **Copy the prompt** into your AI tool of choice (ChatGPT, Claude, Cursor, etc.).
3. **Fill in the placeholders** — protocol name, contract code, chain, etc. The more context you provide, the better the output.
4. **Feed actual code.** Don't just describe the contract — paste the Solidity source directly into the prompt for real analysis.
5. **Chain prompts together.** These work best as a pipeline: recon → code review → severity assessment → report writing.
6. **Iterate.** If the first output isn't useful, refine with more context or break the task into smaller pieces.

### Tips

- **Always verify AI output.** These prompts accelerate your workflow — they don't replace your expertise. The AI will hallucinate findings, miss context, and get severity wrong. You are the final reviewer.
- **Go contract by contract.** Don't feed the entire codebase at once — focus on one contract at a time.
- **Write a PoC.** If you can't prove a finding with a Foundry/Hardhat test, it's probably not valid.
- **Use the best model available.** For deep code analysis, use Claude, GPT-5.2 or +, or similar. Don't use lightweight models for security work.

## Inspirations & Credits

The prompts, rules, and heuristics in this repo are shaped by lessons learned from these resources:
**Links**
- [Solodit](https://solodit.cyfrin.io/)
- [Updraft Cyfrin](https://updraft.cyfrin.io/)

**Videos:**
- [Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI)
- [How I use LLMs](https://www.youtube.com/watch?v=EWvNQjAaOHw)
- [DevDacian - Smart Contract Heuristics & Auditor Branding](https://youtu.be/AiNneURcxDw)

**ZK Security:**
- [PositiveSecurity — ZK Audit Guide](https://github.com/PositiveSecurity/zk-audit-guide)
- [Nethermind — ZK Circuit Security: A Guide for Engineers and Architects](https://www.nethermind.io/blog/zk-circuit-security-a-guide-for-engineers-and-architects)
- [Safe Edges — A Comprehensive Engineering Guide to ZK Circuit Security](https://medium.com/@safeedges/the-silent-guardian-a-comprehensive-engineering-guide-to-zk-circuit-security-9db143dc67f9)
- [Bernhard Mueller — Finding Soundness Bugs in ZK Circuits](https://muellerberndt.medium.com/finding-soundness-bugs-in-zk-circuits-ea23387a0e1e)
- [Floating Pragma — Awesome ZK Proofs](https://floatingpragma.io/awesome-zk-proofs/)
- [Nethermind ZK Security Checklist (PDF)](https://drive.google.com/file/d/1hOkeY2U4K8eyf-Vcy1UsgqXqLRuiGgAi/view)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding or improving prompts.

## License

MIT
