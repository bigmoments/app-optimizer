# Changelog

All notable changes to this skill are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Added
- `references/ux.md` §3 "States yang wajib ada": anatomi konkret halaman error (bahasa manusiawi, aksi pemulihan, bedakan 404/500/error boundary, ID error untuk support) dan state offline ala YouTube ("Anda sedang offline": tampil jelas, nonaktifkan aksi network + penjelasan, coba lagi/muat ulang otomatis). Tetap YAGNI — state UX, bukan arsitektur offline-first.

### Fixed
- `references/ux.md` §3: paradoks offline state — panduan kini dibedakan untuk aplikasi online-first (offline = degraded, nonaktifkan aksi network) vs offline-first (offline = mode operasi normal; tampilkan status sync, bukan larangan pakai).

## [1.0.0] - 2026-10-10

### Added
- Initial public release.
- `SKILL.md`: two-path workflow (Path A greenfield planning, Path B brownfield audit), five core principles, 80/20 priority order, chat triggers, auto-trigger conditions.
- Nine knowledge modules under `references/`: performance, database, caching, rate-limiting, ux, security, observability, debugging, checklists. Each module covers "when planning" vs "when optimizing", anti-patterns, and verification steps.
- `examples/audit-report-template.md`: report shape for Path B audits.
- Professional repo files: README, LICENSE (MIT), CONTRIBUTING, CHANGELOG, .gitignore.

### Notes
- Skill content is written in Bahasa Indonesia; README/CONTRIBUTING in English.
- Pure Markdown, no hooks/plugins/MCP/scripts. Compatible with Claude Code, Cursor, Codex, Antigravity, Gemini CLI.
