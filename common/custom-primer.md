# Custom Primer Guide

How to build a protocol-specific primer that turns generic AI prompts into laser-focused audit tools.

## What Is a Custom Primer?

A custom primer is a list of observations, questions, and hunches you collect **while manually reading the codebase** — before you ever ask the AI to find bugs. Instead of throwing raw code at a generic prompt and hoping for the best, you feed the AI a curated list of "here's what I noticed, here's what smells off, here's what I need you to dig into."

The primer is not a checklist you copy from someone else. It's **your** observations from **this** codebase. That's what makes it powerful — you're pointing the AI at the exact spots where bugs are most likely hiding, based on your own reading.

## Why It Works

Generic prompts produce generic output. The AI doesn't know which parts of the codebase are unusual, which design decisions are suspicious, or which assumptions the developers might have gotten wrong. **You do** — after reading the code. The primer bridges that gap.

A well-built primer:
- Eliminates noise by focusing the AI on areas you've already flagged as sensitive.
- Catches bugs you sensed but couldn't fully trace during manual review.
- Surfaces issues across files — the AI can cross-reference your hunches against the full codebase.
- Turns vague "something feels off" instincts into concrete, answerable questions.

## How to Build a Primer (Step by Step)

Open the contracts. Read them top to bottom. Don't try to find bugs yet — just **observe and record**. Every time something catches your eye, write it down as a primer entry.

### Step 1: Compiler & Language-Level Observations

As soon as you open the first file, note the basics.

**What to look for:**
- Solidity version — old, floating, or pinned?
- Optimizer settings if visible
- Unusual imports or compiler flags
- Use of `unchecked` blocks, inline assembly, or `delegatecall`

**Example primer entries:**
```
- Why is this project using ^0.5.16? What known issues exist in this compiler version that could affect the code?
- Contract X uses unchecked blocks in 4 places — verify that every unchecked arithmetic operation is genuinely safe and cannot overflow/underflow.
- Inline assembly is used in the swap function — trace every memory operation and verify it doesn't corrupt adjacent slots.
```

### Step 2: Constants, Magic Numbers & Configuration

Check every hardcoded value. Compare against docs, specs, and comments.

**What to look for:**
- Constants that don't match the whitepaper or docs
- Magic numbers without explanation
- Fee percentages, precision factors, time durations
- Values that look copy-pasted from another protocol

**Example primer entries:**
```
- PRECISION_FACTOR is set to 1e12 but the docs say the protocol uses 18-decimal precision — is this a mismatch?
- MAX_FEE is 10000 (100%) — can the admin actually set fee to 100% and drain all funds?
- WITHDRAWAL_DELAY is 7 days but the governance timelock is 2 days — can governance change parameters and rug users before the withdrawal delay expires?
- The constant SQRT_RATIO_LIMIT = 4295128739 — where does this number come from? Is it correct for the tick range being used?
```

### Step 3: Standards & EIP Compliance

When you see the contract implements or references an EIP, note it.

**What to look for:**
- Which EIPs are claimed (ERC20, ERC721, ERC4626, EIP-2612, ERC-2981, etc.)
- Whether the implementation matches the spec
- Optional vs required functions
- Edge cases in the standard that protocols commonly get wrong

**Example primer entries:**
```
- This vault claims ERC4626 compliance — check all 9 required functions against the EIP-4626 spec. Specifically verify: (1) preview functions match actual execution, (2) maxDeposit/maxWithdraw return correct limits, (3) rounding direction favors the vault not the user.
- Contract uses EIP-2612 permit — verify the DOMAIN_SEPARATOR is built correctly, the nonce increments properly, and replay protection works across chains and after upgrades.
- The NFT contract says ERC721 but doesn't implement ERC165 — will integrating protocols fail to detect it?
```

### Step 4: Checks, Guards & Modifiers

When you see a check (pause, access control, reentrancy guard, deadline), ask two questions: where is it missing, and where is it unnecessary?

**What to look for:**
- `whenNotPaused` — is it on every function that should be pausable?
- `nonReentrant` — is it on every function that does external calls before state updates?
- `onlyOwner` / role checks — are there functions that should be restricted but aren't?
- Deadline or slippage checks — present in some paths but missing in others?

**Example primer entries:**
```
- The pause modifier is on deposit() and withdraw() but NOT on liquidate() — should liquidations be pausable? What happens if the protocol is paused but liquidations keep running?
- nonReentrant is on swap() but not on the callback function — can an attacker reenter through the callback?
- There's a deadline check on swapExactTokens but not on addLiquidity — can a stale addLiquidity transaction be sandwiched?
- onlyAdmin is on setFee() but not on setFeeRecipient() — can anyone change where fees are sent?
```

