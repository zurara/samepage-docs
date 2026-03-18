# GEMU-1: VM Selection & Team Credit Management

> Part of [VM-1: Virtual Machine Simulation — Full Feature](./VM-1-virtual-machine-simulation-full.md) · **Phase 1 — Early Access**

**Status**: `Ideation`
**Owner**: @TODO
**Priority**: `P1`
**Target**: TODO
**Progress**: 0/16 tasks ✓


## Why

Engineers need to choose the right compute environment for their simulation and have clear visibility of their own usage within the team quota. This feature introduces a simple credit display (team quota + your usage) and a VM selection flow designed to scale as more machine types become available. Full team usage breakdown is managed in the backend admin panel, not exposed in the UI.

## Who

GEMU team members using the ODE Platform. Each member sees only their own usage and the shared team quota — not other members' usage. Full breakdown is admin-only.

Feature visibility is controlled at the user level — specific ODE user accounts belonging to the GEMU team are flagged in the backend to enable access. Non-flagged users on the same platform see no difference in their UI.

## What

**Core Capabilities:**
- Member selects simulation environment from a 3-option dropdown in the setup panel
- VM option opens a modal to configure machine settings — designed to support multiple machine types in future
- Home / payment page shows a "Computation Credits" section with one row per VM type, each showing its own team quota and your usage
- Each VM has an independent credit gate — one VM being out of credits does not affect others
- Hours displayed as decimals (e.g. 1.50 hr, 0.03 hr)
- No other members' usage is shown — full breakdown is backend admin only
- Credit gate follows metro card model: credit > 0 allows VM launch; credit ≤ 0 blocks new launches
- In-flight simulations are never interrupted — block applies to new launches only
- Top-up covers deficit first, then restores usable balance
- Feature is Early Access — visible to whitelisted users only, indicated by icon

**User Flow:**
View per-VM quota + your usage (Home → Computation Credits) → Select simulation environment in setup dropdown → If VM selected: configure in modal → Launch blocked if that VM's team credit ≤ 0 → Credit billed per minute (rounded up), displayed as decimal hours → If balance hits 0 mid-run, current simulation completes → Next launch blocked until support tops up

## Key Terms

- **Team Credit**: A shared credit pool per VM type. Each VM has its own independent quota allocated to the team. Unit is hours (decimal).
- **Team Quota (per VM)**: Total hours allocated to the team for a specific VM type, across all top-up batches.
- **Top-up Batch**: A single top-up purchase with its own hour amount and expiry date. Multiple batches can exist per VM type.
- **Your Usage (per VM)**: How much credit (in decimal hours) the current user has consumed from a specific VM's quota.
- **Team Remaining (per VM)**: Sum of all unexpired batch hours minus total member usage. This is the operative balance for the credit gate.
- **Consumption Order**: Hours are consumed from the earliest-expiring batch first.
- **Expiry Warning**: A red indicator shown on a batch row when it is expiring soon. No hover hint — the indicator itself communicates urgency.
- **VM Modal**: A configuration dialog that opens when "Run on virtual machine" is selected. Currently shows one machine spec; designed to support multiple machine types in future.
- **Credit Gate (per VM)**: Independent per VM type. Credit > 0 allows launch on that VM. Credit ≤ 0 blocks new launches on that VM only.
- **Credit Metering Unit**: Billed per minute, expressed as decimal hours. Runtime rounded up to nearest minute — e.g. 1 min 30 sec → 2 min → 0.033 hr deducted.
- **Top-up**: Adding a new hour batch to a VM's quota — handled by support in V1. Deficit settled first before new balance is usable.
- **Early Access**: Feature only visible to ODE users whose accounts are flagged as GEMU members in the backend. Non-flagged users see no VM option and no Computation Credits section. Indicated by icon on visible UI elements.

---

## Decisions

**Feature visibility — user-level flag on ODE Platform:**
- The VM option and Computation Credits section are only rendered for ODE user accounts flagged as GEMU members. Flagging is managed in the backend — no UI toggle. Non-flagged users on the same ODE instance see no change to their experience. This is not a role or permission system; it is a simple boolean flag per user account.

**Top-up batch model — multiple batches per VM:**
- Each top-up creates a new batch with its own hour amount and expiry date. Multiple batches can coexist per VM type. Hours consumed from the earliest-expiring batch first — reduces risk of credit loss from expiry.

**Expiry display — per batch, totalled:**
- Each batch is shown as its own row with its expiry date. Total remaining across all batches is shown as a summary. Batches expiring within 30 days display a red indicator — no hover hint, the colour communicates urgency.

