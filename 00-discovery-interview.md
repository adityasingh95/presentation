# FX Sales Workstation — Discovery Interview Transcript

## Session metadata

| Field | Value |
|---|---|
| Project | FX Sales Workstation (working title) |
| Stakeholder | Aditya Singh (Product) |
| Interviewer | Claude |
| Sessions | 3 over 2 days |
| Total duration | ~5h 15m active discussion |
| Output | 11-document spec pack (~1,500 lines) |

The interview ran across three working sessions before any code was written. Each session opened with a recap of the previous and closed with a list of artifacts Claude would draft for review. The artifacts were iterated overnight and confirmed at the start of the next session.

---

# Session 1 — Problem framing, users, scope

**Day -2, Tuesday morning · ~90 minutes**

**Claude:** Before we get into anything technical, can you tell me in plain language what you want to build? Pretend I know nothing about your domain.

**Aditya:** I want to build a frontend prototype of a sales-trader workstation. Specifically the screen a human uses when an FX quote request can't be priced automatically and needs manual intervention. Sales Intervention is the industry term for it.

**Claude:** When you say "can't be priced automatically" — what's actually happening upstream? Walk me through the moment a request lands on the trader's screen.

**Aditya:** A client out there in the world — could be a corporate treasury, could be a hedge fund — sends a request for a quote on a currency pair through their trading app. Behind the scenes there's an auto-pricer that handles the easy ones. But sometimes it can't or shouldn't. The trade is too big, or the client's credit is already maxed out, or it's outside normal trading hours. When that happens, the request gets routed to a human sales trader. That's the moment my screen comes alive.

**Claude:** So the screen exists at the boundary between "automation failed" and "human judgement needed."

**Aditya:** Exactly. And the trader needs to do three things fast: see why automation declined, decide how to price it, and send something back to the client. Or reject the deal if it really shouldn't be priced.

**Claude:** Got it. Couple of clarifying questions before I go further. First — is this an actual product you're shipping to traders, or a demonstration?

**Aditya:** Demonstration. It's a frontend prototype. No real backend, no real market data, no real trades. The point is to show that an LLM-driven dev workflow can deliver a polished, functional, tested workstation in about a week from a spec pack.

**Claude:** Okay, that changes a lot of constraints. It means I should design this so it's offline-runnable, demo-able from a static URL, and doesn't depend on any service that could be down during a pitch. Are you comfortable with that?

**Aditya:** Yes. Static-host deployable. GitHub Pages would be fine.

**Claude:** Second question — is this a re-implementation of a specific vendor's product? Because if it is, there are licensing and IP considerations I'd want to surface now.

**Aditya:** No. I want it to be brand-neutral. The workflow itself is a generic industry pattern — every bank that does FX manual pricing has something like this. But the product shouldn't read as a clone of any specific vendor. No vendor names in the UI, the README, the package metadata, the deployed build.

**Claude:** That's a useful guardrail. I'll capture it as an explicit operating rule for the build. Industry-standard FX terminology — "Quoted," "PickedUp," "Executable," "RFS," "Sales Intervention," "Active Deals Blotter" — those are fair game, right? They're not vendor-proprietary.

**Aditya:** Right. Those are the vocabulary of the field. Anyone who's worked in FX recognises them.

**Claude:** Good. Let's talk about users. Who's actually sitting in front of this screen all day?

**Aditya:** Primary user is a sales trader. They monitor a live blotter of deals. When something needs intervention, they click in, open a ticket, see the reason auto-pricing failed, adjust the margin, send a price back. They do this dozens of times a day. They live on the keyboard. Dense UI, no friction.

**Claude:** Secondary users?

**Aditya:** Sales-desk manager or observer — someone watching the workflow over the trader's shoulder during a demo. They care about the operational story more than the keyboard speed. And the client is off-screen — they initiate requests from a separate trading app and either accept, reject, or let the quote expire.

**Claude:** So the client never appears as a user of this screen — they're modelled as an event source.

**Aditya:** Correct.

**Claude:** Let me try to surface user stories. Tell me if these match what a trader would actually want, and add the ones I'm missing.

> 1. As a sales trader, I want a single screen showing all live deals so I can see what needs attention.
> 2. When a deal needs intervention I want visual and audible cues.
> 3. I want to mute audible notifications without losing visual ones.
> 4. I want to open a ticket and see why auto-pricing failed.
> 5. I want streaming and fixed quote modes.

**Aditya:** Yes to all. Add:

> 6. I want to adjust margin in pips with a client-price preview.
> 7. I want to send a streaming quote and see completion in Historic.
> 8. I want to release a ticket back to the desk if I can't take it.
> 9. I want to reject a deal when it should not be priced.
> 10. Terminal deals should clear from the live blotter after a brief grace period.

**Claude:** Eleven through fifteen — what about AI?

**Aditya:** Right, I want to highlight that as a feature. Add:

> 11. I want an AI-suggested margin and a one-line rationale.
> 12. I want to see the factors behind the suggestion so I can audit it.
> 13. When credit would be breached, I want a decline recommendation rather than a wider price.

