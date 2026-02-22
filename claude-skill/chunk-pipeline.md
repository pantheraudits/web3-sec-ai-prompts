# Chunk Pipeline (Large Codebase Mode)

This file is read by the panther-audit skill when NSLOC > 5000. It extends the standard 4-phase audit into a modular pipeline with persistent state, deduplication, and cross-module analysis.

**Do NOT read this file for codebases under 5000 NSLOC.** Use the standard mode in SKILL.md instead.

---

## How Chunk Mode Works

```
Phase 0: SCAN & PLAN       →  Map files, group into modules, create audit plan
Phase 1: CONTEXT            →  Protocol detection (once for entire codebase)
Phase 2: MODULE AUDIT LOOP  →  For each module: review → triage → deduplicate → save state
Phase 3: CROSS-MODULE       →  Find issues spanning multiple modules
Phase 4: FINAL AGGREGATION  →  Merge, validate, generate consolidated report
```

All findings are persisted to `audit_state.json` after each module. This enables:
- Resuming interrupted audits
- Deduplication across modules
- Cross-module analysis with full context

---

## Phase 0: SCAN AND PLAN

Map the codebase and create a structured audit plan.

### Step 0.1: List all files with NSLOC

Run:
```bash
find [SCOPE_DIR] -name "*.sol" -o -name "*.vy" -o -name "*.rs" -o -name "*.move" | xargs wc -l | sort -rn
```

Record each file and its line count.

### Step 0.2: Group files into modules

Group files into logical modules using these rules (in priority order):

1. **Name root grouping** — Files sharing a name root belong together:
   - `Vault.sol` + `VaultLib.sol` + `VaultStorage.sol` + `IVault.sol` → Module "Vault"
2. **Import grouping** — Files that heavily import each other belong together. Read the import statements to identify clusters.
3. **Directory grouping** — Files in the same subdirectory that don't fit rules 1-2 form a module.
4. **Standalone** — Files that don't group with anything = individual modules.

**Module size cap: ~2000 NSLOC.** If a module exceeds this, split it into sub-modules by logical boundary (e.g., separate the library from the main contract). If a single file exceeds 2000 lines, it becomes its own module — the AI will need to focus on sections rather than the whole file.

### Step 0.3: Determine processing order

Order modules by risk priority:
1. **Fund-holding contracts** — Vaults, pools, treasuries, token contracts (highest priority)
2. **Core logic** — Routers, lending engines, swap logic, liquidation
3. **Access control & governance** — Admin functions, timelocks, multisigs
4. **Oracles & external integrations** — Price feeds, bridges, cross-chain messaging
5. **Libraries & utilities** — Math libraries, helpers, abstract bases (lowest priority)

### Step 0.4: Initialize audit_state.json

Write the following JSON file to the project root:

```json
{
  "protocol": "[PROTOCOL_NAME]",
  "chain": "[CHAIN]",
  "language": "[LANGUAGE + VERSION]",
  "protocol_type": "[TYPE]",
  "mode": "chunk",
  "total_nsloc": 0,
  "modules": [],
  "context": {
    "protocol_profile": "",
    "attack_vectors": [],
    "privileged_roles": []
  },
  "findings": [],
  "dismissed": []
}
```

Populate `total_nsloc` and `modules` with the data from Steps 0.1-0.3. Each module entry:

```json
{
  "name": "Vault",
  "files": ["src/Vault.sol", "src/VaultLib.sol"],
  "nsloc": 1200,
  "priority": 1,
  "status": "pending"
}
```

State:
```
AUDIT PLAN
==========
Protocol:     [Name]
Total NSLOC:  [N]
Total modules:[N]
Processing order:
  1. [Module] ([NSLOC] lines, [N] files) — [reason for priority]
  2. [Module] ([NSLOC] lines, [N] files)
  ...
```

State: `--- PHASE 0 COMPLETE ---`

---

## Phase 1: CONTEXT (Once)

This runs exactly once for the entire protocol — NOT per module.

**Read** `common/protocol-detection.md` for the full detection prompt and type-specific attack checklists.

Produce the same protocol profile as standard mode:

```
PROTOCOL PROFILE
================
Chain:           [EVM / Solana / Cosmos / etc.]
Language:        [Solidity X.X.X / Vyper / Rust-Anchor / etc.]
Protocol type:   [Lending / DEX / Bridge / Vault / etc.]
Contracts:       [List each contract and its role — 1 line each]
Fund holders:    [Which contracts hold user funds]
External deps:   [Oracles, protocols, token standards]
Privileged roles:[List each role and what it can do]
Upgrade pattern: [Proxy type / immutable / etc.]

TOP ATTACK VECTORS (for this protocol type):
1-5. [Ranked list]

THREAT ACTORS:
- External attacker / Flash loan / MEV bot / Compromised admin
```

