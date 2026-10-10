# app-optimizer

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Format: Markdown](https://img.shields.io/badge/format-markdown-blue.svg)](#)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

An agent skill that makes apps **light, fast, secure, and pleasant to use** — without depending on paid services. Free-tier friendly by design.

Works two ways: **triggered from chat** ("optimize this app", "performance audit") or **automatically by an agent** when it plans a new app, audits an existing one, prepares a release, adds an API endpoint, or hears a performance complaint.

> **Note:** the skill content itself is written in **Bahasa Indonesia** (its trigger phrases and working language). This README is in English for the GitHub audience.

---

## Why this skill exists

Most performance work fails for boring reasons: no baseline numbers, guessing instead of measuring, caching added to hide wasteful code, security postponed "until later". This skill encodes a disciplined workflow so an agent does the cheap, high-impact things first — and can prove every improvement with numbers.

**Core principles:**

1. **Measure first.** Without a baseline, optimization is guessing.
2. **Portable patterns, not stack-specific tricks.** Examples may name a stack; the rules must survive any stack.
3. **Cheapest first.** Start with what's cheap and high-impact (see the 80/20 list).
4. **Never trade correctness.** Stale caches, lost data, or weakened security is not "optimization".
5. **Verify every change.** Each fix must be provable with numbers, not feelings.

---

## The 80/20 priority order

When time is limited, work top-down and stop when the target is hit:

1. **Measure the baseline** — Web Vitals (LCP, INP, CLS), API p95, slow DB queries, JS bundle size.
2. **Database** — missing indexes + N+1 queries. The #1 culprit in data-driven apps.
3. **Caching** — for data computed repeatedly and rarely changed.
4. **Frontend** — images (WebP, responsive) + JS bundle size.
5. **Rate limiting + security headers** — cheap, install once, protects from abuse.
6. **UX states** — clear loading/empty/error states (shapes *perceived* speed).
7. **Observability** — logging + error tracking + free-tier limit monitoring.
8. **Micro-optimizations** — only when the data says so. Never first.

---

## Two paths

**Path A — Planning (greenfield).** New app. Bake optimization into the design, don't bolt it on later: set budgets → design schema + indexes day one → plan caching layers → design rate limits → UX system → auth → observability.

**Path B — Optimizing existing (brownfield).** Running app. Audit first, measure the baseline, fix incrementally without breaking what works. *Measure, don't guess; fix, don't rewrite.*

---

## Modules

| Module | When the agent reads it |
|---|---|
| `references/performance.md` | Always — budgets, lean backend & frontend |
| `references/database.md` | Has a database — indexing, EXPLAIN, N+1, pooling, migrations |
| `references/caching.md` | Data computed repeatedly / read far more than written |
| `references/rate-limiting.md` | Public API or abusable endpoints |
| `references/ux.md` | Has a UI — visual hierarchy, states, mobile, accessibility |
| `references/security.md` | Always before release — essential OWASP |
| `references/observability.md` | App running in production |
| `references/debugging.md` | Performance problems or optimization-related bugs |
| `references/checklists.md` | Audit procedures: pre-build (Path A) and existing-app audit (Path B) |

Each module covers "when planning" vs "when optimizing", common anti-patterns, and how to verify. The agent reads only the relevant modules — never everything at once.

---

## Installation

Pure Markdown, standard `SKILL.md` format. No hooks, plugins, MCP, or scripts. Copy the `app-optimizer/` folder into your harness's skills directory:

| Harness | Skills directory |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Cursor | `<project>/.cursor/skills/` or `~/.cursor/skills/` |
| Codex | `~/.codex/skills/` |
| Antigravity / Gemini CLI | skills directory per each tool's docs |

```bash
# example: Claude Code
cp -r app-optimizer ~/.claude/skills/
```

## Usage

Just talk to your agent:

- *"Optimasi aplikasi di `~/my-app`"*
- *"Audit performa sebelum rilis"*
- *"Bikin aplikasi ini lebih ringan"*
- *"Aplikasi lemot pas banyak user, cek dong"*
- *"app-optimizer"*

Or let the agent trigger it automatically: it picks **Path A** when you're planning/building, **Path B** when you're auditing/fixing, runs the pre-release checklist before deploy, and applies rate-limiting + security guidance when you add an API endpoint.

See [`examples/audit-report-template.md`](examples/audit-report-template.md) for the shape of a Path B audit report.

---

## Project structure

```
app-optimizer/
├── SKILL.md                  # Skill definition: triggers, principles, 80/20 order
├── README.md                 # This file
├── LICENSE                   # MIT
├── CHANGELOG.md
├── references/               # The nine knowledge modules
│   ├── performance.md
│   ├── database.md
│   ├── caching.md
│   ├── rate-limiting.md
│   ├── ux.md
│   ├── security.md
│   ├── observability.md
│   ├── debugging.md
│   └── checklists.md
└── examples/
    └── audit-report-template.md
```

---

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
