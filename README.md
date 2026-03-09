# web3-sec-ai-prompts

A curated collection of AI prompts for Web3 security researchers, auditors, and bug bounty hunters.

## What is this?

Battle-tested prompts designed to accelerate your Web3 security workflow — from bug bounty target selection to private audit code reviews, contest strategies, and common vulnerability patterns.

## Structure

| Directory | Description |
|-----------|-------------|
| `bug-bounty/` | Bug bounty hunting prompts — scope, severity (critical/high only), trust model |
| `private-audits/` | Private audit prompts — scope, severity (all levels), trust model |
| `contests/` | Audit contest strategy, time management, and report templates |
| `zk-audits/` | ZK circuit audit guide — soundness, completeness, privacy, DSL-specific checks |
| `common/` | **Shared review checklist**, **multi-expert review**, **custom primer guide**, **protocol detection**, **grep patterns**, attack vectors, severity assessment, Solidity patterns |
| `claude-skill/` | **Panther Audit** — automated skill that auto-scales with codebase size: standard 4-phase review for small codebases, chunk mode with persistent state and deduplication for large ones (5k-30k+ NSLOC) |
| `safe-solana-builder/` | **Safe Solana Builder** — skill for generating production-grade, secure Solana programs with full scaffolds, test skeletons, and security checklists. Supports Anchor, Native Rust, and Pinocchio. Based on [Frank Castle's Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder) |

## Quick Start (2 minutes)

Just want to get going? Here's the fastest path:

1. Open the contract you want to review.
2. Copy the prompt from **[`bug-bounty/hunting-guide.md`](bug-bounty/hunting-guide.md)** (for bounties) or **[`private-audits/audit-guide.md`](private-audits/audit-guide.md)** (for audits).
3. Paste it into your AI tool (Cursor, ChatGPT, Claude, etc.), fill the placeholders, paste the contract code.
4. Attach **[`common/review-checklist.md`](common/review-checklist.md)** as context (or paste it in).
5. Run it. Review the output. Write a PoC for anything that looks real.

That's the basic flow. For significantly better results, read on for the full pipeline.

## How to Use

1. **Pick the directory** that matches your engagement type (`bug-bounty/`, `private-audits/`, `contests/`, or `zk-audits/`). Use `common/` prompts alongside any of them.
2. **Copy the prompt** into your AI tool of choice (ChatGPT, Claude, Cursor, etc.).
3. **Fill in the placeholders** — protocol name, contract code, chain, etc. The more context you provide, the better the output.
4. **Feed actual code.** Don't just describe the contract — paste the Solidity source directly into the prompt for real analysis.
5. **Chain prompts together.** These work best as a pipeline — see the recommended pipeline below.
6. **Iterate.** If the first output isn't useful, refine with more context or break the task into smaller pieces.

### Recommended Pipeline

For maximum coverage, chain the prompts in this order. Each step is marked as manual (you do it), AI-assisted (paste prompt into AI), or both.

| Step | What | File | Who does it |
|------|------|------|-------------|
| **1** | **Protocol Detection** — identify chain, protocol type, architecture, and type-specific attack vectors | [`common/protocol-detection.md`](common/protocol-detection.md) | AI — paste the prompt + code, get back a protocol profile |
| **2** | **Grep Surface Mapping** — run search patterns to map external calls, access control, dangerous ops | [`common/grep-patterns.md`](common/grep-patterns.md) | Manual — run the `rg`/`grep` commands in your terminal, save results |
| **3** | **Recon / Threat Model** — gather intelligence, map threat actors, prioritize attack surfaces | [`bug-bounty/recon-checklist.md`](bug-bounty/recon-checklist.md) or [`private-audits/threat-modeling.md`](private-audits/threat-modeling.md) | AI — paste the prompt + protocol info |
| **4** | **Build Custom Primer** — read the code yourself, note sensitive areas, hunches, questions | [`common/custom-primer.md`](common/custom-primer.md) | Manual — you read the code and write primer entries |
| **5** | **Run the Primer** — feed your primer entries + code to AI for targeted investigation | Template inside [`common/custom-primer.md`](common/custom-primer.md) | AI — paste the primer template + your entries + code |
| **6** | **Standard Review** — systematic checklist-based review with adversarial triager | [`audit-guide.md`](private-audits/audit-guide.md) or [`hunting-guide.md`](bug-bounty/hunting-guide.md) + [`common/review-checklist.md`](common/review-checklist.md) | AI — paste the prompt + code |
| **7** | **Multi-Expert Review** — 3-pass adversarial review (systematic → economic → skeptical triager) | [`common/multi-expert-review.md`](common/multi-expert-review.md) | AI — paste the prompt + code (use on highest-value contracts) |
| **8** | **Classify Findings** — structure each finding with attack flows, economic analysis, severity score | [`private-audits/finding-classification.md`](private-audits/finding-classification.md) + [`common/severity-assessment.md`](common/severity-assessment.md) | AI — paste the prompt + your raw finding |

**You don't need every step every time.** Pick a path based on your situation:

| Situation | Steps to use |
|-----------|-------------|
| Quick bug bounty hunt (limited time) | 1 → 4 → 6 |
| Serious bug bounty hunt | 1 → 2 → 4 → 5 → 6 → 8 |
| Private audit (full) | All 8 steps |
| Contest (time-boxed) | 1 → 2 → 6 → 7 → 8 |
| ZK circuit audit | 1 → 4 → [`zk-audits/hunting-guide.md`](zk-audits/hunting-guide.md) |

### Automated Pipeline (Panther Audit)

Don't want to run each step manually? **Panther Audit** (`claude-skill/`) is a Cursor/Claude skill that automates the full pipeline. It auto-detects codebase size and scales accordingly:

- **Small codebases (under 5k NSLOC):** Say `"audit src/Vault.sol"` and it runs 4 phases (context → dual-expert review → adversarial triage → structured report) automatically.
- **Large codebases (5k-30k+ NSLOC):** Say `"audit src/"` and it activates chunk mode — splits the codebase into modules, audits each with persistent state saved to `audit_state.json`, deduplicates findings across modules, runs cross-module analysis, and generates a consolidated report. Supports resuming interrupted audits.

See **[`claude-skill/README.md`](claude-skill/README.md)** for installation, walkthroughs for both modes, and the `audit_state.json` reference. The skill reads from the same `common/` files — so any customizations you make to the checklists apply to both the manual prompts and the automated skill.

### What's in `common/`

The `common/` directory contains shared prompts and tools used across all engagement types:

| File | What it does | When to use |
|------|-------------|-------------|
| [`review-checklist.md`](common/review-checklist.md) | 15-section vulnerability checklist + adversarial triager/verifier | Every review — this is the core checklist |
| [`multi-expert-review.md`](common/multi-expert-review.md) | 3-pass review: systematic auditor → economic attacker → skeptical triager | High-value contracts where you need maximum confidence |
| [`custom-primer.md`](common/custom-primer.md) | Guide + template for building protocol-specific primers from manual reading | When you want to turn your manual observations into AI-targeted prompts |
| [`protocol-detection.md`](common/protocol-detection.md) | Auto-detect chain, protocol type, architecture → get type-specific attack checklist | First step of any engagement — gives you a protocol profile |
| [`grep-patterns.md`](common/grep-patterns.md) | Ready-to-run search patterns for mapping attack surface | Start of engagement — run in terminal, takes 5 minutes |
| [`severity-assessment.md`](common/severity-assessment.md) | Quantitative severity formula (Impact × Likelihood × Exploitability) | When classifying any finding — prevents over/under-rating |
| [`defi-attack-vectors.md`](common/defi-attack-vectors.md) | Known DeFi attack patterns (flash loans, oracles, MEV, bridges, tokens) | When reviewing DeFi protocols |
| [`solidity-patterns.md`](common/solidity-patterns.md) | Common Solidity vulnerability patterns (reentrancy, access control, arithmetic) | Quick first-pass scan of any Solidity contract |
| [`move-patterns.md`](common/move-patterns.md) | Move/Sui/Aptos vulnerability patterns from 1141 real findings (generic type confusion, capability forgery, visibility bugs, accumulator exploits) | When reviewing Move contracts — auto-referenced by protocol detection |
| [`solana-patterns.md`](common/solana-patterns.md) | Solana/Anchor vulnerability patterns — 20 categories (account validation, PDA security, CPI safety, arithmetic, Token-2022, oracle manipulation, fee bypass, token dust DoS, state management, pool logic, timing, mint integrity) + framework-specific checks for Anchor and native Rust | When reviewing Solana programs — auto-referenced by protocol detection |

### Make It Your Own

All prompts are intentionally **generic** — they're not tied to any specific protocol category (lending, DEX, bridge, etc.). You can use them on whatever type of protocol you're auditing.

However, the best results come when you **append your own custom heuristics and checks** to `common/review-checklist.md` alongside the existing generic ones. Think of the checklist as a starting point — add your own category-specific patterns, past findings, and edge cases at the end, then run it. The generic + your custom checks together will give you significantly better output than either alone.

### Build a Custom Primer

For even better results, **build a protocol-specific primer before running any prompt.** Read the code manually, note every area that feels sensitive — old compiler versions, constants that don't match docs, EIP compliance gaps, missing guards, confusing logic — and compile these observations into a targeted primer. Then feed the primer to the AI alongside the contract code so it investigates exactly what you flagged instead of running a generic scan.

See **[`common/custom-primer.md`](common/custom-primer.md)** for the full step-by-step methodology, example entries, and a ready-to-use prompt template.

### Example Walkthrough

Here's what the pipeline looks like in practice when auditing a fictional lending protocol:

**Step 1 — Protocol Detection (AI):** Paste the contracts into the protocol detection prompt. The AI tells you it's a Solidity lending protocol on EVM, identifies Chainlink oracle dependency, and lists lending-specific attack vectors (oracle manipulation, liquidation logic, interest overflow, flash loan borrow attacks).

**Step 2 — Grep (Manual):** Run `rg "\.call\{" --glob "*.sol"` and the other patterns from `grep-patterns.md`. You find 12 external calls, 3 without reentrancy guards, and 2 unchecked return values. Save this list.

**Step 3 — Threat Model (AI):** Feed the protocol info into the threat modeling prompt. Get back a prioritized list: P0 is oracle manipulation → bad liquidations, P1 is flash loan collateral inflation.

**Step 4 — Custom Primer (Manual):** You read the contracts. You notice: the liquidation bonus is hardcoded at 10% but docs say 5%. The oracle staleness check allows 24-hour-old prices. The interest rate model uses `unchecked` math. You write these into primer entries.

**Step 5 — Run Primer (AI):** Paste the primer template + your entries + the contract code. The AI confirms the liquidation bonus mismatch is a real bug (Medium), finds the stale oracle creates a 24-hour arbitrage window (High), and dismisses the unchecked math as safe.

**Step 6 — Standard Review (AI):** Run the full checklist review. Catches additional issues: missing zero-address check on a critical setter, a reentrancy path through the liquidation callback.

**Step 7 — Multi-Expert (AI):** Run the 3-pass review on the core lending pool contract. Pass 1 finds 4 issues, Pass 2 finds 2 economic attacks, Pass 3 dismisses 3 as false positives and downgrades 1. Two survive as VALID.

**Step 8 — Classify (AI):** Feed each surviving finding into the classification template. Get structured reports with attack flows, economic analysis, and severity scores.

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
- [Forefy .context](https://github.com/anthropics/courses) — multi-expert review pattern, severity formula, and protocol detection heuristics adapted from their agentic audit framework
- [Move Vulnerability Database](https://movemaverick.github.io/move-vulnerability-database/) by MoveMaverick — 1141 findings from 200+ Move audit reports, used to build `common/move-patterns.md`
- [Safe Solana Builder](https://github.com/Frankcastleauditor/safe-solana-builder) by Frank Castle — Solana security rules for Anchor and native Rust, used to build `common/solana-patterns.md`
- [Trail of Bits Solana Security Research](https://blog.trailofbits.com/) — Solana vulnerability scanner patterns and audit methodology

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

## Disclaimer

These prompts are for **educational purposes** and are meant to **speed up** your audit workflow — not replace it. AI is a force multiplier, not a substitute for manual review.

**Before using any prompt, make sure you understand the protocol and how it works.** Read the docs, study the architecture, and trace the flows yourself first. If you don't understand what the code is doing, no prompt will magically find bugs for you. The output will be shallow, generic, and unreliable.

Manual review is non-negotiable. Use these prompts to augment your process, catch things you might miss, and structure your thinking — but your brain is still the primary tool.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding or improving prompts.

## License

MIT
