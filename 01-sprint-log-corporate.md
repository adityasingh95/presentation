# FX Sales Workstation — Sprint Log

## Project metadata

| Field | Value |
|---|---|
| Project | FX Sales Workstation |
| Repository | adityasingh95/fx-sales-intervention |
| Duration | 5 working days |
| Team size | 4 (1 PM, 1 Dev, 1 SDET, 1 BA) + Product (overseer) |
| Methodology | Agentic SDLC under spec-pack guardrails · TDD discipline labelled per ticket · Bors-style merge queue · phase-gate documentation cadence |
| Backlog | 34 tickets across 5 phases |
| Final state | 33 tickets delivered · 1 deferred (demo recording, manual artifact) · 1 open lint escalation |

## Team

| Role | Responsibilities |
|---|---|
| **Product** | Scope decisions, tiebreakers on contested calls, live-URL validation, mid-flight amendments. |
| **PM** | Sprint planning, ticket coordination, escalation routing, phase-summary authorship. |
| **Dev** | Implementation against spec, test-slate execution, in-flight technical debate. |
| **SDET** | Test slate authorship (🔴 strict tickets), gate enforcement, drift detection, coverage walks. |
| **BA** | Spec ingestion, wiki authorship, ADR drafting, per-phase lint sweeps, drift escalations. |

## Methodology — guardrails over autonomy

The build operated under a **spec-pack-first** model. Before any code, the team received an 11-document specification pack (~1,500 lines) covering glossary, PRD, functional spec, trade-state model, dummy-feed spec, UI/UX spec, tech architecture, scenario pack, test plan, and AI suggestion engine. Each backlog ticket cited the relevant spec sections; tickets without spec coverage were not opened.

TDD intensity was labelled per ticket on a three-tier scale: **🔴 Strict** (test slate written first, run red, implemented to green — used for pure logic, state machines, the AI suggestion engine), **🟡 Alongside** (component API and behaviour tests written together with implementation), **🟢 Acceptance** (the Gherkin scenario IS the test — written when a user-facing flow lands end-to-end). 23 of 34 tickets were 🔴 or 🟡.

The Bors-style merge queue (one merge at a time, verification gates before main lands) ran on every ticket completion. Direct pushes to main were not permitted. Phase-gate documentation cadence: the BA ingested the PM's phase-summary brief at each phase close, ran an eleven-category lint sweep against the live code, and surfaced any drift as a tracked escalation.

Critical operating constraints baked in from day zero:
- **TypeScript strict** — no `any`, no `// @ts-ignore`. Type escapes were treated as ticket failures.
- **No mutation** — Zustand updates immutable. Lifecycle state through XState only.
- **No new dependencies after Phase 1** — pinned-stack discipline. Five external libraries used across the entire build (react, xstate, zustand, clsx, lucide-react).
- **Brand-neutral build** — vendor names forbidden in any shipped artifact. BA-owned wiki layer enforced a stricter rule (no vendor names anywhere in `/wiki` or `/raw`, including citation URLs).
- **Write boundaries** — Dev/SDET own `/src` and `/tests`; PM owns `/docs`; BA owns `/wiki` and `/raw`. Cross-boundary edits routed through escalation.

---

# Day 0 — Hand-off and sprint planning

Product handed off the finalised specification pack. The PM completed an end-to-end read of all 11 documents and proposed a five-sprint plan mapping one-to-one to the spec's phase structure:

| Sprint | Scope | Ticket range | Effort estimate |
|---|---|---|---|
| 1 | Scaffold + first slice | FXSW-001 to FXSW-006 | 1 day |
| 2 | Feed + state coordination | FXSW-007 to FXSW-013 | 1 day |
| 3 | Ticket panels + actions | FXSW-014 to FXSW-021 | 1 day |
| 4 | AI Margin Suggestion | FXSW-022 to FXSW-027 | 1 day |
| 5 | Notifications + polish + ship | FXSW-028 to FXSW-033 | 1 day |

### Decision · Pull FXSW-034 (deploy workflow) forward into Sprint 1

The PM recommended pulling the GitHub Pages deploy ticket (originally final-sprint scope) into Sprint 1. Rationale: a live URL from day one would turn the repository itself into a demoable artifact for stakeholder review and would surface hosting-layer issues early when there was budget to absorb them. Product approved.

### Decision · Parallel BA track for wiki bootstrap

The PM proposed running a parallel work-track for the BA — initial 13-source spec ingest plus per-phase ingest + lint at each sprint close. The BA's write boundary excluded `/src`, `/tests`, and `/docs`, so the two tracks did not contend. Approved.

### Risks identified at planning time

The PM flagged three pre-build risks worth tracking:

| ID | Risk | Likely sprint | Mitigation |
|---|---|---|---|
| R1 | AG-Grid 31 may not support per-row `data-*` attributes; test contract depends on them | 2 | Plain flex-row fallback documented as Plan B |
| R2 | XState v5 strict typing may reject DRY handler factories | 2 | Accept verbose inline form |
| R3 | GitHub Pages environment protection may block deploys from non-main branches | 1 | Merge to main and trigger from there |

All three risks materialised. All three were resolved within the sprint that surfaced them.

---

# Sprint 1 — Scaffold + first slice

## Sprint goal

Stand up the Vite + React + TypeScript scaffold with strict configuration, design tokens, state-machine skeletons, an application shell rendering a hardcoded blotter row, and a deployable live URL. Validate the toolchain (Vitest, Playwright, ESLint, Prettier, CI workflow placeholders).

## Tickets

