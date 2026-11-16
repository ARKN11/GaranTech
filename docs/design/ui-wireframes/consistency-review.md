# Wireframe Consistency Review

**Date:** 2026-11-16  
**Scope:** 16 wireframes vs. task brief, SRS v0.3, Use Cases, Activity Diagram, data model, architecture docs, and `docs/design/consistency-review.md`.  
**Authority:** SRS v0.3 and existing design docs override the task brief where they conflict. The task brief itself requires this ("design based on existing requirements; do not add undefined features without PM coordination").

---

## 1. Resolved conflicts

### CR-01 — Customer detail scope

| | |
|---|---|
| **Task brief** | Page 4 asks for device specs, initial problem description, and **status change history** in the customer view. |
| **Project docs** | BR-06: customer sees only current status, inspection result, proposed cost. `CustomerTrackingView` DTO allowlist: trackingCode, currentStatus, warrantyResult, recommendedAction, diagnosedDefects, estimatedTotalCost. Prior review CR-09 says device info must be removed from the customer view. |
| **Decision** | Wireframe P04 follows the SRS: allowlisted fields only. Full history, device specs, repair operations, and internal notes are excluded. |
| **PM action** | Confirm whether BR-06 should be relaxed. If yes, update SRS, UC-09, `CustomerTrackingView`, and the API contract first — then P04 can be extended. Do not add it to the UI alone. |

### CR-02 — Dashboard counters vs. "no analytics"

| | |
|---|---|
| **Task brief** | Page 8 suggests counters for new, awaiting reception, awaiting inspection, ready, in repair, completed. Section 8 also forbids advanced statistical reports. Section 7 says avoid out-of-scope capabilities. |
| **Project docs** | SRS §7 excludes "admin statistical reports"; FR-15 permits a status-filtered list and detail view only. |
| **Decision** | P08 shows four operational queue counters (awaiting reception, awaiting inspection, ready, in repair) as navigation aids. No charts, trends, KPIs, or historical reporting. "Completed" is terminal and is not a work queue. |
| **PM action** | Confirm the four-counter interpretation is acceptable. |

### CR-03 — Figma source file and link

| | |
|---|---|
| **Task brief** | Expected output: a Figma (or similar tool) source file and a Figma link in README. |
| **Reality** | No Figma account or `.fig` file available in this workspace. |
| **Decision** | Delivered plain-text SVG wireframes (vector, Figma-importable) plus an HTML gallery. README states `Figma link: N/A` instead of inventing a URL. |
| **PM action** | If a `.fig` file is mandatory, someone with Figma access must import the SVGs and add the real link. |

### CR-04 — "View each employee" on P16

Task page 16 asks to view each employee's information. P16 shows a staff table with name, phone, username, role, and status, plus change-role and activate/deactivate actions. No password or hash is displayed. This satisfies FR-15 without adding account features outside scope.

---

## 2. Consistency checks passed

| Check | Result |
|---|---|
| All 16 task pages have a wireframe (task pages 1–16 → P01–P16) | Pass |
| Customer has no signup/login surface | Pass (P01–P06) |
| Customer verification uses tracking code **and** phone | Pass (P03; ADR-006) |
| Customer view contains no staff/internal/history data | Pass (P04, P05; BR-06, CR-09) |
| Warranty status entered manually by Service Expert | Pass (P12; BR-09) |
| ELIGIBLE path skips proposal and customer decision | Pass (P12 → P14) |
| NOT_ELIGIBLE path creates one proposal before repair | Pass (P12 → P13 → P05 → P14) |
| `NOT_ELIGIBLE` requires rejection reason | Pass (P12; IC-07) |
| One or more defects and one or more operations | Pass (P12, P14; IC-08, IC-14) |
| Proposal amount numeric and greater than zero, IRR | Pass (P13; IC-15) |
| Repair starts only from READY_FOR_REPAIR | Pass (P14; IC-12) |
| Customer cancel blocked at/after IN_REPAIR | Pass (P04, P06; BR-07) |
| FAILED keeps IN_REPAIR; PASSED sets COMPLETED | Pass (P15; BR-11, BR-12, IC-18, IC-19) |
| Repeated tests preserved | Pass (P15) |
| Role-specific workspaces, no shared dashboard | Pass (P08 + role matrix) |
| Backend authorization explicitly required | Pass (README, staff-flows, P10, P16; ADR-008) |
| Loading / empty / error / validation / success / confirmation states | Pass — annotated on every frame |
| Out-of-scope features absent | Pass — no customer auth, payment, inventory, external warranty, SMS/email, analytics, accounting |
| Status names and technical enums match SRS/UC-12 | Pass — 8 statuses, `READY_FOR_REPAIR` (no `APPROVED`) |
| 13 domain entities represented in staff detail view | Pass (P10) |

## 3. Flow coverage

| Task flow | Covered | Where |
|---|---|---|
| Flow 1 — Submit request | Yes | customer-flows.md §2 |
| Flow 2 — In-warranty repair | Yes | staff-flows.md §3 |
| Flow 3 — Out-of-warranty, accept and reject branches | Yes | both flow docs |
| Flow 4 — Failed test and retry | Yes | staff-flows.md §5 |
| Flow 5 — Track and cancel | Yes | customer-flows.md §4 |

## 4. Acceptance checklist (task §11)

| Criterion | Status |
|---|---|
| All main customer pages designed | Done — P01–P06 |
| Staff and admin pages designed | Done — P07–P16 |
| Repair flows with and without warranty | Done — Flow 2 and Flow 3 |
| Proposal rejection and cancellation paths | Done — P05, P06 |
| Failed test and rework path | Done — Flow 4 |
| Role access reflected in UI | Done — role matrix + disabled/hidden actions |
| Error, success, loading states | Done — state bands on all 16 frames |
| Customer needs no account | Done — no login/signup on customer pages |
| No out-of-scope feature added | Done — see exclusions |
| Output in the required path, branch and PR | Files in `docs/design/ui-wireframes/`, branch `docs/ui-wireframes`; PR pending approval to push |

## 5. Open items for sign-off

1. **CR-01** — PM decision on whether the customer view may include device specs, problem description, and status history. Current design follows BR-06.
2. **CR-02** — Confirm four operational dashboard counters are acceptable.
3. **CR-03** — Confirm SVG + gallery is acceptable instead of a `.fig` file, or provide Figma access.
4. Team review of the end-to-end scenario (existing repo item CR-10) is still outstanding and is outside this wireframe task.

**Conclusion:** The wireframes are internally consistent with SRS v0.3, Use Cases, the Activity Diagram, the logical data model, and the approved architecture. The four items above need explicit PM/team confirmation before final sign-off.
