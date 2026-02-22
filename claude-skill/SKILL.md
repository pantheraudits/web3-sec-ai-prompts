---
name: panther-audit
description: Automated smart contract security audit pipeline. Auto-detects codebase size and scales accordingly — standard mode for small codebases, chunk mode with persistent state for large ones. Runs context building, dual-expert review, adversarial triage, and structured reporting. Use when auditing smart contracts, reviewing Solidity/Move/Vyper/Rust code for vulnerabilities, or when the user asks to audit, review, or find bugs in Web3 code.
---

# Panther Audit

Automated pipeline that runs a full security review. Auto-detects codebase size and scales accordingly — small codebases get a direct 4-phase review, large codebases activate chunk mode with persistent state, deduplication, and cross-module analysis.

## When to Use

Use when the user:
- Asks to audit or review a smart contract
- Wants to find bugs or vulnerabilities in Web3 code
- Shares contract code and asks for security analysis
- Mentions bug bounty hunting, audit, or security review

Do NOT use for:
- General Solidity questions or code generation
- Non-security code reviews
- Architecture design (without security focus)

## Before Starting

Determine what you're working with:

1. **Contract code** — The user must provide actual code (pasted or file path). Do not proceed without it.
2. **Scope** — Which contracts/directory are in scope? If unclear, ask.
3. **Known issues** — Any previously disclosed bugs to skip? If unclear, assume none.
4. **Custom primer** (optional) — If the user has built a primer using `common/custom-primer.md`, incorporate their entries into the review phase.

---

## Step 0: SIZE DETECTION

Before any audit work, determine the codebase size to select the right mode.

**Run this command** (adapt file extensions to the language):

```bash
find [SCOPE_DIR] -name "*.sol" -o -name "*.vy" -o -name "*.rs" -o -name "*.move" | xargs wc -l
```

**Also check** if a previous `audit_state.json` exists in the project root. If it does and has modules with status "pending", this is a **resumed audit** — read `chunk-pipeline.md` and continue from where it left off.

**Route based on total line count:**

- **NSLOC <= 5000** → Run **STANDARD MODE** (Phase 1-4 below)
- **NSLOC > 5000** → Run **CHUNK MODE** — read and follow [`chunk-pipeline.md`](chunk-pipeline.md)

State the detected mode:
```
CODEBASE SCAN
=============
Total files:  [N]
Total NSLOC:  [N]
Mode:         STANDARD / CHUNK
```

If CHUNK MODE, stop here and switch to `chunk-pipeline.md`. Everything below is STANDARD MODE only.

---

# STANDARD MODE (NSLOC <= 5000)

For small-to-medium codebases that fit comfortably in context. This is the original 4-phase pipeline.

```
Phase 1: CONTEXT    →  Detect chain/type, map architecture, identify attack surface
Phase 2: REVIEW     →  Two independent expert passes (systematic + economic)
Phase 3: TRIAGE     →  Adversarial validation — try to disprove every finding
Phase 4: REPORT     →  Structured findings with severity scores and PoC guidance
```

Each phase produces output that feeds into the next. Do NOT skip phases or reorder them.

---

## Phase 1: CONTEXT

Build a protocol profile before reviewing any code. This phase combines protocol detection, architecture mapping, and threat surface identification into one pass.

**Read** `common/protocol-detection.md` for the full detection prompt and type-specific attack checklists.

**Produce the following:**

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
1. [Most likely vector]
2. [Second most likely]
3. [Third]
4. [Fourth]
5. [Fifth]