**Save this profile** to `audit_state.json` under the `context` field. This profile is injected as context into every module audit.

State: `--- PHASE 1 COMPLETE ---`

---

## Phase 2: MODULE AUDIT LOOP

Process each module sequentially. For every module, run a focused review + triage cycle, then save results before moving to the next.

### For each module in processing order:

#### Step 2.1: Load state and build context

Read `audit_state.json`. Build a context summary for the AI:

```
MODULE AUDIT: [Module Name] ([N] of [Total])
Files: [list]
NSLOC: [N]

PROTOCOL CONTEXT (from Phase 1):
[Paste the protocol profile summary — type, chain, attack vectors, roles]

PREVIOUSLY IDENTIFIED ISSUES ([N] findings so far):
[For each previous finding, list ONE line: "ID - Title - Severity - Module - Root cause"]

INSTRUCTION: Do NOT duplicate the above findings. Focus ONLY on new issues in this module. If you find a variant of an existing finding (same root cause, different location), tag it as a variant.
```

#### Step 2.2: Pass A — Systematic Auditor

**Read** `common/review-checklist.md` (sections 1-15).

Work through this module's files using the checklist. Same methodology as standard mode Pass A, but scoped to this module only.

Focus on:
- Highest-risk functions in THIS module
- Fund flow paths within and through this module
- Every checklist item against this module's code
- Type-specific risks from the protocol profile

If the user provided a custom primer, investigate relevant primer entries during this pass.

State: `--- MODULE [Name] PASS A COMPLETE ---`

#### Step 2.3: Pass B — Economic & Integration Attacker

Start fresh from this module's code. Do NOT reference Pass A yet.

Same methodology as standard mode Pass B, but scoped to this module. Pay special attention to:
- How this module interacts with already-audited modules
- Trust assumptions this module makes about inputs from other modules
- Economic attacks that could span this module and previously reviewed ones

After independent review, cross-reference with Pass A.

State: `--- MODULE [Name] PASS B COMPLETE ---`

#### Step 2.4: Triage this module's findings

**Read** the Verifier & Triager section in `common/review-checklist.md`.

Apply the same 5-point adversarial validation to every finding from this module. Label each: VALID / QUESTIONABLE / DISMISSED / OVERCLASSIFIED.

#### Step 2.5: Deduplicate

Before saving, compare each surviving finding (VALID or QUESTIONABLE) against existing findings in `audit_state.json`:

| Comparison | Result | Action |
|-----------|--------|--------|
| Same root cause AND same location | Exact duplicate | **Skip** — do not save |
| Same root cause AND different location | Variant | **Keep** — save with `"related_to": "ID"` tag |
| Different root cause | New finding | **Keep** — save normally |

Root cause comparison is semantic, not string-matching. Examples of "same root cause":
- Both are "missing reentrancy guard" → same root cause even if different functions
- "Missing access control on setFee" vs "Missing access control on setOracle" → same root cause (missing access control)
- "Oracle staleness" vs "Reentrancy" → different root causes

#### Step 2.6: Save state

Update `audit_state.json`:
1. Append all new findings (VALID and QUESTIONABLE) to the `findings` array
2. Append dismissed findings to the `dismissed` array
3. Set this module's status to `"completed"`

Each finding entry:
```json
{
  "id": "[C/H/M/L]-[N]",
  "module": "[Module Name]",
  "title": "[Short title]",
  "severity": "[Critical/High/Medium/Low]",
  "root_cause": "[Category: reentrancy / access-control / arithmetic / oracle / etc.]",
  "location": "[file:function:lines]",
  "status": "VALID",
  "related_to": null,
  "summary": "[One-sentence description of the issue]"
}
```

State: `--- MODULE [Name] COMPLETE ([N] of [Total]) ---`

#### Repeat for next module.

State after all modules: `--- PHASE 2 COMPLETE — ALL MODULES AUDITED ---`

---

## Phase 3: CROSS-MODULE ANALYSIS

After all modules are individually audited, analyze interactions BETWEEN modules.

Read `audit_state.json` to load the full findings list and protocol context.

**Check for systemic issues:**

1. **State coupling** — Is a state variable written in Module A and read in Module B without re-validation? Does Module B assume Module A has already validated the data?

2. **Access control inconsistency** — Is the same action restricted in one module but unrestricted in another? Are role checks consistent across all modules?

