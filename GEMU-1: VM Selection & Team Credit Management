# GEMU-1: VM Selection & Team Credit Management

> Part of [VM-1: Virtual Machine Simulation — Full Feature](./VM-1-virtual-machine-simulation-full.md) · **Phase 1 — Early Access**

**Status**: `Ideation`
**Owner**: @TODO
**Priority**: `P1`
**Target**: TODO
**Progress**: 0/13 tasks ✓


## Why

Engineers need to choose the right compute environment for their simulation and have clear visibility of their own usage within the team quota. This feature introduces a simple credit display (team quota + your usage) and a VM selection flow designed to scale as more machine types become available. Full team usage breakdown is managed in the backend admin panel, not exposed in the UI.

## Who

Team members running simulations on the GEMU platform. Each member sees only their own usage and the shared team quota — not other members' usage. Full breakdown is admin-only.

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
- **Team Quota (per VM)**: Total hours allocated to the team for a specific VM type by admin. Each VM type has its own quota.
- **Your Usage (per VM)**: How much credit (in decimal hours) the current user has consumed from a specific VM's team quota.
- **Team Remaining (per VM)**: That VM's team quota minus all member usage. This is the operative balance for the credit gate of that VM.
- **VM Modal**: A configuration dialog that opens when "Run on virtual machine" is selected. Currently shows one machine spec; designed to support multiple machine types in future.
- **Credit Gate (per VM)**: Independent per VM type. Credit > 0 allows launch on that VM. Credit ≤ 0 blocks new launches on that VM only.
- **Credit Metering Unit**: Billed per minute, expressed as decimal hours. Runtime rounded up to nearest minute — e.g. 1 min 30 sec → 2 min → 0.033 hr deducted.
- **Top-up**: Adding hours to a specific VM's team quota — handled by support in V1. Deficit settled first before new balance is usable.
- **Early Access**: Feature is in limited rollout for whitelisted users. Indicated by icon, not text labels in hints.

---

## Decisions

**Team-only credit model — no per-member visibility in UI:**
- There is one shared team pool per VM type. Members only see team quota and their own usage — not other members'. Full usage breakdown is backend admin only. This keeps the UI simple and avoids privacy concerns between team members. Per-member quota allocation is a future admin panel feature.

**Per-VM independent quota:**
- Each VM type has its own team quota. Credit gate is evaluated independently per VM — one VM being out of credits does not affect other VMs. This allows teams to have different budgets for different compute tiers. Backend must store quota and usage indexed by VM type, not as a single pool.

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
- One row per VM type, each row shows:
  - VM spec label (e.g. `2vCPU · 16 GB RAM`)
  - `Your usage` · `[decimal] hr`
  - `Team quota` · `[decimal] hr`
- Example display:
  ```
  2vCPU · 16 GB RAM    Your usage  0.50 hr    Team quota  120.00 hr
  4vCPU · 32 GB RAM    Your usage  1.03 hr    Team quota   50.00 hr
  ```
- No other members' usage shown
- **Per-VM row ⓘ hint:**
  *"Team quota for this VM type. Shared across all members. Billed per minute (rounded up), shown in decimal hours. Current simulation always completes — new launches blocked at 0. Contact support to top up."*
- **Low credit inline warning (< 1% of that VM's team quota remaining)** — shown under the affected VM row:
  *"Running low. Contact support to top up — top-up covers your deficit first, then adds to your balance."*

---

## Tasks

### Design [D]
- [ ] GEMU-1-D1 Design "Computation Credits" section — one row per VM type, each showing VM spec label, your usage (decimal hr), team quota (decimal hr). Low credit and disabled states per row.
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
- [ ] GEMU-1-F1 Implement "Computation Credits" section — render one row per VM type from API response. Each row: VM spec label, your usage, team quota in decimal hr. Component must handle variable number of VM rows.
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
- [ ] GEMU-1-B1 API integration: Expose endpoint returning list of VM types with team quota (decimal hr) and current user's usage (decimal hr) per VM. Full per-member breakdown not exposed to client in V1.
  - Owner: @TODO
  - Estimate: 1d

- [ ] GEMU-1-B2 API integration: Expose endpoint to accept VM launch with selected machine config; confirm per-minute metering (rounded up) deducts from correct VM quota on simulation end.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-B1

- [ ] GEMU-1-B3 API integration: Confirm credit gate logic (per VM: block launch at ≤ 0, never interrupt in-flight) and top-up deficit-first behaviour are exposed correctly to frontend.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-B2

### Test [T]
- [ ] GEMU-1-T1 Test credit display — accuracy of per-VM quota and usage rows, decimal formatting, low credit threshold per VM. Verify other members' data not exposed.
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: GEMU-1-F1, GEMU-1-B1

- [ ] GEMU-1-T2 Test credit gate scenarios — (1) credit runs out mid-run: simulation completes, balance goes negative; (2) balance ≤ 0 at launch: VM disabled, hint shown; (3) top-up while negative: deficit settled first
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: GEMU-1-F4, GEMU-1-B3

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

- VM provisioning, per-minute metering, and team/member account data model are already implemented in the backend — no new backend infrastructure required for this feature
- TODO: Confirm what "contact support" means in UI — email link, in-app chat, or other channel
- Backend should store per-VM, per-member usage from day one — client API only exposes current user's usage per VM in V1, full data ready for admin panel in V2
- Frontend component must render VM rows dynamically from API list — never hardcode VM types

## Changelog

- 2026-03-18: Created feature for GEMU
- 2026-03-18: Changed credit unit to hours, billed per minute rounded up (1 min 30 sec → 2 min → 0.033 hr)
- 2026-03-18: Removed per-member usage from UI — client shows per-VM team quota + your usage only
- 2026-03-18: Changed to per-VM independent quota model — each VM has its own team quota and credit gate; hours displayed as decimals