And two for the demo operator:

> 14. As a demo operator, I want a hidden dev panel for scenario injection.
> 15. As a developer or reviewer, I want demo scenarios backed by automated E2E tests.

**Claude:** Number 13 is interesting. You're saying when credit is breached, the AI should *not* suggest pricing — it should suggest declining. That's the AI as a guardrail, not a gas pedal.

**Aditya:** That's exactly the framing. The "smart" moment is knowing when not to price. Anyone can widen a margin; an actual judgement call is recognising the deal shouldn't be done.

**Claude:** I'll capture that as a design principle. It also tells me the AI engine has a special case for credit-limit breaches — different output shape from the normal pricing suggestion.

**Aditya:** Yes.

**Claude:** Let's talk about scope. What is this prototype *not*?

**Aditya:** Lots of things. Spot only — no forwards, no NDF, no flexible forwards, no swaps, no block trades. Three rejection reasons only — off-hours, size limit, credit limit. No real market data. No backend at all. No auth. No persistence except a session-level UI preference like the mute toggle. No internationalisation. No accessibility certification. No real model call for the AI suggestion — it's a deterministic rule engine.

**Claude:** That last one is important. You're saying I should call it "AI" in the UI because that's how the desk would describe and perceive the feature, but the implementation is rule-based.

**Aditya:** Yes. The label is real because the product surface is the demonstration. The model is what gets swapped in for v2.

**Claude:** Goals. How will we measure whether this prototype succeeded?

**Aditya:** Six measures. One — visually credible reproduction of a modern sales-trader workstation. Two — end-to-end working flow with simulated data, a deal can be injected, priced, sent, accepted, and archived. Three — three distinct manual-pricing scenarios, each reproducible from a single click in a dev panel. Four — AI Margin Suggestion with an Apply action, working live in the ticket. Five — automated E2E test coverage of the scenarios. Six — demonstrates an agentic development loop. Built in five working days from a structured spec pack.

**Claude:** That's the six I'll lock as the PRD goals. Last question before lunch — primary browser target?

**Aditya:** Latest Chrome. Desktop-first. If it works on a tablet that's nice but not required. Mobile redesign explicitly out.

**Claude:** Understood. I'll draft the PRD overnight covering everything we discussed — problem statement, goals, personas, user stories, scope, non-goals, constraints. We'll review at the start of Session 2 before going into domain modelling.

> **Artifacts queued for draft:** `docs/01-prd.md` — Background, Problem statement, Goals, Non-goals, Personas, User stories, Success criteria, Constraints, Open questions, Out-of-scope.

---

# Session 2 — Domain modelling, state machines, scenarios

**Day -2, Tuesday afternoon · ~2 hours**

**Claude:** I read back the PRD draft this morning — you had two edits, both captured. Now I want to go deep on the domain. Walk me through one complete deal from request to settlement. Take your time.

**Aditya:** Okay. Client wants to buy ten million euros against US dollars. They go to their trading app and either request a stream — a continuously updating price — or a single quote at a moment in time. The request hits the bank's quote handler. Quote handler asks the pricing system for a rate, asks the risk system whether the trade is okay credit-wise and limits-wise. If both come back green, the auto-pricer sends a price to the client. Client either hits the price, lets it expire, or rejects it. If they hit it, the trade books.

**Claude:** And the failure paths?

**Aditya:** Three relevant for our prototype. Off-hours — the request came in outside the auto-pricer's configured trading window for that pair. Size limit — the notional exceeds what the auto-pricer is willing to handle without human review. Credit limit — the client's available credit would be breached if this trade booked. In all three cases, the auto-pricer rejects, and the quote handler routes the request to the sales desk. A human picks it up.

**Claude:** So the manual workflow is essentially a recovery path for an upstream decline.

**Aditya:** Exactly. The trader sees a notification, opens the ticket, reviews why automation declined, applies judgement, and either sends a price or rejects the deal.

**Claude:** Let me sketch the screen at a very high level and you tell me if I have the geography right. There's a header at the top. Underneath, a live blotter of active deals — the ones currently in flight, including ones the trader is actively working. And below that, or somewhere else on the page, a historic blotter showing completed or rejected deals. When the trader clicks an active row that needs intervention, a ticket panel opens.

**Aditya:** That's right. Active deals on top, Historic below it. The ticket is a right-side overlay panel that slides in.

**Claude:** Inside the ticket — what panels does the trader actually need?

**Aditya:** Several. A Reasons Panel showing why auto-pricing failed. A Summary Panel — a natural-language sentence describing the deal. The AI Margin Suggestion Panel — that's the feature. A Pricing Panel showing trader bid/ask and a margin control. A Client Summary Panel — what the client will see, including estimated profit. A Deal Summary Panel with the trade specifics. And a footer with action buttons.

**Claude:** Footer actions?

