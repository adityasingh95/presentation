# FX Sales Workstation — Automation Test Report

**Run date:** 2026-05-28
**Branch:** `main` (post-merge of PR #13 — cleanup)
**Result:** ✅ **All 322 tests passed**

---

## Executive summary

| Suite           | Tests   | Passed | Failed | Skipped | Duration |
|-----------------|---------|--------|--------|---------|----------|
| **Unit + component** (Vitest) | 316 | 316 | 0 | 0 | 1.8s |
| **End-to-end** (Playwright)   | 6   | 6   | 0 | 0 | 35.5s |
| **Total**                     | **322** | **322** | **0** | **0** | **~37s** |

Zero flakes. Zero skipped. Zero failures.

---

## What's covered

The test pyramid is intentionally bottom-heavy: hundreds of fast, deterministic unit and component tests at the base, with a small number of full browser-driven E2E scenarios at the top that exercise the canonical user journeys.

### Layer 1 — Unit + component tests (316 tests, 38 files)

Validate the building blocks in isolation:

- **State machines** — RFS Trade Model + Sales Intervention Trade Model state graphs, every transition, every terminal state.
- **Pure logic** — pip math, margin calculations, formatters, time helpers, AI Margin Suggestion engine.
- **State stores** — Zustand stores for deals, UI, notifications, settings.
- **Feed simulation** — pricing tick generation, deal feed scenarios.
- **React components** — every panel in the ticket (Reasons, Summary, Pricing, Suggestion), blotter rows, status pills, toasts, mute toggle.
- **App shell** — renders without errors, dev-injector visibility, **no leakage of vendor names anywhere in the rendered output**.

Runtime: ~2 seconds. Run on every commit.

### Layer 2 — End-to-end scenarios (6 tests)

Real browser (Chromium), real production build, simulated pricing feed pinned to a deterministic seed. Each scenario maps 1:1 to a documented business flow:

| Scenario                  | What it proves                                                                 | Duration |
|---------------------------|--------------------------------------------------------------------------------|----------|
| **HAPPY_PATH_ESP**        | Electronic deal flows untouched: `inject → AUTO → DONE → Historic blotter`     | 8.0s |
| **OFF_HOURS_INTERVENTION**| Full trader-driven Sales Intervention flow from pickup to quote to client-accept | 8.1s |
| **CREDIT_BREACH**         | Credit-decline AI suggestion fires, trader rejects, deal lands in Historic correctly | 7.5s |
| **SIZE_LIMIT_MARGIN_TUNE**| AI Margin Suggestion is applied, stream is sent, client accepts                | 9.2s |
| **RELEASE_PATH**          | Trader picks up an intervention, releases it back to the desk, row stays Active | 0.7s |
| **smoke**                 | App loads, body renders                                                        | 0.1s |

These scenarios collectively exercise:

- Both lifecycle state machines (RFS Trade Model + Sales Intervention Trade Model) running in parallel
- The simulated pricing feed
- The AI Margin Suggestion engine across happy-path and rejection paths
- The 5-second auto-removal of completed deals from the Active blotter
- The Active → Historic blotter promotion

Each E2E scenario asserts on stable `data-*` attributes (`data-deal-id`, `data-si-state`, `data-display-status`, `data-outcome`) rather than text or color — meaning the tests don't break if we re-skin the UI.

---

## How to read the HTML report

Open `e2e-report.html` in any browser. It's a self-contained file (no server required, no internet required).

- The **landing view** shows all 6 scenarios with pass/fail status and timing.
- Click any scenario to see the **step-by-step trace** of what the test did.
- On failure, the report would include screenshots, DOM snapshots, network logs, and console output for every step — making post-mortem trivial. (No failures in this run, so those panels are empty.)

---

## Continuous integration

These same tests run on every push to `main` and every pull request via GitHub Actions (`.github/workflows/ci.yml`). The pipeline is:

1. `pnpm typecheck` — TypeScript strict mode, no `any`
2. `pnpm lint` — ESLint with zero-warnings policy
3. `pnpm test:run` — all 316 Vitest tests
4. `pnpm test:e2e` — all 6 Playwright scenarios

A pull request cannot be merged unless every step is green. On failure, Playwright traces are retained as build artifacts for 7 days.

---

## What's intentionally not tested at this layer

- **Real network calls** — the prototype uses a simulated pricing feed. No external APIs are dialed.
- **Persistence beyond the session** — there is no backend; state lives in memory.
- **Audio playback** — gated behind a user-gesture unlock per browser autoplay policy; smoke-validated manually.

These are product decisions documented in `CLAUDE.md` and `docs/`, not test-coverage gaps.