3. **Fund flow gaps** — Trace the complete path of funds across modules (deposit → route → strategy → withdraw). Are there gaps where funds could get stuck, double-counted, or drained at a module boundary?

4. **Shared library assumptions** — If a math library or utility is used differently in different modules, could one usage be safe while another is not?

5. **Event/callback chains** — Does an event or callback in Module A trigger logic in Module B? Could the ordering or reentrancy between modules create an exploit?

6. **Upgrade interactions** — If modules are upgradeable independently, could upgrading one break assumptions in another?

For each cross-module finding, save to `audit_state.json` with `"module": "cross-module"`.

State: `--- PHASE 3 COMPLETE ---`

---

## Phase 4: FINAL AGGREGATION

Generate the consolidated report from all findings.

### Step 4.1: Final validation pass

Read ALL findings from `audit_state.json`. Now that you have full codebase context, re-evaluate:
- Any finding that seemed valid in isolation but is actually prevented by code in another module? **Dismiss it.**
- Any finding whose severity should change given the full picture? **Reclassify it.**
- Any QUESTIONABLE findings that can now be resolved with full context? Upgrade to VALID or dismiss.

### Step 4.2: Merge variants

Group findings that share the same root cause:
- If 3 functions all have the same "missing access control" issue, merge into one finding with multiple locations listed.
- Keep the highest severity from the group.

### Step 4.3: Generate final report

**Read** `common/severity-assessment.md` for the severity formula and `private-audits/finding-classification.md` for the output format.

Format each finding using the same template as standard mode Phase 4:

```
### [C/H/M/L]-[Number]: [Impact] via [Weakness] in [Feature]

**Status:** VALID / QUESTIONABLE
**Severity:** [Critical/High/Medium/Low]
**Confidence:** [High/Medium/Low]
**Module(s):** [Which module(s) this spans]
**Location:** contract.sol:function:lines XX-YY

**Innocent User Story:**
[Normal user expectation]

**Attack Flow:**
1. [Setup]
2. [Trigger]
3. [Exploit]
4. [Profit/damage]

**Economic Analysis:**
- Attack cost: [gas + capital + slippage]
- Attack profit: [stolen funds / damage]
- Profitable: [Yes/No]

**Why This Survived Triage:**
[Which disproval attempts failed and why]

**Recommended Fix:**
[Specific code change]

**PoC Direction:**
[How to write a test to prove this]
```

Report order: Critical → High → Medium → Low. VALID before QUESTIONABLE.

### Step 4.4: Include report metadata

**Summary table:**
| ID | Title | Severity | Status | Module | Location |
|----|-------|----------|--------|--------|----------|
| C-1 | ... | Critical | VALID | Vault | ... |

**Audit coverage:**
| Module | NSLOC | Status | Findings |
|--------|-------|--------|----------|
| Vault | 1200 | Completed | 3 |
| Router | 800 | Completed | 1 |
| ... | ... | ... | ... |

**What was checked and dismissed:**
Brief list of dismissed findings and why.

### Step 4.5: Write report to disk

Save the final report as `audit_report.md` in the project root.

Update `audit_state.json`: set all module statuses to `"completed"`, add a `"report_generated": true` field.

State: `--- PHASE 4 COMPLETE — CHUNK AUDIT FINISHED ---`

---

## Resuming an Interrupted Audit

If the user says "resume audit" or the skill detects an existing `audit_state.json` with pending modules:

1. Read `audit_state.json`
2. Load the protocol context from Phase 1
3. Load all existing findings
4. Identify the first module with `"status": "pending"`
5. Resume Phase 2 from that module — inject the full context summary including all previous findings
6. Continue the loop from there

Do NOT re-run Phase 0 or Phase 1 on resume — they're already saved in the state file.

State on resume: `--- RESUMING AUDIT — [N] of [Total] modules remaining ---`

---

## Important Rules for Chunk Mode

- **Always save state after each module.** If the session dies mid-audit, the user can resume.
- **Never load full file contents from previous modules into context.** Only load the finding summaries (one line each). This keeps context lean.
- **The 2000 NSLOC cap is a guideline, not a hard limit.** If splitting a module at 2100 lines would break a logical unit, keep it together.
- **Cross-module analysis (Phase 3) is where chunk mode finds bugs that per-contract auditing misses.** Do not skip it.
- **The deduplication check is critical.** Without it, the same reentrancy bug found in 5 modules becomes 5 duplicate findings. Deduplicate by root cause.
- **If the user provides a custom primer, distribute primer entries to the relevant modules.** Don't check every primer entry against every module — match entries to the modules they reference.