| Ticket | Title | TDD | Status |
|---|---|---|---|
| FXSW-001 | Project scaffolding | 🟡 | ☑ (E2E gate via CI in Sprint 5) |
| FXSW-002 | Design tokens + Tailwind config | 🟡 | ☑ |
| FXSW-003 | Folder structure | — | ☑ |
| FXSW-004 | prebuild reference-mids script | 🔴 | ☑ |
| FXSW-005 | State machine skeletons | 🔴 | ☑ |
| FXSW-006 | AppShell + Header + empty blotter | 🟡 | ☑ |
| FXSW-034 | GitHub Pages deploy workflow (pulled forward) | 🟡 | ☑ |

## Sprint highlights

**FXSW-001 — toolchain validation.** Dev initialised Vite + TypeScript strict + React 18 with all dependencies pinned per the architecture document. ESLint, Prettier, Vitest with jsdom environment, Playwright with Chromium-only single-worker configuration, all green except the Playwright browser binary download (the local sandbox blocked the CDN). SDET escalated the Playwright issue to PM, who routed to Product with a recommendation to defer the E2E gate to CI (FXSW-032). Ticket merged with status ◐, flipped fully ☑ once CI confirmed green in Sprint 5.

**FXSW-002 — design tokens.** Dev declared all 43 CSS custom properties from the UI/UX spec, mirrored as Tailwind theme extensions. Geist and Geist Mono loaded via the dedicated font packages.

**FXSW-003 — folder structure.** Structural ticket only. All 23 directories and placeholder files (`export {}` for TS files) created to keep the build green. No tests required.

**FXSW-004 — reference-mids prebuild.** Dev wrote the test slate first (three cases), all red. Implementation: `scripts/fetch-reference-mids.ts` with network-failure fallback to hard-coded May 2026 anchor values. Build never breaks on network failure (exit code 0 either way). Tests green.

### Decision · FXSW-005 — `timings.ackDelayMs` as mutable property

Dev's first implementation exported `ackDelayMs` as a `const`. The SDET-authored test slate required reassignment from test files to zero the delay; the reassignment had no effect.

Root cause: ES module `const` bindings are read-only across module boundaries — the reassignment was silently rejected at the consumer side.

The SDET diagnosed the issue and proposed two fixes:
1. Export an object: `export const timings = { ackDelayMs: 250 }` and reassign the property.
2. Add an explicit setter function.

The Dev applied option 1 (more symmetric with how tests read). A related XState quirk surfaced in the same diagnosis: `after: { 250: ... }` bakes the delay at machine-creation time, so the named-delay function form (`delays: { ackDelay: () => timings.ackDelayMs }`) was required for the override to take effect on subsequent transitions.

The BA was notified to draft ADR-0009 (Simulated 250ms ack delays) at sprint close, with the mutability rationale documented in the Implementation block.

**Outcome:** Test slate green. Pattern formalised in `wiki/components/test-patterns.md §2` so future ticket holders don't re-encounter the same issue.

**FXSW-006 — application shell.** Dev rendered the full-window dark workstation layout — header (56px), Active blotter (55%), Historic blotter (45%), 2px linear gradient top strip in the AI-accent indigo. A hardcoded blotter row exercised every design token. Product completed the eye-test and approved.

### Decision · FXSW-034 — environment protection surfaced

Dev wired the GitHub Pages deploy workflow per spec. Push from the feature branch ran the build job successfully (artifact uploaded), but the deploy job was silently skipped. No diagnostic appeared in the worker logs because the build agent had no API access to inspect environment-level configuration.

Dev escalated as HIGH severity. PM routed to Product, who read the failure annotation directly from the Actions UI: "Deployment branches rule prevents deployment from branch `claude/<feature>`. Allowed branches: main."

Product elected to **keep the protection rule** (it was doing the right thing) and resolve by merging to main, where the workflow triggered naturally. Dev followed up with a one-commit patch to the tech-architecture document so future deployments don't repeat the diagnostic detour.

> **Cross-reference:** Build Log Convoy 1 · 13:04 fxsw-034 entry.

**Outcome:** Live URL operational. Deploy workflow restricted to main, documented in `docs/06-tech-architecture.md §7.1`.

## Sprint 1 retrospective

- All 7 tickets merged. Two escalations: one MEDIUM (Playwright sandbox, accepted), one HIGH (Pages env protection, resolved).
- The pull-forward of FXSW-034 paid for itself within the sprint — getting the env protection issue out of the way before Sprint 5's polish work avoided a same-day demo blocker.
- BA completed the parallel wiki bootstrap: 13-source ingest, 10 ADRs backfilled, 14 wiki pages drafted. Status pages flagged `in-progress` pending later phase work.
- PM authored the first phase summary at `docs/phase-summaries/FXSW-006-summary.md`. BA ingested and showed drafts before any wiki write per the standing review gate.

**Sprint 1 metrics:** 7 tickets delivered · 35 unit tests · CI gates green · live URL operational.

---

# Sprint 2 — Feed + state coordination

## Sprint goal

Make scenarios injectable end-to-end. PricingFeed emitting deterministic ticks. DealFeed driving scripted scenarios. dealsStore spawning XState machines per deal. dealMachine coordinating the two child machines (RFS + SI) per the documented cross-model relationship table. Active and Historic blotters wired with the 5-second post-terminal removal rule. First end-to-end E2E test passing (Happy Path ESP scenario).

## Tickets

