# Panther Audit

An automated smart contract security audit pipeline that scales with codebase size. Small codebases (under 5k NSLOC) get a direct 4-phase review. Large codebases (5k-30k+ NSLOC) automatically activate chunk mode — splitting the codebase into modules, auditing each with persistent state, deduplicating findings, and generating a consolidated report.

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
cp web3-sec-ai-prompts/claude-skill/chunk-pipeline.md ~/.cursor/skills/panther-audit/chunk-pipeline.md
```

That's it. Cursor will detect the skill automatically whenever you ask it to audit a contract.

**Verify it worked:** Open any project in Cursor, open the chat panel (Cmd+L), and type "what skills do you have?" — you should see `panther-audit` listed.

### Option B: Claude Code (CLI)

Claude Code reads files directly from the filesystem. No special installation needed — just make sure this repo is cloned somewhere accessible.

```bash
# Navigate to the protocol you want to audit
cd ~/projects/my-protocol

# Start Claude Code
claude

# Tell it where the skill is and what to audit
> Read ~/web3-sec-ai-prompts/claude-skill/SKILL.md and audit src/
```

### Option C: Claude.ai Projects (Web)

Claude.ai web can't read your filesystem, so you upload files manually. **Note:** Chunk mode with persistent state works best in Cursor or Claude Code. Claude.ai web can run standard mode but cannot write `audit_state.json`.

1. Go to [claude.ai](https://claude.ai) → Create a new **Project**
2. Upload these files to the Project knowledge:
   - `claude-skill/SKILL.md`
   - `claude-skill/chunk-pipeline.md`
   - `common/review-checklist.md`
   - `common/multi-expert-review.md`
   - `common/protocol-detection.md`
   - `common/severity-assessment.md`
   - `private-audits/finding-classification.md`
3. Set the Project instructions to: `Follow the audit pipeline defined in SKILL.md for all security reviews.`
4. Start a conversation → **paste your contract code** (this is the only setup where pasting is needed)

---

## How It Works: Two Modes

The skill auto-detects which mode to use based on your codebase size.

### Standard Mode (NSLOC <= 5000)

For small-to-medium codebases. The skill runs 4 phases per contract:

```
Phase 1: CONTEXT    →  Detect chain/type, map architecture, identify attack surface
Phase 2: REVIEW     →  Two independent expert passes (systematic + economic)
Phase 3: TRIAGE     →  Adversarial validation — try to disprove every finding
Phase 4: REPORT     →  Structured findings with severity scores and PoC guidance
```

You audit one contract at a time. The AI reads each file directly.

### Chunk Mode (NSLOC > 5000)

For large codebases that would exceed context limits. The skill automatically:

```
Phase 0: SCAN & PLAN       →  Count NSLOC, group files into modules, create audit plan
Phase 1: CONTEXT            →  Protocol detection (once for entire codebase)
Phase 2: MODULE AUDIT LOOP  →  For each module: review → triage → deduplicate → save state
Phase 3: CROSS-MODULE       →  Find issues spanning multiple modules
Phase 4: FINAL AGGREGATION  →  Merge findings, final validation, generate report
```

Key features of chunk mode:
- **Persistent state** — Findings saved to `audit_state.json` after each module
- **Deduplication** — Same root cause across modules is detected and merged
- **Resume support** — If interrupted, say "resume audit" and it picks up where it left off
- **Cross-module analysis** — Catches bugs that per-contract auditing misses (state coupling, access control gaps, fund flow breaks)

---

## Walkthrough: Small Codebase (Standard Mode)

Protocol: **Panther**, contracts in `src/`, ~3k NSLOC.

### Step 1: Open the project

**Cursor:** Open the Panther project folder in Cursor.

**Claude Code:** `cd ~/projects/panther-protocol && claude`

### Step 2: Start the audit

In Cursor's chat panel (Cmd+L or Cmd+I for Agent mode):

> Audit `src/PantherVault.sol` for security vulnerabilities. This is Panther, a DeFi vault protocol on EVM.

**What happens:** The skill counts lines, detects standard mode, and runs 4 phases. You'll see:

```
CODEBASE SCAN — Total NSLOC: 3200, Mode: STANDARD
--- PHASE 1 COMPLETE ---     ← Protocol profile
--- PASS A COMPLETE ---      ← Systematic findings
--- PASS B COMPLETE ---      ← Economic attack findings
--- PHASE 2 COMPLETE ---
--- PHASE 3 COMPLETE ---     ← Triage results
--- PHASE 4 COMPLETE ---     ← Final report
```

### Step 3: Review output, move to next contract

> Now audit `src/PantherRouter.sol`.

Repeat for each contract. Write PoCs for VALID and QUESTIONABLE findings.

---

## Walkthrough: Large Codebase (Chunk Mode)

Protocol: **Panther**, contracts in `src/`, ~20k NSLOC, 15 contracts.

### Step 1: Open the project and start

**Cursor:** Open project, then in chat:

> Audit the contracts in `src/` for security vulnerabilities. This is Panther, a DeFi lending protocol on EVM. Full scope — all contracts in src/.

### Step 2: Watch the scan and plan

The skill counts lines, detects chunk mode, and creates a plan:

```
CODEBASE SCAN — Total NSLOC: 20450, Mode: CHUNK

