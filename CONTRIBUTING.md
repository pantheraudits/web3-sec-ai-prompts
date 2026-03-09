# Contributing

Thanks for your interest in contributing to web3-sec-ai-prompts!

## How to Contribute

1. **Fork** this repository.
2. **Create a branch** for your changes.
3. **Add or improve** a prompt file in the appropriate directory.
4. **Submit a pull request** with a clear description of what your prompt does.

## Prompt Guidelines

- Each prompt file should be a `.md` file with a clear title and description.
- Include a **Purpose** section explaining when to use the prompt.
- Include the **Prompt** itself in a code block or clearly marked section.
- Add **Usage Tips** or **Customization Notes** where helpful.
- Keep prompts focused — one workflow per file.

## Quality Standards

- Prompts should be tested with at least one AI model before submission.
- Avoid prompts that produce generic or unhelpful output.
- Include real-world context (e.g., DeFi, NFT, bridge) where applicable.
- Do not include any sensitive or proprietary audit findings.

## Directory Structure

- `bug-bounty/` — Bug bounty hunting workflows
- `private-audits/` — Private audit engagement workflows
- `contests/` — Audit contest strategies
- `zk-audits/` — ZK circuit audit guides
- `common/` — Shared patterns, attack vectors, severity assessment, language-specific vulnerability patterns
- `claude-skill/` — Panther Audit automated pipeline (SKILL.md + chunk-pipeline.md)
- `safe-solana-builder/` — Safe Solana Builder skill for generating secure Solana programs (SKILL.md + references/)

## Adding a New Skill

If you're adding a new skill (like `safe-solana-builder/`):

1. Create a directory at the repo root: `your-skill-name/`
2. Add `SKILL.md` with YAML frontmatter (`name:` and `description:`) and the full workflow
3. Put reference files in `your-skill-name/references/` if the skill needs to load knowledge files
4. Add a `README.md` with installation instructions for Cursor, Claude Code, and Claude.ai
5. Update the root `README.md` structure table to include your new skill