**Aditya:** Five depending on state. Reject, Release, Send Stream, Send Quote, Withdraw, and Return to Stream. Not all are visible at the same time — they're gated by what state the deal is in.

**Claude:** Now this is going to be the most important question for the architecture. When you say "what state the deal is in" — what *is* the state machine? Is there one lifecycle, or more than one?

**Aditya:** There are two. And they run in parallel.

**Claude:** Tell me more.

**Aditya:** A single deal has two related lifecycles. One is the RFS lifecycle — the client-side request-for-stream lifecycle. The other is the Sales Intervention lifecycle — the trader-side manual-pricing lifecycle. The states aren't identical. The UI often needs both to derive what to show.

**Claude:** Why two and not one combined machine? You could imagine flattening them — states like `Queued-Initial`, `PickedUp-PickedUp`, `Executable-Quoted`.

**Aditya:** You could, but you'd lose clarity. The two lifecycles are documented separately in the industry. They represent different concerns — client-side vs. trader-side. And if a future version of this prototype talks to a real backend, each machine becomes a thin adapter onto a real state stream. Collapsing them now would force a restructure later.

**Claude:** That's a strong rationale. I'll capture it as the first architectural decision. Two parallel state machines per deal, coordinated by a parent. Now let me extract the states. What are the RFS states?

**Aditya:** Six prototype states. `Queued` — request is waiting for pricing or manual intervention. `PickedUp` — a sales trader has picked it up. `Executable` — a price is live with the client. `TradeConfirmed` — trade is booked. `ClientClosed` — client closed the request. `Expired` — request expired.

**Claude:** And the SI states?

**Aditya:** Eleven. `Initial` — deal available for pickup. `PickUpSent` — pickup acknowledgement is pending. `PickedUp` — trader owns the ticket. `QuoteSent` — quote or stream acknowledgement is pending. `Quoted` — quote or stream is live. `WithdrawSent` — withdraw acknowledgement is pending. `HoldSent` — release acknowledgement is pending. `RejectSent` — reject acknowledgement is pending. `TraderRejected`, `ClientRejected`, `TradeConfirmed` — terminals.

**Claude:** I'm noticing a pattern — there's a `*Sent` family of states. PickUpSent, QuoteSent, WithdrawSent, HoldSent, RejectSent. These look like in-flight acknowledgement states.

**Aditya:** Yes. In a real deployment they're visible because there's real network latency between the trader clicking a button and the trading backend acknowledging. The prototype has no backend.

**Claude:** So we have a choice. Either we collapse the `*Sent` states because they're vestigial for the prototype, or we keep them with a simulated ack delay. There's an argument either way. Collapsing is simpler. Keeping them gives UX fidelity — the button shows a "sending" affordance, the demo feels real.

**Aditya:** Keep them. The 200 to 300 millisecond window matters for how the screen feels. And v2 talking to a real backend doesn't have to re-introduce them.

**Claude:** Then I want to bake in one more property: the simulated delay should be configurable so tests can zero it. Otherwise every E2E test waits 250ms five times per scenario.

**Aditya:** That makes sense.

**Claude:** Good. I'll capture that as an ADR — `*Sent` states kept, 250ms default ack delay, zeroable from tests via a window-level flag. Now the cross-model relationships. When the trader picks up a deal, what happens on both machines?

**Aditya:** SI transitions from `Initial` through `PickUpSent` to `PickedUp`. RFS transitions from `Queued` to `PickedUp`.

**Claude:** When the trader sends a stream?

**Aditya:** SI from `PickedUp` through `QuoteSent` to `Quoted`. RFS receives a price update and goes to `Executable`.

**Claude:** Withdraw?

**Aditya:** SI from `Quoted` through `WithdrawSent` back to `PickedUp`. RFS goes from `Executable` back to `PickedUp`.

**Claude:** Release?

**Aditya:** SI from `PickedUp` or `Quoted` through `HoldSent` back to `Initial`. RFS returns to `Queued` or whatever pre-pickup state.

**Claude:** Reject?

**Aditya:** SI through `RejectSent` to `TraderRejected`. That's a terminal.

**Claude:** Client decline or timeout?

**Aditya:** SI to `ClientRejected`. Terminal.

**Claude:** Trade booked?

**Aditya:** Both to `TradeConfirmed`. Terminal.

**Claude:** I'll write this up as a relationship table — one row per cross-model event with the SI transition and the RFS effect side by side. That table becomes the canonical reference both for the implementation and for the tests.

**Aditya:** Yes.

**Claude:** One more thing on terminal handling. Once a deal terminates — confirmed, rejected, declined, expired — how long does it stay in the live blotter before moving to Historic?

**Aditya:** Five seconds. Long enough that the trader sees what happened and can read the outcome. Then it should auto-archive.

**Claude:** Five-second removal rule. I'll capture that. One question I want to flag now and resolve in the architecture session — the 5-second removal needs a timer that doesn't get zeroed in tests, otherwise the demo and the production behaviour diverge. Tests can zero the *Sent ack delay but should respect the 5-second wall clock.

