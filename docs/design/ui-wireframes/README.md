# GaranTech — UI Wireframes

**Task:** Low-Fidelity UI Wireframes  
**Assignee:** Arash Farhadi-Rad  
**Status:** Draft for team review  
**Branch:** `docs/ui-wireframes`  
**Basis:** SRS v0.3, Use Cases (UC-01…UC-12), Activity Diagram, Logical Data Model, ERD, Architecture docs, `docs/prototype/`

---

## 1. Purpose

Define page structure, element placement, navigation paths, and UI feedback for every GaranTech screen **before implementation**. These are low-fidelity wireframes: grayscale boxes and labels only — no final colors, icons, animation, or visual polish.

Authority rule: **SRS v0.3 is the source of truth.** Where the task brief and the SRS disagree, the SRS wins and the difference is logged in [consistency-review.md](consistency-review.md).

## 2. Tool used

| Item | Decision |
|---|---|
| Authoring | SVG 1.1, plain text, version-controlled |
| Preview | [index.html](index.html) — browsable gallery of all 16 frames |
| Figma source file | **Not provided.** No Figma account/file was available. SVGs are vector and can be imported into Figma; no Figma URL is claimed. |

Figma link: **N/A — substitute deliverable is the SVG set + gallery.** Confirm with the project manager if an actual `.fig` file is required.

## 3. Pages designed

### Customer (mobile-width) — no account, no login

| ID | Page | Wireframe | Main states |
|---|---|---|---|
| P01 | Home | [01-home.svg](images/customer/01-home.svg) | Initial |
| P02 | Submit Repair Request | [02-submit-request.svg](images/customer/02-submit-request.svg) | Validation, out-of-scope, success, loading |
| P03 | Track Request | [03-track-request.svg](images/customer/03-track-request.svg) | Empty, not-found, loading, success |
| P04 | Request Details | [04-request-details.svg](images/customer/04-request-details.svg) | Conditional actions, BR-06 allowlist |
| P05 | Customer Decision | [05-customer-decision.svg](images/customer/05-customer-decision.svg) | Confirmation, rejection, loading |
| P06 | Cancel Request | [06-cancel-request.svg](images/customer/06-cancel-request.svg) | Confirmation, blocked, success |

### Staff (desktop-width) — role-gated

| ID | Page | Wireframe | Role | Main states |
|---|---|---|---|---|
| P07 | Staff Login | [07-staff-login.svg](images/staff/07-staff-login.svg) | All staff | Validation, failed login |
| P08 | Staff Dashboard | [08-staff-dashboard.svg](images/staff/08-staff-dashboard.svg) | Role-specific | Empty, loading, role filtering |
| P09 | Repair Requests List | [09-requests-list.svg](images/staff/09-requests-list.svg) | All staff | Search, filter, empty |
| P10 | Staff Request Details | [10-staff-request-details.svg](images/staff/10-staff-request-details.svg) | All staff | Full internal view, role-gated actions |
| P11 | Device Reception | [11-device-reception.svg](images/staff/11-device-reception.svg) | Service Expert | Mismatch path, blocked before save |
| P12 | Inspection & Warranty | [12-inspection-warranty.svg](images/staff/12-inspection-warranty.svg) | Service Expert | 1..N defects, two warranty paths |
| P13 | Repair Proposal | [13-repair-proposal.svg](images/staff/13-repair-proposal.svg) | Service Expert | Blocked, amount validation, preview |
| P14 | Repair Management | [14-repair-management.svg](images/staff/14-repair-management.svg) | Technician | Start, operations, validation |
| P15 | Repair Testing | [15-repair-testing.svg](images/staff/15-repair-testing.svg) | Technician | PASSED / FAILED, repeated tests |
| P16 | Staff Management | [16-staff-management.svg](images/staff/16-staff-management.svg) | Admin | Duplicate username, activate/deactivate |

## 4. Roles covered

| Role | Pages | Notes |
|---|---|---|
| Customer | P01–P06 | No account. Verification = tracking code **+** registered phone. |
| Service Expert | P07–P10, P11, P12, P13 | Receives device, inspects, sets warranty manually, proposes cost. |
| Technician | P07–P10, P14, P15 | Starts repair, records operations and test results. |
| Admin | P07–P10, P16 | Monitors requests (list + filter + details), manages staff accounts. |

No shared dashboard with identical capabilities across roles. Backend enforces all permissions; UI hiding is not authorization.

## 5. Main flows

| Flow | Path | Detail |
|---|---|---|
| 1 — Submit request | P01 → P02 → tracking code → P03 | [customer-flows.md](customer-flows.md#flow-1--submit-request) |
| 2 — In-warranty repair | P02 → P11 → P12 (ELIGIBLE) → P14 → P15 (PASSED) → P04 | [staff-flows.md](staff-flows.md#flow-2--in-warranty-repair) |
| 3 — Out-of-warranty repair | P02 → P11 → P12 (NOT_ELIGIBLE) → P13 → P05 → P14 → P15 → P04 | both docs |
| 4 — Failed test and retry | P14 (IN_REPAIR) → P15 (FAILED) → P14 → P15 (PASSED) | [staff-flows.md](staff-flows.md#flow-4--failed-test-and-retry) |
| 5 — Track and cancel | P03 → P04 → P06 (if allowed) | [customer-flows.md](customer-flows.md#flow-5--track-and-cancel) |

Rejecting the proposal ends at `CANCELLED` and never enters repair.

## 6. Deliberate exclusions

Customer signup/login, online payment, parts inventory, external warranty services, SMS/email integrations, advanced analytics, accounting. Inline success/error messages are UI-only.

## 7. Review

See [consistency-review.md](consistency-review.md) for: conflict flags against the task brief, SRS/UC/Activity-Diagram traceability, role matrix, UI-state coverage, and the final acceptance checklist.