THREAT ACTORS:
- External attacker: [capabilities, likely targets]
- Flash loan attacker: [single-tx opportunities]
- MEV bot: [sandwich/frontrun targets]
- Compromised admin: [worst-case blast radius]
```

**This profile is context for all subsequent phases.** Keep it visible.

State: `--- PHASE 1 COMPLETE ---`

---

## Phase 2: REVIEW

Two independent expert passes over the contract code. Each pass has a different focus to maximize coverage.

**Read** `common/review-checklist.md` for the 15-section checklist and `common/multi-expert-review.md` for the full multi-expert methodology.

If the user provided a custom primer, investigate every primer entry during BOTH passes.

### Pass A: Systematic Auditor

Work through the code methodically using the checklist from `common/review-checklist.md` (sections 1-15).

Focus:
- Start with highest-risk functions (payable, external calls, admin)
- Map all fund flow paths and state changes
- Check every item in the review checklist against the code
- Reference the protocol profile from Phase 1 to focus on type-specific risks

For each potential finding, note:
- Location (contract:function:lines)
- What's wrong
- Who can exploit it
- Preliminary severity

State: `--- PASS A COMPLETE ---`

### Pass B: Economic & Integration Attacker

Start fresh. Do NOT reference Pass A findings yet.

Focus:
- Single-transaction profit opportunities (flash loans, price manipulation)
- Multi-step economic attacks (governance, liquidation cascades, bad debt)
- Cross-contract interaction risks (callbacks, reentrancy via integrations)
- Token economics misalignment (share price manipulation, donation attacks)
- MEV extraction (front-running, sandwich, JIT)
- Oracle risks (stale prices, manipulation, L2 sequencer downtime)

**For each potential finding, tell two stories:**
- INNOCENT USER: What does a normal user expect? (e.g., "Alice deposits 100 USDC, expects proportional shares")
- ATTACKER: What does the attacker do? Step by step. (e.g., "Bob flash loans → inflates price → Alice gets fewer shares → Bob profits")

If you cannot write a concrete attacker story, the finding is likely invalid.

**After your independent review:**
- List your own findings first
- Then review Pass A — state agreements, disagreements, and why

State: `--- PASS B COMPLETE ---`

State: `--- PHASE 2 COMPLETE ---`

---

## Phase 3: TRIAGE

Adversarial validation. Your job is to DISPROVE every finding from Phase 2.

**Read** the Verifier & Triager section in `common/review-checklist.md` for the full validation methodology.

**For EVERY finding from Pass A and Pass B, answer:**

1. Can you prove this is NOT exploitable? (trace the full call path — are there upstream/downstream checks?)
2. Is the attack economically rational? (cost to attack vs profit — include gas, slippage, capital)
3. Does it require unrealistic preconditions? (admin malice, specific token behavior, exact timing)
4. Do existing protections block it? (reentrancy guards, timelocks, slippage checks, pause mechanisms)
5. Is the severity accurate? (apply the formula from `common/severity-assessment.md`: Impact × Likelihood × Exploitability, each scored 1-3)

**Label each finding:**
- **VALID** — Survives all checks. Exploitable with realistic conditions.
- **QUESTIONABLE** — Cannot fully disprove, cannot fully prove. Needs PoC.
- **DISMISSED** — Disproven. Explain exactly why.
- **OVERCLASSIFIED** — Real but severity is wrong. State correct severity.

State: `--- PHASE 3 COMPLETE ---`

---

## Phase 4: REPORT

Structured report containing ONLY findings labeled VALID or QUESTIONABLE.

**Read** `common/severity-assessment.md` for the severity formula and `private-audits/finding-classification.md` for the output format.

### Format each finding as:

```
### [C/H/M/L]-[Number]: [Impact] via [Weakness] in [Feature]

**Status:** VALID / QUESTIONABLE
**Severity:** [Critical/High/Medium/Low]
**Confidence:** [High/Medium/Low]
**Location:** contract.sol:function:lines XX-YY

**Innocent User Story:**
[Normal user expectation — 1-2 sentences]

**Attack Flow:**
1. [Setup / preconditions]
2. [Trigger the vulnerability]
3. [Exploit / extract value]
4. [Profit / damage]

**Economic Analysis:**
- Attack cost: [gas + capital + slippage]
- Attack profit: [stolen funds / damage]
- Profitable: [Yes/No]

**Why This Survived Triage:**
[Which disproval attempts failed and why]

**Recommended Fix:**
[Specific code change]

**PoC Direction:**
[How to write a Foundry/Hardhat test to prove this]
```

### Report order: Critical → High → Medium → Low. VALID before QUESTIONABLE within each severity.

### After the findings, include:

**Summary table:**
| ID | Title | Severity | Status | Location |
|----|-------|----------|--------|----------|
| C-1 | ... | Critical | VALID | ... |

**What was checked and dismissed:**
Brief list of findings that were dismissed in Phase 3 and why — so the user knows those areas were reviewed.

State: `--- PHASE 4 COMPLETE — AUDIT FINISHED ---`

---

## Important Rules

- **Never skip phases.** Even if the contract looks simple, run all 4 phases.
- **Never fabricate findings.** If the code is solid, say so. An empty report is better than false positives.
- **If borderline severity, round DOWN.** Over-classification wastes everyone's time.
- **Standard mode: one contract at a time.** If multiple contracts are in scope, run the full pipeline on each separately. Cross-contract interactions should be noted but reviewed per-contract.
- **Chunk mode: follow chunk-pipeline.md.** Do not try to manually manage large codebases — let the chunking system handle it.
- **Custom primers take priority.** If the user provides primer entries, they've already done manual review. Treat their observations as high-priority investigation targets.
- **The user is the final reviewer.** Flag findings for PoC verification. Never claim certainty without proof.
