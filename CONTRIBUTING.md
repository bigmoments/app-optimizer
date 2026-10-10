# Contributing to app-optimizer

Thanks for considering a contribution! This is a small, focused skill — a short, accurate addition beats a long, stale one.

## What makes a good contribution

- **New anti-patterns** you've seen in the wild (with the fix, not just the complaint).
- **Verification methods** — concrete ways to prove an improvement with numbers.
- **Corrections** — outdated advice, wrong defaults, broken assumptions.
- **Translations** — the skill content is in Bahasa Indonesia; an English `references/` would be a welcome parallel effort (open an issue first so work doesn't duplicate).

## Ground rules

1. **Portable over stack-specific.** Examples may name a stack, but the rule must survive any stack.
2. **Measure, don't guess.** Any new optimization advice must include how to verify it.
3. **Free-tier friendly.** Don't recommend a paid service when a free pattern solves it.
4. **Keep modules focused.** If it doesn't fit an existing module's topic, say so in the PR — don't shoehorn it.
5. **No AI slop.** Write like an engineer, not a brochure.

## How to contribute

1. Fork the repo and create a branch (`fix/...`, `feat/...`).
2. Edit the relevant file under `references/` (or `SKILL.md` for workflow changes).
3. Keep the existing structure per module: guidance → anti-patterns → verification.
4. Open a pull request describing **what**, **why**, and **how you verified it**.

## Reporting issues

Open an issue with: what you expected, what happened, and the smallest reproduction (app type, scale, numbers if any).