| Ticket | Title | TDD | Status |
|---|---|---|---|
| FXSW-007 | PricingFeed with seeded RNG | 🔴 | ☑ |
| FXSW-008 | DealFeed + scenario player | 🔴 | ☑ |
| FXSW-009 | dealsStore + machine spawning | 🔴 | ☑ |
| FXSW-010 | dealMachine cross-model coordination | 🔴 | ☑ |
| FXSW-011 | statusFromMachines derivation | 🔴 | ☑ |
| FXSW-012 | Active + Historic blotters | 🟡 | ☑ |
| FXSW-013 | DevInjector + HAPPY_PATH_ESP E2E | 🟢 | ☑ |
| FXSW-resp | Mid-flight responsive layout amendment | 🟡 | ☑ |

## Sprint highlights

**FXSW-007 — PricingFeed.** Dev wrote the test slate first (six cases). Implementation used a seeded Mulberry32 PRNG with Box-Muller normal sampling, 300ms tick interval, 10% mean-reversion to the bootstrapped reference mids from FXSW-004. The `stop()` method clears subscriptions, the latest-tick cache, mids, references, and the interval — the harshest valid interpretation of the AC test "no further callbacks after stop()." Golden seed-42 EURUSD mid sequence locked at `[1.1715, 1.1714, 1.1714, 1.1714, 1.1714]` as a regression guard against future PRNG changes.

### Decision · FXSW-008 — `notifyDealState` extends DealFeed interface beyond spec

Dev implemented the scenario player and discovered a gap: the dummy-feed spec defined three DealFeed methods (`subscribe`, `inject`, `reset`), but the SI scenarios required *state-gated* follow-ups — `CLIENT_ACCEPT` should only fire after a `SEND_STREAM` acknowledgement, not on a fixed timer.

Dev proposed adding `notifyDealState(dealId, siState)` as a fourth method, called by the dealsStore SI subscriber on every state transition. The player would listen for the gating state and fire the queued event.

Dev surfaced as agent-directed deviation. PM reviewed the alternative (out-of-band bridge module imported by both sides) and judged it tighter coupling for the same effect. **Decision: ship the extension, document inline at the interface definition, flag in the phase summary as an agent-directed deviation, no spec amendment.**

> **Cross-reference:** Build Log Convoy 2 · 09:01 fxsw-008 entry.

**Outcome:** One extra interface method with rationale in source. Listed as an explicit agent-directed deviation in the FXSW-013 phase summary.

**FXSW-009 — dealsStore.** Zustand store holding `Map<dealId, DealEntry>`. Each `addDeal` spawned a parent dealMachine actor; the store subscribed to both spawned children (RFS and SI) and mirrored their state names into the entry for React selectors. The dealsBootstrap function wired DealFeed events to `addDeal` once at app boot.

### Decision · FXSW-010 — XState v5 generic narrowing rejected DRY factory

Dev's first draft of the parent `dealMachine` factored a helper to compress the sixteen-row cross-model relationship table:

```typescript
function toSi(type) { return sendTo(({context}) => context.si, { type }); }
function toRfs(type) { return sendTo(({context}) => context.rfs, { type }); }
on: {
  PickUp: [toSi('PickUp'), toRfs('PickUp')],
  Quote:  [toSi('Quote'),  toRfs('PriceUpdate')],
  // 14 more rows
}
```

TypeScript compilation failed with 31 errors. Root cause: `setup({...}).createMachine` narrows the action signature so tightly that the inline `sendTo` form is the only one satisfying the strong generics. Helper-returned actions fail type narrowing.

The team considered three options:
1. Verbose-but-inline (16 explicit `sendTo` calls).
2. Cast to `never` (violates `no any` rule).
3. Refactor `setup` to weaker types (loses fxsw-010 type safety).

**Decision: option 1.** Type-strict codebase wins; the verbose form ships. BA was notified to extend ADR-0003 (XState + Zustand) with a consequences entry documenting the quirk.

> **Cross-reference:** Build Log Convoy 2 · 11:02 fxsw-010 entry.

**Outcome:** All 10 unit tests covering the cross-model coordination table passed. Trade-off documented in ADR-0003 Consequences.

**FXSW-011 — status derivation.** Pure-function ticket with 100% branch coverage AC. SDET-led test slate covered every row in the trade-state-model status-derivation table (13 cases via `it.each`).

### Decision · FXSW-012 — AG-Grid superseded for plain flex-row table

The Dev wiring the Active and Historic blotters discovered that AG-Grid Community 31 had no first-class API for per-row `data-*` attributes. The entire Playwright test contract was built on `data-deal-id`, `data-rfs-state`, `data-si-state`, `data-display-status`, `data-dealable`, and `data-removing` on each row.

