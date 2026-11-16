# Staff UI — Pages, Flows and Role Matrix

**Scope:** P07–P16, desktop-first. All staff accounts; backend enforces every permission.  
**Authority:** SRS v0.3 FR-05–FR-12, FR-15, BR-03, BR-08, BR-09, BR-11, BR-12; UC-02–UC-04, UC-07, UC-08, UC-10–UC-12; Activity Diagram.

## 1. Role matrix

| Action | Service Expert | Technician | Admin |
|---|:---:|:---:|:---:|
| Login (P07) | Yes | Yes | Yes |
| View requests list / details (P09, P10) | Yes | Yes | Yes |
| Filter requests by status (P09) | Yes | Yes | Yes |
| Receive device (P11) | **Yes** | No | No |
| Inspect device / warranty (P12) | **Yes** | No | No |
| Create cost proposal (P13) | **Yes** | No | No |
| Start repair / record operations (P14) | No | **Yes** | No |
| Record final test (P15) | No | **Yes** | No |
| Create staff / change role / activate (P16) | No | No | **Yes** |

Widgets and actions are filtered per role; hiding a button is not authorization. The backend re-checks role and status for every protected operation (ADR-008).

## 2. Page inventory

### P07 — Staff Login
Username, password, Sign In. Failed login shows an inline error. No customer login exists here.

### P08 — Staff Dashboard
Operational counters: awaiting reception, awaiting inspection, ready for repair, in repair. Recent request table with status and open action. Widget set varies by role. **No charts, KPIs, or analytical reports** (SRS §7).

### P09 — Repair Requests List
Columns: tracking code, customer, device type/model, status, submitted date. Search by tracking code. Filter by status. Row opens P10. Empty state: "No requests match the selected filters."

### P10 — Staff Request Details
Full internal record: customer, device, initial description, current status, status history, device receipt, inspection result and defects, proposal and customer decision, repair job, operations, all tests. Action buttons rendered only for the matching role **and** valid status.

### P11 — Device Reception (Service Expert)
- Request and device summary read-only.
- Device match selector: `MATCH` / `MISMATCH`.
- On `MISMATCH`: correction fields for brand (optional), model (required), serial (optional) — limited to `Device` data only (UC-02).
- Reception timestamp, optional notes, receiving expert from session.
- Save → `RECEIVED` + StatusHistory. Inspection remains blocked until this succeeds.

### P12 — Inspection & Warranty (Service Expert)
- Warranty result entered manually: `ELIGIBLE` / `NOT_ELIGIBLE`.
- Rejection reason required when `NOT_ELIGIBLE` (BR-09, IC-07).
- One or more defects, each with title and description; add-defect control.
- Recommended action: `REPAIR` / `DEVICE_REPLACEMENT` (expert decision, not a technician operation).
- Optional inspection notes.
- `ELIGIBLE` → `READY_FOR_REPAIR` directly: no proposal, no customer decision.
- `NOT_ELIGIBLE` → `INSPECTED` → P13.

### P13 — Repair Proposal (Service Expert)
- Inspection summary shown read-only.
- Single estimated total in **IRR**, numeric and greater than zero (IC-15).
- Optional notes; customer preview shows only allowlisted fields.
- Blocked unless warranty = `NOT_ELIGIBLE` and status = `INSPECTED`.
- Save → `AWAITING_CUSTOMER_DECISION` + StatusHistory. No line items, no payment.

### P14 — Repair Management (Technician)
- Jobs in `READY_FOR_REPAIR` / `IN_REPAIR` with device and diagnosed defects.
- **Start Repair** → `IN_REPAIR`; customer cancellation closes from this moment (BR-07).
- Operation fields: type (`REPAIR`, `PART_REPLACEMENT`, `DEVICE_REPLACEMENT`), description (required), part description (for part replacement), status (`PENDING`, `IN_PROGRESS`, `COMPLETED`).
- At least one operation required before a final test can be recorded.

### P15 — Repair Testing (Technician)
- Test result `PASSED` / `FAILED`, notes, tested-by from session.
- `FAILED`: notes mandatory, status stays `IN_REPAIR`, return to P14 for more operations, then retest (BR-12).
- `PASSED`: system sets `COMPLETED`, writes StatusHistory and `repairJob.completedAt` (BR-11).
- All previous test rows remain visible.

### P16 — Staff Management (Admin)
- Create account: full name, phone, username, initial password, role, active flag.
- Only `password_hash` is stored by the backend; raw password never persisted.
- Change role, activate/deactivate existing accounts. Duplicate username rejected.
- No reporting, finance, or inventory extensions.

## 3. Flow 2 — In-warranty repair

```
P02 Register (REGISTERED)
  → P11 Receive (RECEIVED)
  → P12 Inspect: ELIGIBLE (READY_FOR_REPAIR)
  → P14 Start repair + operations (IN_REPAIR)
  → P15 Test PASSED (COMPLETED)
  → P04 customer sees final allowlisted status
```

No proposal, no customer decision, no payment.

## 4. Flow 3 (staff part) — Out-of-warranty repair

```
P11 Receive → P12 Inspect: NOT_ELIGIBLE (INSPECTED)
  → P13 Proposal (AWAITING_CUSTOMER_DECISION)
  → customer P05
       ├─ Accept → READY_FOR_REPAIR → P14 → P15 PASSED → COMPLETED
       └─ Reject → CANCELLED (end; repair never starts)
```

## 5. Flow 4 — Failed test and retry

```
P14 Start repair (IN_REPAIR)
  → record operations
  → P15 Test FAILED -> notes required, status stays IN_REPAIR
  → P14 add further operations
  → P15 Test PASSED -> system sets COMPLETED
```

`FAILED` never produces a status transition. All test attempts are kept.

## 6. UI states

| State | Staff behavior |
|---|---|
| Initial | Forms empty; workflow-gated actions disabled with an explanatory notice. |
| Loading | Buttons disabled; skeletons in counters and tables. |
| Empty | "No requests" / "No operations yet" / "No tests yet" guidance. |
| Error | Load failure with retry; invalid credentials; stale status after concurrent change. |
| Validation | Missing required field, zero/negative amount, defect or operation without description. |
| Success | Confirmation message and status refresh after each save; location stays on the updated record. |
| Confirmation | Only destructive/irreversible staff actions (deactivate account, status-closing actions). |
| Disabled | Inspect before reception, proposal before inspection, start before `READY_FOR_REPAIR`, test before an operation. |

## 7. Traceability

| Wireframe | Use case | Requirements / rules |
|---|---|---|
| P07 | UC-11 related | FR-15, NFR-01 |
| P08 | UC-10 | FR-15 |
| P09 | UC-10 | FR-15 |
| P10 | UC-10 | FR-15, BR-06 boundary |
| P11 | UC-02 | FR-05, BR-02, IC-06 |
| P12 | UC-03 | FR-06, FR-07, BR-03, BR-09, IC-07, IC-08 |
| P13 | UC-04 | FR-08, BR-04, IC-15 |
| P14 | UC-07 | FR-11, BR-04, BR-07, IC-12, IC-13 |
| P15 | UC-08 | FR-11, BR-08, BR-11, BR-12, IC-18, IC-19 |
| P16 | UC-11 | FR-15, NFR-01 |
