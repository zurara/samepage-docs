# ODE-X: VM Selection & Credit Management

**Status**: `Ideation`
**Owner**: @TODO
**Priority**: `P1`
**Target**: TODO
**Progress**: 0/12 tasks ✓

---

## Why

Engineers running simulations need control over compute resources and cost visibility. This feature gives whitelisted clients transparent credit tracking and VM choice, with a simple credit gate: credit > 0 allows VM simulation launch; credit ≤ 0 blocks it. In-flight simulations are never interrupted. Launching as an Early Access feature for a selected user group before wider rollout.

## Who

ODE clients (teams and individual engineers) who run server-based simulations. Both the engineer initiating the simulation and the team admin managing shared credit are key actors.

## What

**Core Capabilities:**
- Client sees personal credit and shared team credit in the Home / Payment page under a new "Computation Credits" section
- Client selects simulation environment from a dropdown in the setup panel: Local (Free), Cloud (Subscription), or Virtual Machine (Top-up Credits)
- Credit > 0: VM simulation launch is allowed. Credit ≤ 0: VM option is disabled with a hint to contact support
- In-flight simulations are never interrupted — the credit block applies to new launches only
- When credit goes negative mid-run, the next top-up covers the deficit first before adding usable balance (metro card model)
- All VM credit UI is labeled as Early Access — visible to whitelisted users only

**User Flow:**
View credit balances (Home → Computation Credits) → Select "Virtual Machine" in simulation setup dropdown → Launch blocked if credit ≤ 0 → Credit deducted per second → If balance hits 0 mid-run, current simulation completes → Next launch blocked until top-up via support

## Key Terms

- **Personal Credit**: Credit allocated to an individual user account
- **Team Credit**: Shared credit pool available to all members of a team — consumed as fallback when personal credit is exhausted
- **VM Tier**: One of two machine options — Free Machine (no credit cost) or Paid Machine (credit deducted per second of runtime)
- **Free Machine**: Standard compute, available to all users, consumes no credit
- **Paid Machine**: Higher compute, credit is deducted per second of active simulation runtime
- **Credit Balance**: Current available credit; can go negative after a simulation completes
- **Top-up**: Adding credit to an account — handled exclusively by support in V1, not self-serve

## Decisions

**Early Access / whitelist:**
- **Label all VM credit UI as Early Access**: The feature is only visible to whitelisted users. All hover hints end with *Early Access.* tag. This sets expectations and explains why other users may not see it.

**Copywriting — Simulation Setup Dropdown:**
- Label: `Run on`
- Options and subtitles:
  - `Run in local browser` · `Private · Free · Best for small models`
  - `Run on cloud server` · `1vCPU · 8 GB RAM · Subscription`
  - `Run on virtual machine` `Early Access` (badge on title) · `2vCPU · 16 GB RAM · Top-up Credits`
  - `Run on virtual machine` (disabled) · `2vCPU · 16 GB RAM · Out of Credits`
- Hover hint on disabled VM: *"No credits remaining. Contact support to top up."*

**Copywriting — Computation Credits Section (Home / Payment):**
- Section title: `Computation Credits`
- Row labels: `Your remaining credits` and `Shared remaining credits`
- Balance format: `–00:12:12 / 12:30:59` (negative shown in red)
- Label placement: small `Remaining · Total` label below the numbers, right-aligned

- **Your remaining credits ⓘ hint:**
  *"Deducted per second on VM runs. Personal credits go first, then shared. Current simulation always completes — new launches blocked at 0. Contact support to top up."*

- **Shared remaining credits ⓘ hint:**
  *"Team credit pool. Used automatically when your personal credits run out. Contact support to top up."*

- **Low credit inline warning (< 1% of total remaining)** — shown under either row, reusable for both:
  *"Running low. Contact support to top up — top-up covers your deficit first, then adds to your balance."*

**VM tiers:**
- **Two tiers only in V1 — Free and Paid**: Free Machine has no cost and suits standard simulations. Paid Machine is billed per second of runtime for higher compute needs. No intermediate tiers in V1; keeps selection simple and billing logic straightforward.

**Credit consumption order:**
- **Personal credit depletes first, team credit is the fallback**: When a simulation runs, personal credit is consumed first. Team credit is only drawn when personal credit reaches 0. This gives individuals autonomy while allowing team credit as a safety net.


**Simulation completion guarantee:**
- **In-flight runs are never interrupted**: A simulation that has already started always runs to completion, even if credit hits 0 or goes negative mid-run. Engineers cannot afford to restart a complex simulation from scratch.

**Credit gate — metro card model:**
- **Credit > 0: new VM simulation launch allowed. Credit ≤ 0: new launch blocked.** The block only applies to initiating a new run — never to an active one.
- **Top-up covers deficit first**: When support adds credit, the negative balance is settled first, then the remainder becomes usable credit. e.g. balance = –30, top-up = 100 → new balance = 70.