Two workarounds were available:
- Custom row template (heavy, brittle, fights the library on every styling change).
- Post-render DOM mutation (fragile, hostile to React's reconciler).

Dev escalated as HIGH severity. The team reviewed what the v1 blotter actually needed from a grid library: no virtualization (deal counts stay small), no column resize or pinning or grouping, no inline editing, no internal sort. The trader-blotter aesthetic came from the design tokens and spacing, not from the grid library.

**Decision: ship a plain flex-row table built from `<button>` rows. ADR-0004 marked superseded (not rewritten — the original decision was sound pre-build; the supersession captures the new information from implementation). AG-Grid kept in `package.json` for now in case a v2 feature genuinely needs virtualization.**

> **Cross-reference:** Build Log Convoy 2 · 14:22 fxsw-012 entry. ADR-0004 status.

**Outcome:** Each row gets `data-*` attributes as JSX props. Component size dropped from ~250 lines (grid config plumbing) to ~80 lines. Bundle ~200KB lighter.

### Decision · FXSW-013 — ESP terminal coordination via mirrored Removed state

The Happy Path ESP E2E ran red on first attempt with a previously-unsurfaced invariant break. The dealsStore subscriber watched SI machines reaching the hidden `Removed` final state; ESP deals leave SI at `Initial` forever (no pickup, no quote), so `siMachine.after: { removalDelay: 'Removed' }` never fired. The row sat in Active indefinitely.

The team weighed two fixes:
1. Synthetic SI `Initial → TradeConfirmed` transition gated on the RFS terminal. (Violates the spec — SI staying at `Initial` for ESP is documented.)
2. Mirror the `Removed` cleanup state onto `rfsMachine`. (Introduces a state the spec doesn't mention but changes no documented state's semantics.)

**Decision: option 2.** An idempotent `archive()` helper with a guard ensures the parallel SI subscriber doesn't double-archive for SI flows. A related fix to `outcomeFromFinalStates` correctly classified the ESP shape (`siState === 'Initial' && rfsState === 'TradeConfirmed'`) as `Executed` rather than falling through to `Cancelled`.

> **Cross-reference:** Build Log Convoy 2 · 15:51 fxsw-013 entry.

**Outcome:** Happy Path ESP E2E passes in 8.0 seconds. The two-machine invariant ("every deal has both machines reach a terminal") preserved.

### Decision · Mid-sprint amendment — responsive layout in scope

Late on Day 2, Product opened the live URL on mobile and reported that the header was cut off, blotter rows exceeded viewport width, and the injector buttons stacked poorly. The PRD explicitly scoped responsive layout out, but Product wanted to demo the URL on a phone.

PM proposed two paths:
1. Amend the PRD and execute a card-stacked mobile redesign (a Sprint-5-shaped piece of work).
2. Keep a single layout, add horizontal-scroll containment within both blotters, accept that mobile is "usable, not pretty."

**Decision: option 2.** A new ticket (FXSW-resp) was added to the active sprint via the sprint-amendment process. Dev delivered the change as one container wrapper per blotter (`overflow-auto` at `min-w-[1100px]` and `min-w-[920px]`), sticky-top column headers, and a horizontally-scrollable header dev-injector strip. SDET verified at three viewports (390×844, 768×1024, 1440×900) via Playwright.

Four documents required reconciliation (`docs/01-prd.md §4`, `docs/02-functional-spec.md §1`, `docs/05-ui-ux-spec.md §9`, `docs/BACKLOG.md` polish item). PM directed Dev to document the amendment in `docs/dev-log.md` as the **precedent for future in-flight spec changes** — the dev-log is the source of truth; older docs were patched to match.

> **Cross-reference:** Build Log Convoy 2 · 16:39 mid-phase amendment entry.

**Outcome:** Mobile usable via horizontal scroll. Single codepath maintained. The amendment process itself became part of the documented operating model for in-flight changes.

## Sprint 2 retrospective

- 8 tickets delivered (7 planned + 1 mid-sprint amendment). 4 escalations across sprint (1 cleared as accepted, 3 resolved).
- The ESP terminal coordination bug was the most consequential surprise: the doc-pack hadn't called it out, the failing E2E surfaced it. Lesson captured in the phase summary's "What surprised you" section and in the rfs-machine wiki page.
- AG-Grid supersession was the most consequential decision: the spec called for a library that couldn't satisfy the test contract. The ADR's `superseded` status (rather than rewrite) preserved the original decision trail.
- BA ingested the phase summary, drafted 13 new wiki pages and updated ADR-0004 to superseded, and ran the first phase lint. **LINT-002 escalation:** the BA discovered `docs/BACKLOG.md` showed 2 ☑ tickets when 14 were complete. The BA's write boundary forbade fixing `/docs/`; the escalation routed to PM, who slung a batch-fix to a fresh Dev. 11 tickets flipped to ☑ in one commit. Escalation closed within the same session.

**Sprint 2 metrics:** 8 tickets delivered · 183 unit tests pass · 2 E2E tests pass (smoke + happy-path-esp).

---

# Sprint 3 — Ticket panels + actions

## Sprint goal

Make every INTERVENE row clickable through to a working ticket. Right-side glass overlay panel with seven sub-panels (Reasons, Summary, Pricing, ClientSummary, DealSummary, Footer; AI Suggestion deferred to Sprint 4). Hold-to-confirm footer actions. End-to-end driveability of the off-hours intervention scenario.

## Tickets

| Ticket | Title | TDD | Status |
|---|---|---|---|
| FXSW-014 | TicketPanel shell + glass overlay | 🟡 | ☑ |
| FXSW-015 | ReasonsPanel | 🟡 | ☑ |
| FXSW-016 | Summary + DealSummary panels | 🟡 | ☑ |
| FXSW-017 | PricingPanel streaming mode | 🟡 | ☑ |
| FXSW-018 | PricingPanel fixed mode + margin controls | 🟡 | ☑ |
| FXSW-019 | ClientSummaryPanel + pips library | 🔴 | ☑ |
| FXSW-020 | TicketFooter + *Sent → *Ack flow | 🔴 | ☑ |
| FXSW-021 | OFF_HOURS_INTERVENTION E2E | 🟢 | ☑ |

## Sprint highlights

**FXSW-014 — TicketPanel shell.** Right-side 640px panel sliding in via `transform: translateX` over 240ms with the cubic-bezier easing curve from the UI/UX spec. Glass background with `backdrop-filter: blur(20px) saturate(140%)`. Blotters dim to 75% opacity when the ticket is open. Esc and backdrop click both close. Opening fires SI `PickUp`; closing does NOT fire `Hold` (passive close is the documented spec).

**FXSW-015 / FXSW-016 — supporting panels.** Routine implementation. ReasonsPanel renders one Chip per rejection reason. SummaryPanel renders the natural-language deal sentence; DealSummaryPanel renders the trade fields with T+2 weekday settlement-date calculation in `lib/time.ts`.

### Decision · FXSW-017 — seed-42 bid/ask rounding asymmetry

The PricingPanel test slate first ran red on the bid assertion. SDET wrote a throwaway debug spec that dumped the float-level values:

```
tick 1 mid_float:  1.171467284...
rounded mid:       1.1715
bid_float:         1.171442284  (mid - 0.000025)
rounded bid:       1.1714  ← rounds DOWN
ask_float:         1.171492284
rounded ask:       1.1715  ← rounds UP
```

The golden seed-42 sequence locked in FXSW-007 covered the mid only. The bid_float landed just below the 1.17145 rounding boundary; the ask_float landed above. The asymmetry was real, not a bug — but the test slate had assumed symmetric rounding.

SDET's diagnostic mail to Dev included the recommendation to use GBPUSD seed-42 tick 1→2 for tick-flash tests (both cells drop one pip cleanly, no rounding edge cases). Dev applied the assertion fix and added an inline comment so future cleanup passes don't "fix" the assertion back to 1.1715.

> **Cross-reference:** Build Log Convoy 3 · 11:05 fxsw-017 entry. `wiki/components/test-patterns.md §1`.

**Outcome:** Pattern formalised as `test-patterns.md §1` (seed pinning — watch for half-spread rounding asymmetry). The throwaway-debug-spec workflow was captured as `test-patterns.md §9`. Two reusable patterns from one bug.

**FXSW-018 — PricingPanel fixed mode.** Bid/Ask boxes as click targets entering fixed mode for that side. Refresh button only visible in fixed mode. Margin field with +/- buttons (and keyboard +/- when panel focused), floor at 1. The programmatic margin update animation with indigo outline glow was wired ahead of Sprint 4's Apply hook.

**FXSW-019 — pips library + ClientSummaryPanel.** Pure-logic ticket. Dev wrote the test slate first. `lib/pips.ts` provided `pipSize(pair)` returning 0.0001 for non-JPY pairs and 0.01 for JPY pairs, `addPips(rate, pair, pips)` pipSize-aware, and `applyMargin(bid, ask, pair, marginPips)` widening the spread by `2 * marginPips`. ClientSummaryPanel composes these for the read-only client preview, frozen on the captured rate when in fixed mode.

### Decision · FXSW-020 — HoldButton inline now, lift later

The TicketFooter required two hold-to-confirm buttons (Reject, Send Stream) with 600ms hold or double-click semantics. Dev asked PM whether to lift the `HoldButton` wrapper to `src/components/` immediately or inline it.

PM applied the standing rule: **lift when a second consumer arrives, not before.** Premature abstraction creates ghost code with one user pretending to be a primitive.

Dev inlined `HoldButton` in `TicketFooter.tsx` and added a note to the dev-log entry: "lift when a second consumer arrives — likely FXSW-026 (credit-decline Reject shortcut)."

> **Cross-reference:** Build Log Convoy 3 · 15:38 fxsw-020 entry. Promise redeemed in Sprint 5 FXSW-030.

**Outcome:** Single consumer ships inline. Lift promise carried to Sprint 5 polish ticket once the second consumer (Sprint 4 FXSW-026) materialised.

**FXSW-021 — OFF_HOURS_INTERVENTION E2E.** Direct port of the Gherkin scenario from `docs/07-scenario-pack.md` Scenario 2. Playwright init-script pins `__seedFeed = 42` and `__zeroAckDelay = true`. All assertions on `data-*` attributes; never on text or color (per the spec's test-fidelity notes). 8.2 seconds end-to-end.

## Cross-cutting · wiki/CLAUDE.md merge conflict

Mid-Sprint, the BA pulled main and discovered a merge conflict on `wiki/CLAUDE.md`. A Sprint-1 Dev had written a softer "scope" version of the file (essentially just a scope note); the BA's version was the strict ten-rule operating manual drafted from the spec.

The BA followed protocol — never silently overwrite — and escalated to Product with the two options:
1. Take the strict version, discard the soft.
2. Merge the useful soft sections into the strict structure.

Product chose option 2. The BA executed the merge manually, preserving the strict ten-rule spine and lifting the build-agent scoping context from the soft version. The resolution itself was logged as a `reconcile` entry in `wiki/log.md`.

> **Cross-reference:** Build Log Convoy 3 · 18:02 cross-cutting entry.

## Sprint 3 retrospective

- All 8 tickets merged. 1 cross-cutting escalation (merge conflict, resolved).
- The seed-42 rounding asymmetry was the most diagnostically interesting issue: 15 minutes of throwaway-debug-spec work surfaced a real float-level invariant the test slate had assumed away. Two reusable test patterns captured.
- BA Phase 3 lint focus was data-testids and component naming. Two findings resolved (`LINT-301`, `LINT-302`); one deferred to Sprint 4 (`LINT-303` — suggestion-* testids exist in Phase 4 source but not yet documented).

**Sprint 3 metrics:** 8 tickets delivered · 235 unit tests pass · 3 E2E tests pass.

---

# Sprint 4 — AI Margin Suggestion

## Sprint goal

Deliver the AI Margin Suggestion panel — the visual moment-of-delight in the ticket. Deterministic rule engine producing pip suggestions per deal context, client profile, and market state, with a one-line rationale and an Apply button. Credit-limit short-circuit returning a Reject recommendation rather than a wider price. Two end-to-end scenarios passing (`SIZE_LIMIT_MARGIN_TUNE` and `CREDIT_BREACH`).

## Tickets

| Ticket | Title | TDD | Status |
|---|---|---|---|
| FXSW-022 | clientProfiles seed data | 🔴 | ☑ |
| FXSW-023 | Suggestion engine (100% branch) | 🔴 | ☑ |
| FXSW-024 | Rationale builder | 🔴 | ☑ |
| FXSW-025 | SuggestionPanel ready/applied/Undo | 🟡 | ☑ |
| FXSW-026 | SuggestionPanel credit-decline + Recompute | 🟡 | ☑ |
| FXSW-027 | SIZE_LIMIT + CREDIT_BREACH E2E | 🟢 | ☑ |

## Sprint highlights

### Decision · FXSW-022 — Halcyon neutral prior

Dev encoded the five client profiles per the spec table. Halcyon Capital's `recent30dAcceptanceRate` was listed in the spec as "—" (no history — new client). Dev's first draft encoded the missing value as `0`.

SDET intercepted at code review before merge. The suggestion engine (FXSW-023, next ticket) included a rule: `recent30dAcceptanceRate < 0.4` deducted 0.5 pips ("softer margin for clients who've been declining"). A new client with no quotes would have been penalised on day one for an acceptance rate they didn't have.

**The semantic argument: "no data" ≠ "data showing zero."** A Beta(1,1) prior on a Bernoulli with no observations is 0.5. Dev applied the neutral prior with an inline comment so the next reader doesn't "fix" it back to 0.

> **Cross-reference:** Build Log Convoy 4 · 08:43 fxsw-022 entry.

**Outcome:** Halcyon encoded at 0.5 with rationale in both the source comment and `wiki/data-models/client-profile.md`. The kind of "no data ≠ zero" judgement that distinguishes a deterministic rule engine from a careless one.

### Decision · FXSW-023 — `@vitest/coverage-v8` skipped; manual branch walk

The 100% branch coverage AC pointed naturally toward installing the Vitest coverage instrumentation and adding a CI threshold gate. Dev raised the trade-off with PM.

Cost: a new dependency, adding to a codebase that had held a strict pinned-stack discipline since Sprint 1.

Benefit: automated drift detection — if a future ticket adds a branch and forgets a test, CI would catch it.

Counter: `engine.ts` is ~80 lines, 13 logical branches. Dev proposed 34 unit cases exhaustively covering each branch (every `if/else if/&&/||` with both true and false), with a manual coverage walk to verify. **Right point to add coverage instrumentation is when the file grows large enough that manual walks no longer scale.**

**Decision: skip the dependency.** Manual branch walk stands. Phase summary records the trade-off so the next ticket holder sees the rationale.

> **Cross-reference:** Build Log Convoy 4 · 09:51 fxsw-023 entry.

**Outcome:** 34 unit cases. Coverage walk verifies 13/13 branches × 2 cases each + 8 boundary/floor cases. No new dependency. Through the rest of the build, the zero-new-deps discipline continued to hold.

### Decision · FXSW-024 — pluralisation polish ("1 pip" not "1 pips")

The doc template ended every rationale with `— suggesting ${suggestedPips} pips.` The first test case where `suggestedPips === 1` produced `"— suggesting 1 pips."` — grammatically wrong.

Dev raised the choice: silent polish (two-line fix), or spec amendment (update `docs/09 §8` to specify pluralisation).

PM judged the doc template illustrative rather than authoritative: deviating to grammatically-correct output was a fidelity issue with the template, not a spec deviation. **Decision: silent polish with inline comment, phase summary note, no doc rewrite.**

> **Cross-reference:** Build Log Convoy 4 · 12:14 fxsw-024 entry.

**FXSW-025 — SuggestionPanel ready/applied/Undo.** Header with the Sparkles icon, 32-pixel mono pips number, rationale text, Apply and Why? buttons. Applied state collapses to a single-line strip ("Applied N pips · Undo"). The factors table in the Why? expansion shows the tier row as `baseline` (the starting point, not an adjustment). Dev refactored the inline engine call to a `useCallback` mid-ticket so FXSW-026's Recompute path could invoke the same code — one-commit refactor that turned out to be the right architecture from the start.

**FXSW-026 — credit-decline + Recompute.** Credit-decline branch swaps the Apply button for a Reject shortcut on red chrome, firing the same SI `Reject` event the TicketFooter uses (single code path). Recompute icon triggers an 800ms shimmer state with the spec's "Recomputing…" copy. Volatility-shift effect wired for v2 (static in v1 since `marketContext` returns constants per pair).

### Decision · FXSW-027 — `suggestion-pips` testid scoping bug

The `SIZE_LIMIT_MARGIN_TUNE` E2E ran red on first attempt with an unexpected error. The assertion `getByTestId('suggestion-pips').toHaveText('4')` matched `"4pips"` — no separator.

SDET diagnosed via a throwaway debug spec that dumped the live ticket panel HTML:

```jsx
<div data-testid="suggestion-pips">
  4<span>pips</span>
</div>
```

Playwright's `toHaveText` matcher reads `innerText`, which concatenates children into a single string. The testid wrapped both the number and the unit span; the combined text was `"4pips"`.

Dev's fix scoped the testid to the value-only child:

```jsx
<div className="row">
  <span data-testid="suggestion-pips">{suggestion.suggestedPips}</span>
  <span>pips</span>
</div>
```

> **Cross-reference:** Build Log Convoy 4 · 16:38 fxsw-027 entry. `wiki/components/test-patterns.md §8`.

**Outcome:** Pattern formalised as `test-patterns.md §8` (Cell-testid scoping — number + unit composition). The same lesson applies to margin display, estimated profit, and any other "N units" cell. Both E2Es green; total E2E suite running in 34.5 seconds.

## Sprint 4 retrospective

- All 6 tickets merged. Zero new dependencies added — pinned-stack discipline continued from Sprint 1.
- The neutral-prior catch at FXSW-022 was the most consequential review intervention of the sprint: had it shipped with `0`, the SIZE_LIMIT_MARGIN_TUNE E2E would still have passed (Halcyon isn't the client in that scenario), but the engine would have had a quiet semantic bug waiting for any future ticket that used Halcyon's profile.
- BA ran the Phase 4 mandatory code-drift check: pip-delta values in the engine vs the wiki suggestion-engine page. **13/13 values matched.** This was the most drift-prone area of the codebase per the schema; clean result.
- The `LINT-303` deferral from Sprint 3 was resolved as part of the Sprint 4 ingest (`suggestion-*` testids now documented in the AI panel wiki page).
- `LINT-401` (fixed): `client-bid` / `client-ask` / `estimated-profit` testids were documented in prose but not in explicit test-contract format. Replaced.

**Sprint 4 metrics:** 6 tickets delivered · 296 unit tests pass · 5 E2E tests pass · suite runtime 34.5 seconds.

---

# Sprint 5 — Notifications, polish, ship

## Sprint goal

Notification layer (toast + document-title prefix + row flash + WebAudio chime with mute toggle). Shared component primitives. RELEASE_PATH end-to-end test. CI workflow. README and demo packaging.

## Tickets

| Ticket | Title | TDD | Status |
|---|---|---|---|
| FXSW-028 | Notifications visual layer | 🟡 | ☑ |
| FXSW-029 | Audio chime + mute + settingsStore | 🔴 | ☑ |
| FXSW-030 | Visual polish + HoldButton lift | — | ☑ |
| FXSW-031 | RELEASE_PATH E2E | 🟢 | ☑ |
| FXSW-032 | CI workflow | 🟡 | ☑ |
| FXSW-033 | README + demo recording | — | ◐ (recording manual) |

## Sprint highlights

**FXSW-028 — toast + title + row flash.** Toast stack appears on new SI deals matching the dispatcher's eligibility check (Initial SI + dealable=true), auto-dismisses at 6 seconds, click opens the ticket. Document title prefixes with `● ` for 5 seconds on new SI deals. Row flash is a 300ms amber fade.

### Decision · FXSW-029 — one chime per new deal via `Set.size` growth

Dev's first naive wiring of the chime fired on every store update — toast hover, row flash transitions, anything that changed the dispatcher state.

SDET's diagnosis: the dispatcher already maintained `notifiedDealIds: Set<string>`. The dispatcher only ever *added* to the Set, never removed. Subscribing to the Set's **size** (not its contents) gave an O(1) "new deal arrived" signal — Set semantics dedupe, so a re-Release of an already-notified deal doesn't grow the size.

Dev applied the signal change. The hook now re-fires only on size growth.

> **Cross-reference:** Build Log Convoy 5 · 10:33 fxsw-029 entry.

**Outcome:** Per-deal one-shot chime. Pattern documented in `wiki/features/notifications.md` (O(1) signal). Audio unlock gated on first user gesture (Vitest spies on `AudioContext.prototype.createOscillator` to verify scheduling).

### Decision · FXSW-030 — HoldButton lifted per "second consumer arrived"

The FXSW-020 inline HoldButton in TicketFooter had been joined in Sprint 4 by an inline `RejectHoldButton` in SuggestionPanel (the credit-decline shortcut). Two consumers, identical shape — the trigger for the standing lift rule.

Dev consolidated both into a shared `src/components/Button.tsx` with a `holdToConfirm` prop. Both call sites updated, behavioural tests passed without modification, new unit test covered the prop variants.

> **Cross-reference:** Build Log Convoy 3 · 15:38 fxsw-020 (promise) → Build Log Convoy 5 · 12:09 fxsw-030 (delivery). 

**Outcome:** Single shared primitive. The "wait for the second consumer" rule survived three sprints and delivered the cleaner abstraction at exactly the right moment.

**FXSW-031 — RELEASE_PATH E2E.** Polaris Holdings, USDINR, 3M. Click row → ticket opens → click Release → SI transitions through `HoldSent` back to `Initial`, `data-dealable` returns to `true`, ticket closes, row STAYS in Active (non-terminal path). Fastest E2E in the suite at 0.7 seconds.

**FXSW-032 — CI workflow.** Typecheck, lint, Vitest unit suite, Playwright Chromium-only single-worker E2E suite. Trace artifacts uploaded on failure with 7-day retention. Total CI runtime 4 minutes 12 seconds — well under the 5-minute budget. FXSW-001's deferred E2E gate confirmed green in CI on the first run, flipping FXSW-001 from ◐ to ☑.

**FXSW-033 — README + demo.** Full rewrite for the shipped state. Live demo link with `?dev=1`, five-scenario table with what-to-look-for guidance, stack and commands summary, full docs index. Vendor-neutrality grep clean. The demo recording itself remained ◐ as a manual artifact for Product to capture post-merge.

## Sprint 5 retrospective and final-sweep lint

The BA's final-sweep lint ran all eleven categories from the wiki schema:

| Category | Result |
|---|---|
| Vendor neutrality | Clean (0 hits in content pages) |
| State machines | Clean (siMachine 11 + Removed, rfsMachine 6 + Removed — matches wiki) |
| Dependency versions | Clean (all match `package.json`) |
| Scenario data | Clean (all 5 match `definitions.ts`) |
| Client profiles | Clean (unchanged since Sprint 4) |
| **PIP-DELTA DRIFT (mandatory)** | **Clean — 13/13 values match** |
| data-testid | Clean (false-positive review confirmed) |
| Component naming | Clean (stub files inventoried but not built — wiki documents what shipped) |
| Broken links | Clean (0 unresolved targets) |
| Orphan pages | Clean (every page has ≥1 inbound link) |
| Backlog completion | **LINT-501 escalated** — see below |

### Open escalation · LINT-501

The BA's backlog-completion check surfaced one outstanding drift: `docs/BACKLOG.md` showed FXSW-034 (GitHub Pages deploy workflow) as ☐, but the workflow had shipped at commit `0762c4e` in Sprint 1 (pulled forward) and the live URL had been operational since Day 1. The BA's write boundary forbade editing `/docs/`.

The escalation was raised as MEDIUM severity to the PM for resolution in the next available session. The live URL and the deployed workflow are unaffected; only the backlog tickbox is out of sync.

> **Cross-reference:** Build Log Convoy 5 · 17:14 c5 wrap.

### wiki/onboarding.md — final synthesis

Per schema mandate, the BA rewrote `wiki/onboarding.md` from scratch at end-of-Sprint-5 — a 15-section new-engineer guide covering product framing, architecture, stack, repo layout, commands, three-agent operating model, where-to-start map, testing strategy, the five demo scenarios, cross-cutting rules, definition of done, build progression table, and a "lessons that survived the build" section. Status: stable.

## Sprint 5 metrics

- 6 tickets delivered (5 ☑ + 1 ◐ awaiting manual demo capture).
- **316 unit tests pass, 0 todo.**
- **6 E2E tests pass in 35.9 seconds** (smoke + 5 named scenarios).
- CI runtime 4 minutes 12 seconds.
- Bundle size ~340KB gzipped.
- One open escalation (LINT-501 backlog drift).

---

# Project close

## Final metrics

| Metric | Value |
|---|---|
| Total tickets delivered | 33 of 34 (97%) |
| Deferred (manual artifact) | 1 (FXSW-033 demo recording) |
| Open escalations at close | 1 (LINT-501) |
| Closed escalations during build | 6 |
| Unit tests | 316 pass / 0 todo |
| E2E tests | 6 pass in 35.9 seconds |
| CI runtime | 4m 12s (budget 5m) |
| ADRs created during build | 4 (drafted at sprint close) |
| ADRs backfilled at Sprint 1 close | 6 |
| ADRs superseded | 1 (ADR-0004 AG-Grid) |
| Spec deviations documented | 2 (notifyDealState extension, rfsMachine Removed) |
| Synthetic state added beyond spec | 1 (RFS Removed for ESP terminal coordination) |
| Mid-sprint amendments | 1 (responsive layout, FXSW-resp) |
| New dependencies added after Sprint 1 | 0 |
| Bundle size (gzipped) | ~340KB |
| Total external library imports | 5 (react, xstate, zustand, clsx, lucide-react) |

## Patterns captured for reuse

The build surfaced eleven recurring patterns now documented in `wiki/components/test-patterns.md` as a stable reference for future engineers:

1. Deterministic pricing via seed pinning — including the half-spread rounding-asymmetry footnote
2. Fake timers for `*Sent` ack delays
3. Hold-to-confirm with pointer events and double-click fallback
4. The Harness pattern for components consuming lifted state
5. State-gated vs time-gated scenario follow-ups
6. Subscribe-to-children (XState v5) rather than subscribe-to-parent
7. `queueMicrotask` to defer `actor.stop()` past the current subscription callback
8. Cell-testid scoping for number + unit compositions
9. The throwaway debug-spec workflow for diagnosing failing E2Es
10. `data-*` attributes over text or color for E2E assertions
11. Idempotent setup and teardown for feed subscribers

## What worked

- **The spec pack was load-bearing.** Every ticket cited spec sections; deviations were named, escalated, and documented. The team never asked "what should this do?" without a section reference.
- **Per-ticket TDD intensity labels were predictive.** Tickets marked 🔴 Strict consistently caught real issues before merge (the mutable `ackDelayMs` discovery, the AG-Grid blocker, the Halcyon neutral prior, the suggestion-pips testid scoping).
- **The escalation chain handled all surprises.** Every issue that needed Product input got it; every issue that needed cross-team coordination got routed. No silent overwrites.
- **The "second consumer" rule for abstraction.** The HoldButton story is the canonical example: inline at FXSW-020, second consumer at FXSW-026, lift at FXSW-030. The rule delivered the cleaner abstraction at exactly the right moment and avoided ghost code in between.
- **Phase-gate documentation cadence.** The BA's lint sweep at each sprint close caught drift while it was cheap to fix. The two surfaced cross-session escalations (LINT-002 and LINT-501) followed the same pattern: BA sees, BA can't fix due to write boundary, PM closes in the next available session.

## What we'd change

- **CI from day one.** FXSW-032 was scheduled for Sprint 5. Pulling it forward would have flipped FXSW-001 from ◐ to ☑ at first merge rather than at Sprint 5. Cost: ~30 minutes in Sprint 1.
- **Spec pluralisation pass.** "1 pip" vs "1 pips" was nowhere in the spec; surfaced in implementation. Worth a 15-minute pass through the doc pack to catch similar template-string fidelity issues before Sprint 1 opens.
- **Environment-protection visibility for the toolchain.** The deploy-job silent skip in FXSW-034 was diagnosable only from outside the build sandbox. A pre-flight check for environment allowlist configuration would have shaved an hour.

## Outstanding follow-ups

1. Capture the 30-60 second demo recording, drop at `docs/demo.mp4`, flip FXSW-033 to ☑.
2. Resolve LINT-501 in the next session: flip FXSW-034 to ☑ in `docs/BACKLOG.md` to match the live deploy state.

There is no Sprint 6.