AUDIT PLAN
==========
Protocol:      Panther
Total NSLOC:   20450
Total modules: 8
Processing order:
  1. Vault (3200 lines, 3 files) — holds user funds
  2. LendingPool (2800 lines, 2 files) — core lending logic
  3. Liquidation (1800 lines, 2 files) — liquidation engine
  4. Router (1500 lines, 1 file) — routing and swaps
  5. Oracle (1200 lines, 2 files) — price feeds
  6. Governance (1100 lines, 2 files) — parameter control
  7. Token (900 lines, 1 file) — protocol token
  8. Libraries (1950 lines, 4 files) — shared math and utils

--- PHASE 0 COMPLETE ---
```

It also creates `audit_state.json` in your project root.

### Step 3: Watch the module loop

The skill audits each module sequentially:

```
--- PHASE 1 COMPLETE ---                        ← Protocol profile (once)

MODULE AUDIT: Vault (1 of 8)
--- MODULE Vault PASS A COMPLETE ---
--- MODULE Vault PASS B COMPLETE ---
--- MODULE Vault COMPLETE (1 of 8) ---          ← State saved

MODULE AUDIT: LendingPool (2 of 8)
Previously identified issues: 3 findings from Vault
--- MODULE LendingPool PASS A COMPLETE ---
--- MODULE LendingPool PASS B COMPLETE ---
--- MODULE LendingPool COMPLETE (2 of 8) ---    ← State saved, duplicates removed

... (continues for all 8 modules) ...

--- PHASE 2 COMPLETE — ALL MODULES AUDITED ---
```

### Step 4: Cross-module analysis and final report

```
--- PHASE 3 COMPLETE ---                        ← Cross-module issues found
--- PHASE 4 COMPLETE — CHUNK AUDIT FINISHED --- ← Final report in audit_report.md
```

The final report is saved to `audit_report.md` in your project root.

### Step 5: Review

- Open `audit_report.md` for the full consolidated report
- Open `audit_state.json` to see all findings, dismissed items, and module coverage
- Write PoCs for VALID and QUESTIONABLE findings

---

## Resuming an Interrupted Audit

If your session dies mid-audit (context limit, crash, timeout), your progress is saved in `audit_state.json`. Just start a new session and say:

> Resume the audit. The state file is at `audit_state.json`.

The skill reads the state file, identifies which modules are still pending, and continues from where it left off. It does NOT re-audit completed modules.

---

## The audit_state.json File

This file is created automatically in chunk mode. It tracks:

| Field | What it stores |
|-------|---------------|
| `protocol` | Protocol name, chain, language, type |
| `modules` | List of modules with files, NSLOC, priority, and completion status |
| `context` | Protocol profile from Phase 1 (shared across all modules) |
| `findings` | All VALID and QUESTIONABLE findings with severity, root cause, location |
| `dismissed` | All dismissed findings with reasons |

You can inspect it mid-audit to see progress. You can also manually edit it (e.g., to skip a module or re-run one).

---

## Custom Primer with Chunk Mode

If you've read the code and built primer entries (see `common/custom-primer.md`), include them when starting:

> Audit `src/` for security vulnerabilities. Here's my custom primer:
> - The oracle in `src/PantherOracle.sol` accepts 24-hour stale prices
> - `LendingPool.liquidate()` is missing the pause check
> - The governance timelock is 2 days but withdrawal delay is 7 — can governance rug?

The skill distributes primer entries to the relevant modules — oracle entries go to the Oracle module audit, lending entries go to the LendingPool module, etc.

---

## Skill vs Manual Prompts

| | Manual Prompts | Panther Audit (Standard) | Panther Audit (Chunk) |
|---|---|---|---|
| **Codebase size** | Any | Up to 5k NSLOC | 5k-30k+ NSLOC |
| **How it works** | Copy prompts, run one at a time | Say "audit this" → 4 phases auto | Say "audit src/" → auto-chunks and loops |
| **State persistence** | None | None | `audit_state.json` after each module |
| **Deduplication** | Manual | N/A (single contract) | Automatic cross-module dedup |
| **Resume support** | No | No | Yes — picks up from last completed module |
| **Cross-module bugs** | Manual only | Manual only | Dedicated Phase 3 analysis |
| **Best for** | Quick bounties, partial reviews | Small audits, single contracts | Full protocol audits, large codebases |

---

## Customization

The skill reads from `common/` files. To customize:

| What to change | File to edit |
|---------------|-------------|
| Add checklist items | `common/review-checklist.md` — add to sections 1-15 or create section 16 |
| Adjust severity thresholds | `common/severity-assessment.md` — edit the formula scores |
| Add protocol-type checks | `common/protocol-detection.md` — add attack vectors for new types |
| Change finding format | `private-audits/finding-classification.md` — edit the template |
| Modify chunking rules | `claude-skill/chunk-pipeline.md` — edit module grouping or size caps |

Changes apply to both the skill and the manual prompts automatically.