**Aditya:** Good catch.

**Claude:** Display status. The blotter row shows a status pill — what should it show?

**Aditya:** It's derived from both states plus a "dealable" flag. There's a mapping — let me walk through it.

> Display statuses we need to support: `AUTO`, `INTERVENE`, `PICKED UP`, `QUOTING`, `QUOTED`, `WITHDRAWING`, `RELEASING`, `REJECTING`, `DONE`, `REJECTED`, `DECLINED`, `EXPIRED`.

For each one, the mapping is roughly:

| Condition | Display |
|---|---|
| ESP flow, executable or complete | `AUTO` then `DONE` |
| SI `Initial` and dealable | `INTERVENE` |
| SI `PickUpSent` or `PickedUp` | `PICKED UP` |
| SI `QuoteSent` | `QUOTING` |
| SI `Quoted` | `QUOTED` |
| SI `WithdrawSent` | `WITHDRAWING` |
| SI `HoldSent` | `RELEASING` |
| SI `RejectSent` | `REJECTING` |
| SI `TraderRejected` | `REJECTED` |
| SI `ClientRejected` | `DECLINED` |
| SI `TradeConfirmed` | `DONE` |
| RFS `Expired` | `EXPIRED` |

**Claude:** That becomes a pure function. `derivedStatus(rfsState, siState, dealable)`. Hundred percent branch coverage candidate.

**Aditya:** Yes.

**Claude:** Quick break, then the scenarios. I want five concrete demo scenarios with specific data — client names, accounts, currency pairs, notionals, rejection reasons — so the implementation has data to lock in and the tests have something to assert against.

**Aditya:** Let me think about it for a minute.

*[break]*

**Aditya:** Five scenarios. Here they are.

> **HAPPY_PATH_ESP.** Auto-priced EURUSD trade for Acme Corp, 1 million EUR, BUY. Flows AUTO to DONE to Historic without any trader involvement. Purpose: show ESP path completes end-to-end.
>
> **OFF_HOURS_INTERVENTION.** Off-hours USDJPY for Globex Industries, 5 million notional, SELL. Trader opens ticket, sends stream, client accepts after 1.5 seconds. Purpose: full SI happy path.
>
> **CREDIT_BREACH.** Credit-breach GBPUSD for Halcyon Capital, 25 million notional, BUY. AI panel flips to credit-decline state, trader rejects. Purpose: AI guardrail and trader reject.
>
> **SIZE_LIMIT_MARGIN_TUNE.** 12-million EURUSD for Northwind FX, SELL, size-limit reason. AI suggests 4 pips at high confidence. Trader applies the suggestion, sends stream, client accepts. Purpose: AI Apply path and live client-summary update.
>
> **RELEASE_PATH.** 3-million USDINR for Polaris Holdings, BUY, size-limit reason. Trader picks up the ticket, then releases. Row stays in Active. Purpose: hand-back-to-desk flow.

**Claude:** I'll write each one up in Gherkin so the test author can implement them directly. And the scenario IDs become injection labels in a dev panel — one button per scenario.

**Aditya:** Right.

**Claude:** Should the dev panel ship in the live URL, or hidden by default?

**Aditya:** Hidden by default. Visible only with a query parameter like `?dev=1`. Otherwise an actual end user could end up in the demo state.

**Claude:** Got it. One more domain question — when the AI suggests a margin and the trader applies it, what should happen visually?

**Aditya:** The margin field animates from current to suggested over about 200 milliseconds. A subtle outline glow on the margin input — indigo, fades over 600ms. The suggestion panel collapses to a single-line "Applied N pips · Undo" strip. Undo restores the previous margin and re-expands the panel.

**Claude:** Indigo? Specifically?

**Aditya:** Yes — there's a design choice I want to lock in. The AI surfaces use indigo-violet. Nothing else in the UI does. Mental model: anywhere the trader sees indigo plus a sparkle icon, it's an AI surface.

**Claude:** That's a strong discipline. I'll capture as an ADR. Reserved exclusively for AI surfaces, soft rule enforced by code review.

**Aditya:** Yes.

**Claude:** End of Session 2. Tonight I'll draft the glossary, the trade state model document with the full cross-model table, the dummy feed spec with the five scenarios in Gherkin, and stub the architecture and UI docs we'll finalise tomorrow. We'll reconvene in the morning.

> **Artifacts queued for draft:** `docs/00-glossary.md`, `docs/03-trade-state-model.md`, `docs/04-dummy-feed-spec.md`, `docs/07-scenario-pack.md`, ADR-0002 (two state machines), ADR-0009 (simulated ack delays), ADR-0008 (indigo accent).

---

# Session 3 — UI/UX, technology stack, AI engine, test plan

**Day -1, Wednesday morning · ~2 hours**

**Claude:** Three documents reviewed overnight — small edits captured. The cross-model table is locked. Today is UI/UX, the technology stack, the AI engine internals, and the test plan. Let's start with visual aesthetic. Show me references.

