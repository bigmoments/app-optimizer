# Audit report template (Path B)

Copy this shape when reporting a brownfield audit. Every claim needs a number.

## Target

- App: `<name / repo / URL>`
- Date: `<YYYY-MM-DD>`
- Scope: `<what was audited>`

## Baseline (before)

| Metric | Value | How measured |
|---|---|---|
| LCP | | Lighthouse / PageSpeed |
| INP | | |
| CLS | | |
| API p95 (hottest endpoint) | | |
| Slowest DB query | | slow query log / pg_stat_statements |
| JS bundle (gzip) | | bundle analyzer |

## Findings (ordered by 80/20 priority)

### 1. `<finding>` — `<module>`
- **Evidence:** `<numbers, EXPLAIN output, log excerpt>`
- **Fix:** `<what was changed>`
- **After:** `<numbers>` — delta: `<x → y>`

*(repeat per finding)*

## Not changed (and why)

- `<considered but skipped, with reason — e.g. "rewrite auth: data shows it's not the bottleneck">`

## Regression guards installed

- `<what prevents this from regressing: perf budget in CI, alert, checklist item>`

## Verdict

`<one paragraph: what improved, by how much, what remains>`