**Total remaining is sum of all unexpired batches:**
- `Team remaining` = sum of all batch hours minus total member usage. Not shown per-batch. The batch list shows each batch's original amount and expiry, not remaining per batch — keeping the calculation simple on the frontend.

**Consumption order — FIFO by expiry:**
- Hours deducted from the batch with the earliest expiry date first. Backend handles this automatically. Frontend does not need to display which batch is currently being consumed.

**Hours displayed as decimals:**
- All credit values displayed as decimal hours (e.g. 1.50 hr, 0.03 hr). No HH:MM:SS format. Keeps the display simple and consistent with how quotas are set and topped up.

**VM selection opens a modal:**
- Instead of showing machine specs inline in the dropdown subtitle, VM option triggers a modal. This keeps the dropdown clean and allows future machine type selection, configuration options, and spec comparison without redesigning the dropdown. V1 modal shows one machine spec only.

**Credit gate — metro card model (per VM):**
- Per VM: credit > 0 allows new launch; credit ≤ 0 blocks new launch on that VM. In-flight simulations always complete. Top-up settles deficit first, then remainder becomes usable. e.g. balance = –0.5 hr, top-up = 10 hr → new balance = 9.5 hr.

**Credit metering — per minute, rounded up:**
- Runtime rounded up to nearest minute. e.g. 1 min 30 sec → 2 min → 0.033 hr deducted. Stored and displayed as decimal hours.

**No admin panel in V1:**
- Team quota management, per-member allocation, and usage reports are out of scope. This version is read-only visibility for all team members.

**Copywriting — Simulation Setup Dropdown:**
- Label: `Run on`
- Options:
  - `Run in local browser` · `Private · Free · Best for small models`
  - `Run on cloud server` · `1vCPU · 8 GB RAM · Subscription`
  - `Run on virtual machine` *(Early Access icon)* · `2vCPU · 16 GB RAM · Top-up Credits · billed per min`
  - `Run on virtual machine` *(disabled)* · `2vCPU · 16 GB RAM · Out of Credits`
- Disabled VM hover hint: *"No credits remaining. Contact support to top up."*

**Copywriting — Computation Credits Section (Home / Payment):**
- Section title: `Computation Credits`
- Per VM type, display:
  - VM spec label (e.g. `2vCPU · 16 GB RAM`)
  - `Your usage` · `[decimal] hr`
  - Batch list — one row per top-up batch:
    - `[amount] hr` · `Expires [date]` · *(red expiry warning indicator if expiring soon)*
  - `Remaining` · `[sum of all batches minus usage] hr`
- Example display:
  ```
  2vCPU · 16 GB RAM
    Your usage          0.50 hr

    200.00 hr    Expires 2028-03-18
     50.00 hr  ⚠ Expires 2026-04-01
    ──────────────────────────────
    Remaining         249.50 hr
  ```
- No other members' usage shown
- **Per-VM ⓘ hint:**
  *"Team quota for this VM type. Shared across all members. Billed per minute (rounded up), shown in decimal hours. Current simulation always completes — new launches blocked at 0. Contact support to top up."*
- **Low credit inline warning (< 1% of total remaining):**
  *"Running low. Contact support to top up — top-up covers your deficit first, then adds to your balance."*

---

## Tasks

### Design [D]
- [ ] GEMU-1-D1 Design "Computation Credits" section — per VM: your usage, batch list (amount + expiry date + red expiry warning indicator), total remaining. Low credit and disabled states per VM.
  - Owner: @TODO
  - Due: TODO
  - Estimate: 3d

- [ ] GEMU-1-D2 Design simulation setup dropdown — 3 options with subtitles, disabled state for VM at ≤ 0 credit
  - Owner: @TODO
  - Due: TODO
  - Estimate: 2d
  - Depends on: GEMU-1-D1

- [ ] GEMU-1-D3 Design VM configuration modal — V1 shows single machine spec; structure to support multiple machine types in future
  - Owner: @TODO
  - Due: TODO
  - Estimate: 2d
  - Depends on: GEMU-1-D2

- [ ] GEMU-1-D4 Design credit states — low credit warning (< 1%), zero/negative state, disabled VM option
  - Owner: @TODO
  - Due: TODO
  - Estimate: 1d
  - Depends on: GEMU-1-D1