**Aditya:** Modern dark trading-app. Think Talos, ION Anvil, the new Robinhood Pro. Refined typography. Considered spacing. Subtle motion. Confident colour. Dark, dense enough for a trader's eye, professional.

**Claude:** What it's NOT?

**Aditya:** Not Bloomberg Terminal. Not the dense lo-fi look. And not consumer SaaS either. No big hero text. No pastel cards. No oversized rounded corners. No playful illustrations. The trader is a professional; the copy should be clipped and confident.

**Claude:** Typography?

**Aditya:** Geist for sans, Geist Mono for numbers. Numbers always mono — bid, ask, mid, amounts, pip counts, deal IDs. Everything else sans. Tabular numerals throughout — `font-variant-numeric: tabular-nums`. Tightened tracking on numerals, looks more settled.

**Claude:** Specific colour tokens?

**Aditya:** Yes — let me give you the palette.

> Backgrounds: deeper than utilitarian trader-dark. `#0a0a0f` app background, `#111118` panels, `#16161f` panel-2, `#1c1c27` elevated, `#1f1f2b` row hover. Glass overlay at `rgba(22, 22, 31, 0.78)` with `backdrop-filter: blur(20px) saturate(140%)`.
>
> Status colours, slightly desaturated for sophistication. Amber `#fbbf24` for pending intervention. Blue `#3b82f6`. Teal `#14c4a8` for streaming. Green `#22c55e` for done and BUY. Red `#ef4444` for rejected and SELL.
>
> AI accent family — indigo-violet. `#818cf8` primary, `#a78bfa` gradient companion, `rgba(99, 102, 241, 0.08)` for a subtle wash background. Used only on the AI Suggestion Panel, the sparkle icon, and a 2-pixel gradient strip at the very top of the header — the only place outside the AI panel that hints at the AI theme.

**Claude:** Sizes — text scale?

**Aditya:** Eight steps. 11px for metadata and timestamps. 12 for default blotter text. 13 for body and ticket labels. 14 for ticket section titles. 18 for the trader rate display. 26 for big bid/ask boxes. 32 for the AI suggestion number — that's the visual moment.

**Claude:** Spacing scale?

**Aditya:** 4, 8, 12, 16, 20, 24, 32, 40. Slightly more generous than a pure Bloomberg layout. Lets the modern aesthetic breathe while staying dense.

**Claude:** Radii?

**Aditya:** Sharper at the small end, modern at the large. 4 for pills, 6 for buttons and inputs, 10 for panels, 14 for the ticket overlay corners and the AI panel. Nothing above 14 — that's where it starts to feel consumer.

**Claude:** Animations?

**Aditya:** Strict budget. Trading UIs that animate too much feel slow. Ticket slide in/out at 240ms, cubic-bezier(0.16, 1, 0.3, 1) — that's the primary easing curve. Glass blur fade-in 180ms. Row flash on a new intervention 320ms, single shot. Rate cell tick flash 80 milliseconds linear. Toast slide in 200ms out 160ms. Status pill colour change instant — the state matters more than the transition. Margin field animation on AI Apply 200ms with a 600ms glow fade.

**Claude:** No spring physics, no staggered choreography, no skeleton loaders elsewhere?

**Aditya:** Right. State is in-memory; loading is instantaneous everywhere except the simulated `*Sent` window and the AI Suggestion's 800ms recompute debounce.

**Claude:** Layout. Tell me the geography exactly.

**Aditya:** Header at the top, 56 pixels tall. Product name on the left, dev injector slot in the middle when enabled, mute toggle and a lightweight clock on the right. A 2-pixel gradient strip at the very top — blue on the left, AI accent indigo on the right. Linear gradient.

Active Deals Blotter takes 55% of the remaining vertical. Columns: status, time, client, account, pair, side, amount, tenor, rate, reasons.

Historic Deals Blotter takes the remaining 45%. Read-only.

When the ticket is open, it slides in from the right — 640 pixels wide on desktop. The blotters dim to 75% opacity behind it. Glass surface, backdrop blur. The AI panel inside has a subtle indigo edge glow.

**Claude:** Component inventory. What should I build, and what should I not?

**Aditya:** Build only what's needed. No premature kits.

> **Shared components:** Button (variants primary, secondary, ghost, danger; sizes sm and md; supports a `holdToConfirm` prop), IconButton, Pill (status labels accepting a colour token name), Chip (rejection reasons), NumberInput (margin entry with +/- buttons), Tooltip, Toast.
>
> **Blotter:** ActiveBlotter, HistoricBlotter, StatusCell, RateCell (live updating with flash on change), AmountCell, ReasonsCell.
>
> **Ticket:** TicketPanel (the overlay container), ReasonsPanel, SummaryPanel, SuggestionPanel (the AI one, indigo accented), PricingPanel (most complex — streaming/fixed mode, margin controls), ClientSummaryPanel, DealSummaryPanel, TicketFooter.
>
> **Notifications:** ToastStack, useNotificationSound (WebAudio hook with audio-unlock), MuteToggle.
>
> **Dev injector:** DevInjector — visible only with `?dev=1`.

