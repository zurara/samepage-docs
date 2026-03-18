# VM-1: Virtual Machine Simulation — Full Feature

**Status**: `In Progress`
**Owner**: @TODO
**Priority**: `P0`
**Target**: TODO
**Progress**: 0/0 tasks ✓ *(see phases)*


## Why

Engineering teams need flexible, scalable compute options for simulation workloads. A VM-based simulation layer gives teams dedicated compute power beyond what local browser or shared cloud servers can offer, with transparent credit management, admin-controlled quota allocation, and usage visibility at both member and team level. This feature is designed to scale from small early-access teams to large enterprise deployments.

## Who

- **Team members**: Engineers using the ODE Platform whose accounts are flagged for VM access
- **Team admins**: Manage quota allocation per VM type, monitor team-wide usage, top up credits
- **Platform support**: Handle user flagging, credit top-ups, and account management in early phases

## What

**Full System Capabilities:**
- Engineers choose simulation environment from a dropdown: Local, Cloud, or Virtual Machine
- VM selection opens a modal — supports single or multiple machine types with spec comparison
- Each VM type has its own independent team quota (in decimal hours), billed per minute rounded up
- Credit gate per VM: > 0 allows launch, ≤ 0 blocks new launches; in-flight simulations always complete
- Top-up covers deficit first, then adds to usable balance (metro card model)
- Members see their own usage and team quota per VM — no other members' data exposed in member view
- Admins see full per-member usage breakdown per VM type in an admin panel
- Admins can set, adjust, and allocate quota per VM type per team or per member
- Self-serve top-up via payment integration (later phase)
- Feature visible to whitelisted users in early phases, general availability in later phases

**System Architecture:**
```
Cluster Computing Epic (upstream infrastructure)
└── VM-1 (client-facing layer built on top)
    ├── Phase 1 — GEMU-1 (Early Access, current)
    │   ├── Member: VM dropdown + modal (single VM)
    │   ├── Member: Computation Credits display (per-VM quota + your usage)
    │   └── Backend: API integration with existing metering infrastructure
    │
    ├── Phase 2 — Multi-VM + Admin Panel (next)
    │   ├── Admin panel: quota management per VM per team
    │   ├── Admin panel: full per-member usage breakdown
    │   ├── Multi-machine modal: spec comparison, tier selection
    │   └── Usage history and per-simulation cost breakdown
    │
    └── Phase 3 — Self-serve & Scale
        ├── Self-serve top-up with payment integration
        ├── Per-simulation cost estimate before launch
        ├── Credit alerts and notifications
        └── General availability (remove whitelist)
```

**Relationship to Cluster Computing Epic:**

The Cluster Computing Epic defines the underlying distributed infrastructure — FMU task generation, resource allocation, task scheduling and distribution, and result aggregation. VM-1 is the client-facing product layer built on top of that infrastructure. Specifically:

| Cluster Epic | VM-1 Dependency |
|---|---|
| Story 1.x — FMU task packaging & configuration | VM-1 assumes FMUs can be packaged and dispatched to a node |
| Story 2.x — Resource inventory & allocation policies | VM-1 credit gate determines whether a node is requested; allocation policy is backend concern |
| Story 2.3 — Scaling capabilities | VM-1 Phase 3 self-serve and GA depends on elastic resource availability |
| Story 3.x — Task scheduling & execution management | VM-1 shows simulation status; execution lifecycle managed by cluster layer |
| Story 4.1–4.3 — Result collection & visualisation | VM-1 does not own result handling — results surface in the app that launched the simulation |
| Story 4.4 — Failure recovery | In-flight completion guarantee in VM-1 depends on cluster-level retry and recovery |

## Key Terms

- **VM type**: A specific machine configuration (e.g. 2vCPU · 16 GB RAM). Each VM type has its own independent quota.
- **Team quota (per VM)**: Total decimal hours allocated to the team for a specific VM type.
- **Your usage (per VM)**: Decimal hours consumed by the current user from a specific VM's quota.
- **Team remaining (per VM)**: Team quota minus total member usage for that VM. Operative balance for credit gate.
- **Credit gate**: Per VM — credit > 0 allows new launch; credit ≤ 0 blocks new launches on that VM only.
- **Credit metering**: Precise runtime recorded by backend. Expressed as decimal hours in UI. No product-level rounding.
- **Top-up**: Adding hours to a VM's team quota. Deficit settled first. Handled by support (Phase 1–2), self-serve (Phase 3).
- **Admin panel**: Interface for team admins to manage quota, view full usage, and configure VM access per member.
- **Early Access**: Feature only visible to ODE users whose accounts are flagged as members of a participating team. Non-flagged users on the same ODE instance see no VM option and no Computation Credits section. Indicated by icon on visible UI elements. Removed in Phase 3 when feature becomes generally available.
- **User flag**: A boolean field on an ODE user account that enables VM feature access. Set by support or backend admin. Not a role or permission system.

---

## Decisions

**Feature visibility — user-level flag on ODE Platform:**
- VM feature is rendered only for ODE users with the flag enabled. Controlled in backend. Non-flagged users on the same platform see no difference. In Phase 3, the flag is removed and the feature becomes available to all ODE users.