### Frontend [F]
- [ ] GEMU-1-F1 Implement "Computation Credits" section — per VM: your usage, batch list (amount + expiry + red warning if expiring soon), total remaining as sum of all batches minus usage. Dynamic VM rows from API.
  - Owner: @TODO
  - Due: TODO
  - Estimate: 3d
  - Depends on: GEMU-1-D1

- [ ] GEMU-1-F2 Implement simulation setup dropdown with 3 options and credit-aware disabled state for VM
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: GEMU-1-D2, GEMU-1-F1

- [ ] GEMU-1-F3 Implement VM configuration modal — single machine spec in V1, component structured for future multi-machine support
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: GEMU-1-D3, GEMU-1-F2

- [ ] GEMU-1-F4 Implement low credit warning and disabled state hints
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-D4, GEMU-1-F1

### Backend [B]
- [ ] GEMU-1-B1 API integration: Expose endpoint returning list of VM types, each with: top-up batch list (amount + expiry date), current user's usage (decimal hr), total remaining (sum of batches minus all member usage). FIFO consumption by earliest expiry handled in backend. Full per-member breakdown not exposed to client in V1.
  - Owner: @TODO
  - Estimate: 1d

- [ ] GEMU-1-B2 API integration: Expose endpoint to accept VM launch; confirm per-minute metering (rounded up) deducts from earliest-expiring batch first on simulation end.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-B1

- [ ] GEMU-1-B3 API integration: Confirm credit gate logic (per VM: block launch at ≤ 0, never interrupt in-flight) and top-up deficit-first behaviour are exposed correctly to frontend.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-B2

- [ ] GEMU-1-B4 Implement user-level feature flag — mark specific ODE user accounts as GEMU members in backend. API responses for VM options and Computation Credits only returned for flagged users.
  - Owner: @TODO
  - Estimate: 1d

### Test [T]
- [ ] GEMU-1-T1 Test credit display — batch list accuracy (amounts + expiry dates), expiry warning indicator, total remaining calculation, decimal formatting. Verify other members' data not exposed.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-F1, GEMU-1-B1

- [ ] GEMU-1-T2 Test credit gate scenarios — (1) credit runs out mid-run: simulation completes, balance goes negative; (2) balance ≤ 0 at launch: VM disabled, hint shown; (3) top-up while negative: deficit settled first; (4) two batches: verify earliest expiry consumed first
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: GEMU-1-F4, GEMU-1-B3

- [ ] GEMU-1-T3 Test feature flag — flagged user sees VM option and Computation Credits; non-flagged user on same ODE instance sees neither
  - Owner: @TODO
  - Estimate: 0.5d
  - Depends on: GEMU-1-B4, GEMU-1-F2

---

## Out of Scope

**V1 explicitly not included:**
- Admin panel for team quota management and per-member credit allocation (V2)
- Per-member credit limits or caps (V2)
- Multiple VM machine types in modal (V2 — modal component is pre-structured for this)
- Self-serve credit top-up / payment UI (support-only in V1)
- Credit usage history or simulation cost breakdown (V2)
- Email or in-app notifications for low balance (V2)

**Future considerations:**
- Admin panel: set team quota, allocate per-member limits, view usage reports (V2)
- Multi-machine modal: compare specs, select tier, see cost per second per machine (V2)
- Self-serve top-up with payment integration (V2)
- Per-simulation cost estimate before launch (V2)

## Success Metrics

- Zero in-flight simulation interruptions due to credit exhaustion
- Team members can locate credit balance and their own usage without guidance
- Low credit warning appears correctly at < 1% threshold
- Support contact rate after credit block: TODO (baseline after Early Access)

---

## Notes

- **GEMU contract: 200 hr total, expires 3 years from purchase date**
- VM provisioning, per-minute metering, and team/member account data model are already implemented in the backend — no new backend infrastructure required for this feature
- TODO: Confirm what "contact support" means in UI — email link, in-app chat, or other channel
- Backend should store per-VM, per-member usage from day one — client API only exposes current user's usage per VM in V1, full data ready for admin panel in V2
- Frontend component must render VM rows dynamically from API list — never hardcode VM types

## Changelog

- 2026-03-18: Created feature for GEMU
- 2026-03-18: Changed credit unit to hours, billed per minute rounded up (1 min 30 sec → 2 min → 0.033 hr)
- 2026-03-18: Removed per-member usage from UI — client shows per-VM team quota + your usage only
- 2026-03-18: Changed to per-VM independent quota model — each VM has its own team quota and credit gate; hours displayed as decimals
- 2026-03-18: Clarified platform context — ODE Platform; feature controlled by user-level flag for GEMU members