**Claude:** Iconography?

**Aditya:** Lucide-react only. No emoji. Sized to match surrounding text — 12, 14, or 16 pixels.

**Claude:** Now the tech stack. What's the build target?

**Aditya:** Single-page React app. No backend. Static-host deployable. The standard 2024-26 default for a new React app is Next.js, but that's overkill — no SEO, no SSR, no per-route data fetching, no auth. Add it and you carry the configuration surface for nothing.

**Claude:** So Vite.

**Aditya:** Vite 5 plus React 18 plus Tailwind CSS 3. Plus tokens in CSS custom properties, mirrored as Tailwind theme extensions. Geist and Geist Mono via the dedicated font packages.

**Claude:** State management?

**Aditya:** Two libraries. XState 5 for the deal lifecycle — the trade model is a state machine, use the right tool. Zustand for UI and transient state — ticket open/close, mute toggle, focus, pending toasts. They're tiny, no overlap.

**Claude:** That's two libraries to learn. You comfortable with the cognitive cost?

**Aditya:** Yes. XState models the documented state graph directly — transitions, guards, side-effect timers, the 5-second removal. Putting state machines in a reducer pattern would fight the spec. And forcing UI state through machine actors would fight Zustand's simple shape.

**Claude:** Tables?

**Aditya:** AG-Grid Community 31 for the blotters. It's the de facto trading-blotter grid. Free tier covers everything we need.

**Claude:** Flagging a concern — does AG-Grid 31 support per-row `data-*` attributes through its public API? Because our test contract is going to be built on `data-deal-id`, `data-rfs-state`, `data-si-state`, and similar attributes.

**Aditya:** Honestly, I haven't checked. Worth confirming early. If it doesn't, plan B is a plain flex-row table.

**Claude:** I'll add it as an open risk in the architecture doc — R1: AG-Grid `data-*` API verification. If it doesn't support it cleanly we supersede the decision in implementation.

**Aditya:** Good.

**Claude:** Testing?

**Aditya:** Vitest plus React Testing Library for unit and component. Playwright for E2E, Chromium-only, single worker. Best-in-class for finance UIs — auto-waits, deterministic with seed pinning.

**Claude:** Package manager and tooling?

**Aditya:** pnpm. ESLint and Prettier. TypeScript strict, no `any`, no `// @ts-ignore`.

**Claude:** No CSS-in-JS? No UI kits like Material or Chakra or Mantine?

**Aditya:** No. Tailwind plus the component inventory I listed is sufficient. CSS-in-JS adds a runtime layer we don't need.

**Claude:** Pinned versions. Want me to write them down explicitly?

**Aditya:** Yes — pin everything in the architecture document. React 18.3.1, react-dom 18.3.1, typescript 5.4.5, vite 5.2.10, tailwindcss 3.4.3, zustand 4.5.2, xstate 5.13.0, @xstate/react 4.1.1, ag-grid-community 31.3.2, ag-grid-react 31.3.2, lucide-react 0.378.0, clsx 2.1.1, vitest 1.6.0, @testing-library/react 16.0.0, @playwright/test 1.44.0.

**Claude:** Folder structure?

**Aditya:** Standard. `/src` with `/components` (shared dumb components), `/features` with `/blotter`, `/ticket`, `/notifications`, `/dev-injector`. `/services` for `/feed`, `/scenarios`, `/suggestion`. `/state` for `/stores` (Zustand) and `/machines` (XState). `/types`, `/lib`, `/styles`. Tests colocated for components and pure logic, with `/tests/e2e` for Playwright.

**Claude:** Deployment?

**Aditya:** GitHub Pages. The deployed URL with `?dev=1` is the demo artifact. The `predev` and `prebuild` scripts fetch fresh reference FX mids from a public API like Frankfurter — daily-resolution data is enough for a random-walk simulator. Hard-coded fallback if the network fetch fails so the build never breaks.

**Claude:** CI?

**Aditya:** GitHub Actions on PR and push to main. Five-minute total budget. Steps: install with frozen lockfile, typecheck, lint, unit test run, Playwright install Chromium with deps, E2E run. Upload Playwright traces on failure as a 7-day artifact.

**Claude:** Now the AI Margin Suggestion engine. This is the feature. Walk me through how a trader would experience it.

**Aditya:** The trader opens a ticket. After they pick up the deal — state is `PickedUp` — the suggestion panel renders. There's a confidence badge in the top right — high, medium, or low. The suggested pip count in 32-pixel mono. A one-line rationale below it. Apply button on the left, "Why?" button next to it.

If the trader clicks Apply, the margin field animates to the suggested value. The panel collapses to a one-line "Applied N pips · Undo" strip. Undo restores the previous margin.

If they click Why?, a factors table expands showing every contributor — factor name, delta in pips, short note.

If the rejection reason includes credit-limit, the panel is in a different state — "AI Recommendation," amber alert icon, message "Credit limit breach — recommend declining." A Reject shortcut button replaces Apply.