**Negative credit handling:**
- **Contact support via email, not self-serve top-up**: In V1, credit top-up is not self-service. When balance ≤ 0, the Paid Machine / Cloud option is disabled in setup and a hint directs the user to email support.

**Credit display:**
- **Show both personal and team credit in dashboard**: Engineers need visibility into both pools to make informed decisions about VM tier selection. Display both, clearly labeled.

---

## Dependencies
- Requires: ODE-0-B1 (Server-based simulation infrastructure — VM provisioning API)
- Requires: ODE-0-B2 (User account & team account data model)

---

## Tasks

### Design [D]
- [ ] ODE-X-D1 Design credit display component for dashboard (personal credit + team credit, both visible at a glance)
  - Owner: @TODO
  - Due: TODO
  - Estimate: 2d

- [ ] ODE-X-D2 Design VM selection UI — pre-launch step showing Free Machine vs Paid Machine, credit cost rate (per second) for paid option, current credit balance
  - Owner: @TODO
  - Due: TODO
  - Estimate: 3d
  - Depends on: ODE-X-D1

- [ ] ODE-X-D3 Design low/negative credit states — in-dashboard warning banner + post-simulation notification with support contact CTA
  - Owner: @TODO
  - Due: TODO
  - Estimate: 2d
  - Depends on: ODE-X-D2

### Frontend [F]
- [ ] ODE-X-F1 Implement credit balance display in dashboard (personal + team, real-time or on-load fetch)
  - Owner: @TODO
  - Due: TODO
  - Estimate: 2d
  - Depends on: ODE-X-D1

- [ ] ODE-X-F2 Implement VM tier selection step in simulation launch flow
  - Owner: @TODO
  - Estimate: 3d
  - Depends on: ODE-X-D2, ODE-X-F1

- [ ] ODE-X-F3 Implement negative credit warning states and support contact prompt
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: ODE-X-D3, ODE-X-F1

### Backend [B]
- [ ] ODE-X-B1 API: Return personal credit + team credit balances for authenticated user
  - Owner: @TODO
  - Estimate: 2d

- [ ] ODE-X-B2 API: Accept VM tier selection at simulation launch; for Paid Machine, begin per-second credit metering on simulation start
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: ODE-X-B1

- [ ] ODE-X-B3 Implement dual credit rules — (1) active simulation always runs to completion regardless of balance; (2) new simulation launch blocked if balance ≤ 0; top-up settles deficit before restoring usable balance
  - Owner: @TODO
  - Estimate: 3d
  - Depends on: ODE-X-B2

- [ ] ODE-X-B4 Trigger notification/flag when credit balance drops below 0 after simulation completes
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: ODE-X-B3

### Test [T]
- [ ] ODE-X-T1 Test credit display accuracy — personal vs team balance, real-time refresh behavior
  - Owner: @TODO
  - Estimate: 1d
  - Depends on: ODE-X-F1, ODE-X-B1

- [ ] ODE-X-T2 Test credit rules — (1) credit runs out mid-run: verify simulation completes and balance goes negative; (2) balance ≤ 0 at launch: verify VM option is disabled and hint shown; (3) top-up while negative: verify deficit settled before credit is usable
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: ODE-X-F3, ODE-X-B4

---

## Out of Scope

**V1 Explicitly Not Included:**
- Self-serve credit top-up / payment UI (handled by support only — payment integration is out of scope)
- Credit reservation or pre-locking before simulation starts (would require complex rollback logic)
- Per-simulation credit cost estimation shown to user before launch (nice to have, V2)
- Credit alerts or email notifications for low balance (V2)
- Admin controls for setting credit limits per team member (V2)

**Future Considerations:**
- Self-serve top-up with payment integration (V2)
- Credit usage history and simulation cost breakdown (V2)
- Pre-run cost estimation based on model complexity + VM tier (V2)

## Success Metrics
- Zero in-flight simulation interruptions due to credit exhaustion
- Whitelisted users can locate Computation Credits section without guidance
- Support contact rate post credit block: TODO (baseline to establish after Early Access)

---

## Notes
- TODO: Define VM tiers available at launch (names, specs, credit cost per second for Paid Machine)
- Support contact method: email

## Changelog
- 2026-03-17: Created feature
- 2026-03-17: Confirmed credit consumption order — personal first, team as fallback
- 2026-03-18: Confirmed support contact method — email
- 2026-03-17: Confirmed dual credit rules — in-flight always completes; new launch blocked at ≤ 0; top-up covers deficit first
- 2026-03-18: Finalised all copywriting — dropdown subtitles, ⓘ hints, low credit warning, disabled state
