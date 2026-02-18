# Panther Audit

An automated 4-phase smart contract security audit pipeline. Instead of manually copying prompts and chaining them together, Panther Audit orchestrates the full pipeline — context building, dual-expert review, adversarial triage, and structured reporting — in a single run.

## Prerequisites

Before using Panther Audit, you need:

1. **This repo cloned** on your machine:
   ```bash
   git clone https://github.com/pantheraudits/web3-sec-ai-prompts.git
   ```
2. **One of these tools installed** (pick whichever you use):
   - [Cursor](https://cursor.sh) (recommended — has built-in agent + file access)
   - [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI tool with file access)
   - [Claude.ai](https://claude.ai) Projects (web — works but requires manual file uploads)
3. **A protocol to audit** — contract source code accessible on your machine

---

## Installation

**Pick ONE option** based on which tool you use. You don't need to do all three.

### Option A: Cursor (Recommended)

Cursor auto-discovers skills from `.cursor/skills/` directories. Install globally so the skill is available in every project:

```bash
mkdir -p ~/.cursor/skills/panther-audit
cp web3-sec-ai-prompts/claude-skill/SKILL.md ~/.cursor/skills/panther-audit/SKILL.md
```

That's it. Cursor will detect the skill automatically whenever you ask it to audit a contract.

**Verify it worked:** Open any project in Cursor, open the chat panel (Cmd+L), and type "what skills do you have?" — you should see `panther-audit` listed.

### Option B: Claude Code (CLI)

Claude Code reads files directly from the filesystem. No special installation needed — just make sure this repo is cloned somewhere accessible.

```bash
# Navigate to the protocol you want to audit
cd ~/projects/panther-protocol

# Start Claude Code
claude

# Tell it where the skill is and what to audit
> Read ~/web3-sec-ai-prompts/claude-skill/SKILL.md and audit src/Vault.sol
```

### Option C: Claude.ai Projects (Web)

Claude.ai web can't read your filesystem, so you upload files manually.

1. Go to [claude.ai](https://claude.ai) → Create a new **Project**
2. Upload these 6 files to the Project knowledge:
   - `claude-skill/SKILL.md`
   - `common/review-checklist.md`
   - `common/multi-expert-review.md`
   - `common/protocol-detection.md`
   - `common/severity-assessment.md`
   - `private-audits/finding-classification.md`
3. Set the Project instructions to: `Follow the audit pipeline defined in SKILL.md for all security reviews.`
4. Start a conversation → **paste your contract code** (this is the only setup where pasting is needed)

---

## How to Audit a Protocol (Step by Step)

Let's say you're auditing a protocol called **Panther** with contracts in `src/`, ~5k nSLOC.

### Step 1: Open the project

**Cursor:** Open the Panther project folder in Cursor.

**Claude Code:** `cd ~/projects/panther-protocol && claude`

### Step 2: Identify your highest-value contracts

Before running anything, figure out which contracts hold user funds or handle critical logic. These go first. For example:
- `src/PantherVault.sol` — where user deposits live
- `src/PantherRouter.sol` — handles swaps and routing
- `src/PantherOracle.sol` — price feeds
- `src/PantherGovernance.sol` — parameter changes

### Step 3: Audit one contract at a time

Start with the highest-value contract. In Cursor's chat panel (Cmd+L or Cmd+I for Agent mode), type:

> Audit `src/PantherVault.sol` for security vulnerabilities. This is Panther, a DeFi vault protocol on EVM. Scope is the Vault contract — it handles user deposits, withdrawals, and yield strategy routing.

**What happens next:** The skill automatically runs 4 phases. You'll see:

```
--- PHASE 1 COMPLETE ---     ← Protocol profile (chain, type, attack vectors, threat actors)
--- PASS A COMPLETE ---      ← Systematic checklist findings
--- PASS B COMPLETE ---      ← Economic/integration attack findings
--- PHASE 2 COMPLETE ---
--- PHASE 3 COMPLETE ---     ← Triage results (VALID / DISMISSED for each finding)
--- PHASE 4 COMPLETE ---     ← Final structured report
```

This takes a few minutes per contract. Don't interrupt it mid-phase.

### Step 4: Review the output

The final report contains:
- Findings sorted Critical → High → Medium → Low
- Each finding has: attack flow, economic analysis, severity score, PoC direction
- A list of what was checked and dismissed (so you know what's covered)

**Your job now:**
- Read each VALID finding — does the attack flow make sense?
- Write a Foundry/Hardhat PoC for anything that looks real
- Check QUESTIONABLE findings manually — the AI couldn't fully prove or disprove them

### Step 5: Move to the next contract

> Now audit `src/PantherRouter.sol`. Same protocol context as before.

Repeat for each contract in scope. The AI keeps the protocol context from Phase 1.

### Step 6 (Optional): Run with a custom primer

If you read the code manually before running the skill and noticed things that felt off, include your observations. This massively improves results:

> Audit `src/PantherOracle.sol`. Here's my custom primer from manual review:
> - The staleness check accepts prices up to 24 hours old — seems too wide
> - `updatePrice()` has no access control — can anyone push a price?
> - PRICE_PRECISION is 1e8 but the docs say 1e18
> - Why is there no sequencer uptime check for L2 deployments?

The skill investigates each primer entry as a priority target. See `common/custom-primer.md` for how to build effective primer entries.

---

## Skill vs Manual Prompts

| | Manual Prompts | Panther Audit Skill |
|---|---|---|
| **How it works** | You copy prompts, fill placeholders, run them one at a time | You say "audit this contract" and the pipeline runs automatically |
| **Control** | Full — you pick which prompts to run and in what order | Less — the skill runs all 4 phases in sequence |
| **Tool support** | Any AI tool (ChatGPT, Claude, Cursor, etc.) | Cursor, Claude Code, or Claude.ai Projects |
| **Context** | You manually carry context between prompts | Context flows automatically between phases |
| **Best for** | Flexible workflows, quick bounty hunts, partial reviews | Full audits where you want maximum automated coverage |

Both use the same underlying checklists from `common/`. The skill is just an automated orchestrator on top. You can mix both — run the skill for the full audit, then use individual manual prompts for deep dives on specific areas.

---

## What to Expect

A complete run per contract produces:

1. **Protocol Profile** — chain, type, architecture, top 5 attack vectors, threat actors
2. **Two independent review passes** — systematic findings + economic attack findings
3. **Triage results** — which findings survived adversarial validation and which were dismissed (with reasons)
4. **Structured report** — severity-scored findings with attack flows, economic analysis, and PoC direction
5. **Dismissed findings list** — what was checked and ruled out

The output is longer than a single-prompt review but significantly higher quality with fewer false positives.

---

## Customization

The skill reads from `common/` files. To customize:

| What to change | File to edit |
|---------------|-------------|
| Add checklist items | `common/review-checklist.md` — add to sections 1-15 or create section 16 |
| Adjust severity thresholds | `common/severity-assessment.md` — edit the formula scores |
| Add protocol-type checks | `common/protocol-detection.md` — add attack vectors for new types |
| Change finding format | `private-audits/finding-classification.md` — edit the template |

Changes apply to both the skill and the manual prompts automatically.