**Claude:** What are the inputs to the suggestion function?

**Aditya:** Deal context — pair, side, notional, default margin pips, rejection reasons. Client profile — tier, recent volume, recent acceptance rate, average margin paid, behaviour flag. Market context — current bid and ask, pair volatility, session liquidity.

**Claude:** The rule structure?

**Aditya:** Tier baseline, then deltas applied in order. Tier base: platinum 1.5, gold 2, standard 3, new 4. Then notional bands — above 20M USD-equivalent adds 2.5, 10 to 20M adds 1.5, 5 to 10M adds 0.5. Then market — pair volatility above 1.5 adds 1, thin session liquidity adds 1.5, high-vol pairs USDJPY and USDINR add 0.5. Then rejection reasons — off-hours adds 1.5, size-limit adds 0.5. Credit-limit short-circuits — no margin proposed. Then behaviour — flight-risk client subtracts 1 to win retention, high-engagement VIP with above 100M volume subtracts 0.5, acceptance rate below 0.4 subtracts 0.5. Floor at 1 pip, round to integer.

**Claude:** Confidence?

**Aditya:** High for established client tier with at least 10M volume and non-thin liquidity. Low for new tier, thin liquidity, or volume below 1M. Medium otherwise.

**Claude:** Five client profiles?

**Aditya:** Acme Corp — platinum, 450M 30-day USD volume, 0.78 acceptance rate, high-engagement. Globex Industries — standard, 35M, 0.62, normal. Halcyon Capital — new, no volume, no acceptance history, normal. Northwind FX — gold, 120M, 0.71, high-engagement. Polaris Holdings — standard, 18M, 0.34, flight-risk.

**Claude:** Halcyon has no acceptance history — how do we encode that? Because the rule "acceptance rate below 0.4 subtracts 0.5" would fire if we encoded zero.

**Aditya:** Use a neutral prior — 0.5. New client, no observed quotes, no penalty.

**Claude:** Good. I'll flag that in the seed-data file with an inline comment so the next reader doesn't change it back to zero.

**Aditya:** Yes.

**Claude:** Rationale builder?

**Aditya:** Composes 2-3 of the largest-magnitude factors into a natural sentence. Cap at 120 characters. Truncate from the lowest-magnitude factor if longer.

**Claude:** Per the canonical examples — for the 12M EURUSD Northwind case, the rationale should read something like "Gold-tier client with 12M EURUSD above auto-pricer band — suggesting 4 pips." For the 5M USDJPY Globex off-hours case, "Standard-tier client, off-hours USDJPY — suggesting 5 pips."

**Aditya:** Right.

**Claude:** Reactivity. Does the suggestion recompute on every price tick?

**Aditya:** No — that would be noise and create overconfidence. Recompute only on three triggers: when the ticket opens after `PickedUp`, when pair volatility shifts by more than 30% from the previous compute, or when the trader explicitly clicks recompute. Debounce 800 milliseconds.

**Claude:** Last topic — test plan. Three layers, three questions?

**Aditya:** Yes. Unit tests with Vitest answer "does this pure function compute the right number?" Component tests with Vitest and React Testing Library answer "does this component render the right thing?" E2E tests with Playwright answer "does the user's journey work?"

**Claude:** Coverage floors?

**Aditya:** Above 90% lines for `/src/lib` and `/src/state/machines`. 80% for the feed and scenarios services. **100% branch coverage** on the suggestion engine — most-tested file in the codebase. 80% for the PricingPanel and SuggestionPanel components. Smoke tests on other components. All five named scenarios as E2E tests, plus a notifications spec.

**Claude:** Test contract. What do E2Es assert on?

**Aditya:** `data-*` attributes. Never text, never colour. Specifically `data-si-state`, `data-rfs-state`, `data-display-status`, `data-dealable`, `data-deal-id`, `data-removing`, plus `data-testid` for component identification. Text and colour may change; the testids and canonical state names should not.

**Claude:** Determinism levers?

**Aditya:** Two. `window.__seedFeed = 42` for deterministic pricing — pinned in Playwright via `addInitScript` before page load. `window.__zeroAckDelay = true` for instant `*Sent` transitions — also pinned in Playwright. The 5-second blotter-removal delay is *not* zeroed; it's a real wall-clock UX behaviour the demo and tests both depend on.

**Claude:** Component tests. What components are most worth testing carefully?

**Aditya:** PricingPanel is the most complex — streaming and fixed modes, margin controls, +/- buttons, fixed-side capture, the margin animation when AI Apply fires. SuggestionPanel second-most — ready/applied/Undo states, the credit-decline shortcut, the "Why?" factors table. TicketFooter for the hold-to-confirm wiring. MuteToggle for sessionStorage persistence. ReasonsPanel for the chip rendering. ClientSummaryPanel for the pip arithmetic.

**Claude:** Pure logic worth deep coverage?