**Per-VM independent quota:**
- Each VM type has its own quota pool. Credit gate is evaluated independently per VM. One VM being exhausted does not affect other VMs. Backend stores quota and usage indexed by VM type.

**Hours as decimals:**
- All credit values displayed as decimal hours (e.g. 1.50 hr). No HH:MM:SS. Consistent with how quotas are set and topped up.

**Credit gate — metro card model:**
- In-flight simulations always complete. New launches blocked at ≤ 0. Top-up settles deficit first. e.g. balance = –0.5 hr, top-up = 10 hr → new balance = 9.5 hr.

**VM selection via modal, not inline:**
- VM option in dropdown opens a modal. Keeps dropdown clean and allows future multi-machine spec comparison without redesigning the dropdown.

**Member view shows own usage only:**
- Members see per-VM team quota and their own usage only. No other members' data exposed. Full breakdown is admin-only. Prevents privacy concerns between team members.

**Admin panel deferred to Phase 2:**
- Phase 1 (GEMU-1) has no admin UI. Quota management is done directly in backend by support. Admin panel ships in Phase 2 when team size and usage patterns are better understood.

**Self-serve top-up deferred to Phase 3:**
- Payment integration adds significant scope and compliance overhead. Support-managed top-up is sufficient for early access. Self-serve ships in Phase 3.

---

## Phases

### Phase 1 — GEMU-1 (Early Access) ✦ Current

**Scope:** Member-facing UI only. Single VM type. API integration with existing backend.

See full task breakdown in [GEMU-1](./GEMU-1-vm-selection-team-credit.md).

**Delivers:**
- VM dropdown in simulation setup (3 options: Local, Cloud, VM)
- VM configuration modal (single machine spec)
- Computation Credits section (per-VM: team quota + your usage in decimal hr)
- Low credit warning (< 1% threshold)
- Disabled state when credit ≤ 0
- API integration with existing metering and quota backend

**Does not include:** Admin panel, multi-VM modal, self-serve top-up, usage history

---

### Phase 2 — Multi-VM + Admin Panel

**Scope:** Expand VM modal to support multiple machine types. Add admin panel for quota management and usage visibility.

**Delivers:**
- Multi-machine modal: list available VM types, show specs and credit rate, allow selection
- Admin panel — quota management: set and adjust team quota per VM type
- Admin panel — member usage: full per-member usage breakdown per VM type
- Admin panel — allocation: optionally cap usage per member per VM (future)
- Usage history: per-simulation cost breakdown for members and admins
- Low balance email / in-app notification

**Key decisions for Phase 2:**
- TODO: Define admin role — is it a separate account type or a permission on existing accounts?
- TODO: Define notification thresholds (10%? 5%? configurable?)
- ⚠️ **Discussion needed: Unified credit system** — as VM types increase, per-VM hour quotas with different rates become hard to manage. Consider introducing a unified credit currency that converts to runtime per VM based on each type's rate. Top-ups would add to a shared credit pool rather than VM-specific hour quotas. Resolve before Phase 2 design starts.

---

### Phase 3 — Self-serve & General Availability

**Scope:** Remove whitelist, add self-serve top-up, launch publicly.

**Delivers:**
- Self-serve credit top-up with payment integration
- Per-simulation cost estimate shown before launch
- General availability — remove Early Access whitelist and icon
- Per-member quota caps and allocation controls (admin)
- Credit usage export / reporting

---

## Out of Scope (entire VM-1)

- Hardware provisioning or data centre management — VM infrastructure is pre-existing
- SLA guarantees on VM uptime or simulation accuracy
- Custom VM spec requests per team (not in roadmap)

---

## Success Metrics

**Phase 1:**
- Zero in-flight simulation interruptions due to credit exhaustion
- Whitelisted members locate Computation Credits without guidance
- Support contact rate after credit block: TODO (baseline after Early Access)

**Phase 2:**
- Admin quota setup completed without support assistance
- Reduction in support tickets related to credit management

**Phase 3:**
- Self-serve top-up conversion rate: TODO
- Time-to-top-up (self-serve vs support): target < 5 min

---

## Notes

- **GEMU contract (Phase 1): 200 hr total, expires 3 years from purchase date**
- **Expiry warning threshold: 30 days before expiry date**
- TODO: Confirm credit rate per hour per VM type
- TODO: Confirm support contact method for top-up (email, in-app, other)
- TODO: Define admin role model before Phase 2 starts
- Backend metering, credit gate logic, and team/member data model are already implemented — Phase 1 only requires API integration
- Cluster Computing Epic is upstream infrastructure — VM-1 is the client-facing layer; coordinate with cluster epic owners before Phase 2 and Phase 3

## Changelog

- 2026-03-18: Created VM-1 master PRD; GEMU-1 defined as Phase 1
- 2026-03-18: Added GEMU contract details — 200 hr total, 3-year expiry
- 2026-03-18: Added Cluster Computing Epic as upstream dependency; mapped epic stories to VM-1 phases
- 2026-03-18: Confirmed expiry warning threshold — 30 days
- 2026-03-18: Clarified platform — ODE Platform; Early Access controlled by user-level flag per account
