# Web3 Security Audit Skill

An automated 4-phase security audit pipeline that runs as a Claude/Cursor skill. Instead of manually copying prompts and chaining them together, the skill orchestrates the full pipeline for you.

## What It Does

When you ask the AI to audit a smart contract, the skill automatically runs:

| Phase | What happens | References |
|-------|-------------|------------|
| **1. CONTEXT** | Detects chain, language, protocol type. Maps architecture, fund flows, privileged roles. Identifies type-specific attack vectors. | `common/protocol-detection.md` |
| **2. REVIEW** | Two independent expert passes — (A) systematic checklist review and (B) economic/integration attack analysis. Neither references the other until the end. | `common/review-checklist.md`, `common/multi-expert-review.md` |
| **3. TRIAGE** | Adversarial validation — actively tries to disprove every finding. Labels each as VALID, QUESTIONABLE, DISMISSED, or OVERCLASSIFIED. | Verifier in `common/review-checklist.md`, `common/severity-assessment.md` |
| **4. REPORT** | Structured findings with attack flows, economic analysis, severity scores, and PoC direction. Only VALID and QUESTIONABLE findings are reported. | `private-audits/finding-classification.md`, `common/severity-assessment.md` |

## Skill vs Manual Prompts

| | Manual Prompts | Skill |
|---|---|---|
| **How it works** | You copy prompts, fill placeholders, run them one at a time | You say "audit this contract" and the pipeline runs automatically |
| **Control** | Full — you pick which prompts to run and in what order | Less — the skill runs all 4 phases in sequence |
| **Tool support** | Any AI tool (ChatGPT, Claude, Cursor, etc.) | Cursor or Claude Code only |
| **Context** | You manually carry context between prompts | Context flows automatically between phases |
| **Custom primer** | You build it and paste it into the primer template | You build it and mention it — the skill incorporates it |
| **Best for** | Flexible workflows, quick bounty hunts, partial reviews | Full audits where you want maximum automated coverage |

**Both use the same underlying prompts and checklists from `common/`.** The skill is just an automated orchestrator on top.

## Installation

### Cursor (Recommended)

Copy the skill into your project so it's available when you open the repo:

```bash
# The skill is already in the repo at claude-skill/SKILL.md
# To make Cursor auto-discover it, copy to .cursor/skills/:

mkdir -p .cursor/skills/web3-security-audit
cp claude-skill/SKILL.md .cursor/skills/web3-security-audit/SKILL.md
```

Or to install it globally (available across all projects):

```bash
mkdir -p ~/.cursor/skills/web3-security-audit
cp claude-skill/SKILL.md ~/.cursor/skills/web3-security-audit/SKILL.md
```

After copying, Cursor's agent will automatically detect and use the skill when you ask it to audit a contract.

### Claude Code

```bash
# Clone the repo into your project (or alongside it)
git clone https://github.com/pantheraudits/web3-sec-ai-prompts.git

# Claude Code has full file access — it reads contracts directly, no pasting needed
# Just ask: "Read claude-skill/SKILL.md and audit the contracts in src/"
```

### Claude.ai Projects (Web)

Claude.ai web doesn't have file access, so you'll need to upload files and paste contract code manually.

1. Create a new Project in Claude.ai
2. Upload these files to the Project knowledge:
   - `claude-skill/SKILL.md`
   - `common/review-checklist.md`
   - `common/multi-expert-review.md`
   - `common/protocol-detection.md`
   - `common/severity-assessment.md`
   - `private-audits/finding-classification.md`
3. In the Project instructions, add: "Follow the audit pipeline defined in SKILL.md for all security reviews."
4. Start a conversation and paste your contract code (this is the only setup where pasting is required).

## Usage

> **Cursor and Claude Code can read your files directly.** You don't need to paste code — just point the AI at the contract or directory. Pasting is only needed for Claude.ai web or ChatGPT where the AI can't access your filesystem.

### Basic

**In Cursor:**
1. Open your project in Cursor
2. Open the chat panel (Cmd+L) or use Agent mode (Cmd+I)
3. Type your message — the skill triggers automatically:

> "Audit `src/LendingPool.sol` for security vulnerabilities. The protocol is [Name], a [type] protocol on EVM."

**In Claude Code:**
1. Open terminal in your project directory
2. Run `claude` to start a session
3. Type the same message

No extra setup beyond installation. The skill reads the file and runs all 4 phases.

### With a Custom Primer

If you've manually read the code and built primer entries (see `common/custom-primer.md` for how), include them:

> "Audit this contract. Here's my custom primer from manual review:
> - The oracle staleness check accepts 24-hour-old prices — is this safe?
> - nonReentrant is missing on the callback function
> - MAX_FEE constant is 10000 (100%) — can admin drain all funds?
> - Why is SQRT_RATIO_LIMIT set to 4295128739?"

The skill will investigate your primer entries as priority targets in Phase 2.

### Scoped Review

If you only want to review specific aspects:

> "Audit `src/Vault.sol`, focusing on the flash loan and liquidation paths."

The skill still runs all 4 phases but focuses the review on your specified areas.

### Multiple Contracts

Run one contract at a time for best results. The multi-expert review and triage work best when focused on a single contract — feeding everything at once dilutes the analysis.

> "Audit `src/Vault.sol` first. When you're done, I'll ask you to do `src/LendingPool.sol`."

The AI reads each file directly. Cross-contract interactions are noted during review, but each contract gets its own full pipeline run.

## What to Expect

A complete run produces:

1. **Protocol Profile** — chain, type, architecture, top attack vectors, threat actors
2. **Two independent review passes** — systematic findings + economic attack findings
3. **Triage results** — which findings survived adversarial validation and which were dismissed
4. **Structured report** — severity-scored findings with attack flows, economic analysis, and PoC direction
5. **Dismissed findings list** — what was checked and ruled out (so you know it was covered)

Expect the full pipeline to take a few minutes per contract. The output is longer than a single-prompt review but significantly higher quality with fewer false positives.

## Customization

The skill reads from `common/` files. To customize the pipeline:

- **Add your own checklist items** — Edit `common/review-checklist.md` sections 1-15 or add a section 16
- **Adjust severity thresholds** — Edit the formula in `common/severity-assessment.md`
- **Add protocol-specific checks** — Edit `common/protocol-detection.md` to add attack vectors for new protocol types
- **Change the finding format** — Edit `private-audits/finding-classification.md`

Changes to `common/` files automatically apply to both the skill and the manual prompts.