**Aditya:** `lib/pips.ts` — `pipSize`, `addPips`, `applyMargin`. JPY pairs use 2-decimal precision; non-JPY uses 4. Most error-prone functions in any FX UI. Cover both explicitly. The XState machines — every transition. The status derivation function — every row in the mapping table. The suggestion engine — every branch.

**Claude:** TDD intensity for the backlog?

**Aditya:** Three tiers. Strict — test list MUST be written first, run red, then implement to green. Used for pure logic, the state machines, the suggestion engine. Alongside — component API and behaviour tests written together. Acceptance — the Gherkin scenario IS the test, written when a user-facing flow lands end-to-end.

**Claude:** Final question — backlog structure. How do you want the work split?

**Aditya:** Five phases mapping to the spec sections. Phase 1 scaffold and first slice — 6 tickets. Phase 2 feed and state coordination — 7 tickets. Phase 3 ticket panels and actions — 8 tickets. Phase 4 AI Margin Suggestion — 6 tickets. Phase 5 notifications, polish, ship — 6 tickets plus the deploy workflow. Each ticket cites the relevant spec sections, has an AC block, and lists the TDD test slate to write first.

**Claude:** That's 34 tickets across 5 phases. Roughly one ticket per hour of focused work plus integration time. Five working days target.

**Aditya:** Yes.

**Claude:** End of Session 3. Tonight I'll draft the remaining documents — functional spec, UI/UX spec with all the tokens we discussed, tech architecture document, AI suggestion engine document, test plan, and the BACKLOG. Plus the ADRs for the decisions captured: Vite over Next.js, the two state machines, XState plus Zustand, AG-Grid Community (with the R1 flag), Frankfurter bake-time mids, deterministic suggestion engine, credit-breach recommend-decline, indigo AI accent, simulated ack delays, brand-neutral build. Ten ADRs total.

> **Artifacts queued for draft:** `docs/02-functional-spec.md`, `docs/05-ui-ux-spec.md`, `docs/06-tech-architecture.md`, `docs/08-test-plan.md`, `docs/09-suggestion-engine.md`, `docs/BACKLOG.md`, ten ADRs in `/wiki/decisions/`.

We'll review tomorrow morning. If everything is signed off, the build can open on Monday.

---

# Day -1 evening — Aditya's review notes

| Document | Status | Notes |
|---|---|---|
| `docs/00-glossary.md` | ✓ approved | One addition — "Dealable" as a blotter field |
| `docs/01-prd.md` | ✓ approved | None |
| `docs/02-functional-spec.md` | ✓ approved | Minor clarification on hold-to-confirm scope |
| `docs/03-trade-state-model.md` | ✓ approved | Cross-model table reads correctly |
| `docs/04-dummy-feed-spec.md` | ✓ approved | None |
| `docs/05-ui-ux-spec.md` | ✓ approved | None — token values exactly as discussed |
| `docs/06-tech-architecture.md` | ✓ approved | R1 (AG-Grid `data-*`) noted in open questions |
| `docs/07-scenario-pack.md` | ✓ approved | None |
| `docs/08-test-plan.md` | ✓ approved | None |
| `docs/09-suggestion-engine.md` | ✓ approved | Halcyon neutral-prior note included |
| `docs/BACKLOG.md` | ✓ approved | TDD intensity labels per ticket confirmed |
| 10 ADRs | ✓ approved | Original-decision dates kept as 2026-05-20 |

---

# Operating model agreed for the build

Three concurrent agents on the project once code work opens:

**Build agent.** Owns `/src`, `/tests`, `/scripts`, `/docs`. Implements tickets in order within each phase. Reads the cited spec sections before writing tests; writes tests before implementation on strict tickets; commits with the ticket ID in the message. Updates `docs/BACKLOG.md` status as tickets close. Authors phase-summary briefs at each phase gate.

**Product (Aditya).** Above the agents. Owns scope decisions, tiebreakers on contested calls, mid-flight amendments. Approves wiki drafts before they land. Invokes the wiki agent at the documented phase gates.

**Wiki agent.** Owns `/wiki` and `/raw`. Ingests the build agent's phase-summary briefs. Updates feature and component pages, drafts new ADRs as needed, maintains the index and glossary. Runs an eleven-category lint sweep at each phase gate including a code-drift check between the wiki and the source. Vendor-neutrality is a stricter rule on the wiki side — no vendor names anywhere, including citation URLs. Phase-summary briefs hand off at `docs/phase-summaries/FXSW-{last-ticket-id}-summary.md` rather than the default location, so the build agent's write boundary stays inside `/docs`.

The wiki agent never writes to `/src` or `/tests`. The build agent never writes to `/wiki` or `/raw`. They communicate via files — the build agent saves a phase-summary brief at each gate; the wiki agent ingests from there.

There is no implicit Phase 6. The five phases close the prototype. Any extension beyond it is a new project with new spec.

---

*Total interview time across three sessions: 5 hours 15 minutes. Total artifact output: 11 documents, 10 ADRs, ~1,500 lines. Project ready to open Monday morning.*
