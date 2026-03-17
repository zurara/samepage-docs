# ODE-X: VM Selection & Credit Management

**Status**: `Ideation`
**Owner**: @TODO
**Priority**: `P1`
**Target**: TODO
**Progress**: 0/12 tasks ✓

---

## Why

Engineers running simulations need control over compute resources and cost visibility — but a simulation that terminates mid-run due to credit exhaustion is far worse than no simulation at all. This feature gives clients transparent credit tracking and VM choice, while guaranteeing every simulation task runs to completion.

## Who

ODE clients (teams and individual engineers) who run server-based simulations. Both the engineer initiating the simulation and the team admin managing shared credit are key actors.

## What

**Core Capabilities:**
- Client can see their personal credit balance and total team credit balance in the dashboard at a glance
- Client can choose which VM tier to use before launching a simulation
- A running simulation is never interrupted when credit runs out — it always completes
- When credit balance goes below 0, the client sees a clear prompt to contact support for a top-up

**User Flow:**
View credit balances in dashboard → Select VM tier before launching simulation → Simulation runs to completion regardless of credit state → If balance < 0 after run, contact support to restore credit

## Key Terms

- **Personal Credit**: Credit allocated to an individual user account
- **Team Credit**: Shared credit pool available to all members of a team — consumed as fallback when personal credit is exhausted
- **VM Tier**: A virtual machine configuration option (e.g., Standard, High-Performance) with different compute specs and credit cost rates
- **Credit Balance**: Current available credit; can go negative after a simulation completes
- **Top-up**: Adding credit to an account — handled exclusively by support in V1, not self-serve

## Decisions

**Credit consumption order:**
- **Personal credit depletes first, team credit is the fallback**: When a simulation runs, personal credit is consumed first. Team credit is only drawn when personal credit reaches 0. This gives individuals autonomy while allowing team credit as a safety net.


- **Never interrupt a running simulation**: A simulation that starts will always run to completion, even if credit hits 0 or goes negative mid-run. A failed mid-run result is worse than a predictable "finish then notify" approach. Engineers cannot restart a complex simulation from scratch without significant time loss.

**Negative credit handling:**
- **Contact support, not self-serve top-up**: In V1, credit top-up is not self-service. When balance goes below 0, the UI prompts the user to contact support. This keeps billing logic simple for launch and avoids payment integration scope.

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

- [ ] ODE-X-D2 Design VM selection UI — pre-launch step showing available tiers, cost per run estimate, current credit balance
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

- [ ] ODE-X-B2 API: Accept VM tier selection at simulation launch; record chosen tier and estimated credit cost
  - Owner: @TODO
  - Estimate: 2d
  - Depends on: ODE-X-B1

- [ ] ODE-X-B3 Implement simulation completion guarantee — simulation task must not be terminated due to credit exhaustion; deduct credit post-completion (balance may go negative)
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

- [ ] ODE-X-T2 Test simulation completion guarantee — simulate credit running out mid-run, verify job completes and balance goes negative correctly
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
- Simulation abandonment rate due to credit issues: 0% (hard requirement — no simulation interrupted by credit)
- Credit balance visibility: Users can see both balances without navigating away from dashboard
- Support contact rate post negative-credit: TODO (baseline to establish)

---

## Notes
TODO: Define VM tiers available at launch (names, specs, credit cost per hour or per run)
- TODO: Define what "contact support" means in the UI — email link, in-app chat, WeChat for CN users?

## Changelog
- 2026-03-17: Created feature
- 2026-03-17: Confirmed credit consumption order — personal first, team as fallback