### Step 5: Fund Flows & Critical Paths

Trace how money moves. Every transfer, approval, mint, and burn is a potential bug site.

**What to look for:**
- Where do tokens enter and leave the protocol?
- Are there intermediate holding contracts?
- What happens to stuck tokens?
- Can the flow be interrupted mid-way leaving funds stranded?

**Example primer entries:**
```
- Deposits go: User → Router → Vault → Strategy. What happens if the Strategy is paused after the Router forwards funds but before the Vault deposits?
- The bridge locks tokens on Chain A and mints on Chain B — what if the mint fails? Is there a recovery mechanism? Can an attacker trigger repeated mints by replaying the message?
- Fees accumulate in the contract but there's no sweep function — are they stuck forever?
- The flash loan callback transfers tokens back, but what if the callback reverts? Does the protocol handle partial repayment or is it all-or-nothing?
```

### Step 6: Trust Your Instincts

Not everything fits a neat category. If something feels off — confusing logic, overly complex code, comments that contradict the implementation, or code that "shouldn't be necessary" — write it down.

**Example primer entries:**
```
- The _updateRewards() function is 80 lines with nested conditionals — this is too complex for what it does. Trace every branch and verify the reward math is correct in all cases.
- There's a comment saying "temporary fix, TODO remove" on line 234 — what was the original bug and does the fix introduce new issues?
- The same calculation appears in 3 functions but with slightly different rounding — is this intentional or a copy-paste error?
- Why does claimRewards() call _checkpoint() twice? Is the second call redundant or does it serve a purpose?
```

### Step 7: AI-Assisted Expansion (Final Step)

After your manual primer is done, ask the AI to identify additional sensitive areas you might have missed.

**Prompt to use:**
```
Based on the contract code I've shared, give me a list of:
1. All areas where fund flow happens (deposits, withdrawals, transfers, mints, burns) and where critical bugs could arise.
2. State-changing functions that interact with external contracts.
3. Areas where precision/rounding could cause accounting errors.
4. Any code that handles multiple token types or variable decimals.

Be specific — give function names and line numbers.
```

Review the output, filter out noise, and add the genuinely useful entries to your primer.

## The Primer Prompt Template

Once your primer is built, wrap it in this template and feed it to the AI alongside the contract code.

```
You are a senior smart contract security researcher. I've manually reviewed this codebase and built a custom primer — a list of observations, questions, and areas I believe are sensitive or potentially vulnerable.

## Your Task

For each primer entry below:
1. Investigate the specific concern thoroughly in the contract code.
2. Trace the relevant code paths end to end.
3. Determine if the concern is a valid bug, a false alarm, or needs more context.
4. If it's a valid bug, classify severity (Critical / High / Medium / Low) and explain the exploit scenario.
5. If it's a false alarm, explain exactly why — which checks or constraints prevent the issue.

Do NOT skip any entry. Do NOT add generic findings that aren't related to my primer. Stay focused on what I've flagged.

## My Custom Primer

[Paste your primer entries here]

## Contract Code

[Paste contract code here]

## After Completing the Primer Review

Once you've addressed every primer entry, list any closely related issues you discovered while investigating my entries — but ONLY if they are directly connected to something in my primer. Do not run a generic audit.
```

## Combining with Other Prompts

The custom primer works best **alongside** the other prompts in this repo, not as a replacement. Recommended workflow:

1. **Recon first** — Use `bug-bounty/recon-checklist.md` or read the docs/whitepaper to understand the protocol.
2. **Build your primer** — Read the code manually, noting observations using the steps above.
3. **Run the primer prompt** — Feed the template above with your primer entries and the contract code.
4. **Run the standard audit/hunting prompt** — Use `private-audits/audit-guide.md` or `bug-bounty/hunting-guide.md` with `common/review-checklist.md` for the systematic checklist-based review.
5. **Cross-reference** — Compare findings from your primer run and the generic checklist run. The overlap is high-confidence. The unique findings from each run are where the real value is.

## Tips

- **Don't over-filter your primer.** If something caught your eye, write it down. A "probably nothing" hunch has caught more bugs than you'd expect.
- **Be specific.** "Check access control" is useless. "onlyAdmin is missing on setFeeRecipient() — can anyone redirect fees?" is actionable.
- **Update as you go.** Your primer is a living document during the audit. Add entries as you discover new things.
- **Keep a primer library.** After each audit, save your primer. Patterns repeat across protocols — your old primers become templates for new ones.
- **One primer per contract (or per module).** Don't dump 50 entries covering 10 contracts into one prompt. Scope it tightly for better results.
- **Version your primers.** If you're auditing across multiple rounds or the code changes, keep a v1/v2 so you know what's been addressed.
