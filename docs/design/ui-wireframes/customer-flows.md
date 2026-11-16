# Customer UI — Pages and Flows

**Scope:** P01–P06, mobile usable. No account, no login.  
**Authority:** SRS v0.3 BR-06, BR-07, BR-10; UC-01, UC-05, UC-06, UC-09; Activity Diagram.

## 1. Page inventory

### P01 — Home (`01-home.svg`)
- Short GaranTech introduction.
- Primary CTA: **Submit Repair Request** → P02.
- Secondary CTA: **Track Request** → P03.
- Four-step process strip: register → hand in device → inspection → repair/test/track.
- No login or signup entry point.

### P02 — Submit Repair Request (`02-submit-request.svg`)

| Field | Constraint |
|---|---|
| Full name | Required |
| Phone number | Required, normalized |
| Device category | Required — `MOBILE` or `LAPTOP` only |
| Brand | Optional (FR-02 decision) |
| Model | Required |
| Serial number | Optional |
| Initial problem description | Required |

- `*` marks required; `(optional)` marks optional.
- Validation errors appear under the offending field; no record is saved.
- Out-of-scope category: message **"Out of system scope"**, no request created (UC-01 alternate 2a).
- On success: request saved as `REGISTERED`, tracking code displayed, user told to hand in the device.
- Notice states clearly that online submission is **not** physical reception (BR-02).

### P03 — Track Request (`03-track-request.svg`)
- Two required inputs: **Tracking Code** and **Registered Phone Number**.
- Both must match; verification applies to tracking, proposal decision, and cancellation (ADR-006).
- Unknown code **and** wrong phone show the identical message **"Request not found"** — no existence leak (UC-09 alternate 1a).
- On success → P04.

### P04 — Request Details (`04-request-details.svg`)

Allowlisted content only (BR-06, `CustomerTrackingView`):

| Shown | Hidden |
|---|---|
| Tracking code | Full status history |
| Current status | Repair operations and test history |
| Warranty result | Internal notes and staff notes |
| Recommended action | Staff identities |
| Diagnosed defects | Device specs beyond identification |
| Proposed cost, when present | Internal proposal notes |

Conditional actions:
- `AWAITING_CUSTOMER_DECISION` → **Decide on Proposal** (P05).
- Any of `REGISTERED`, `RECEIVED`, `INSPECTED`, `AWAITING_CUSTOMER_DECISION`, `READY_FOR_REPAIR` → **Cancel Request** (P06).
- After `IN_REPAIR`: cancel action is hidden/disabled.

**Conflict flag:** the task brief also asks for device specs, initial problem description, and full status history on this page. Those are excluded here because BR-06 and CR-09 prohibit them. See [consistency-review.md](consistency-review.md), item CR-01.

### P05 — Customer Decision (`05-customer-decision.svg`)
- Visible only when warranty = `NOT_ELIGIBLE` and a proposal exists.
- Shows warranty result, estimated total in **IRR**, proposal notes.
- Optional customer note.
- **Accept** → confirmation → `READY_FOR_REPAIR` → P04.
- **Reject** → confirmation → `CANCELLED` → P04. Repair never starts (BR-05).
- Concurrent change (status no longer awaiting) → operation rejected, no extra History row.

### P06 — Cancel Request (`06-cancel-request.svg`)
- Reason is optional.
- Confirmation dialog: **Confirm cancellation** / **Keep request**. Decline leaves status unchanged.
- Allowed before `IN_REPAIR` only (BR-07). `CANCELLED`, `COMPLETED`, `IN_REPAIR` → disabled with reason.
- On confirm: `CANCELLED`, `cancelled_at` recorded, result shown on P04.

## 2. Flow 1 — Submit request

```
P01 Home
  └─ Submit Repair Request → P02
       ├─ invalid / out-of-scope → inline error, no record
       └─ valid → REGISTERED + tracking code → P03 Track
```

## 3. Flow 3 (customer part) — Out-of-warranty decision

```
P03 Track (code + phone)
  └─ P04 Details → AWAITING_CUSTOMER_DECISION
       └─ Decide on Proposal → P05
            ├─ Accept → READY_FOR_REPAIR → P04 → staff continue at P14
            └─ Reject → CANCELLED → P04 (end)
```

## 4. Flow 5 — Track and cancel

```
P03 Track → P04 Details
  └─ status in {REGISTERED, RECEIVED, INSPECTED, AWAITING_CUSTOMER_DECISION, READY_FOR_REPAIR}
       └─ P06 Cancel → confirm → CANCELLED → P04
  └─ status in {IN_REPAIR, COMPLETED, CANCELLED}
       └─ cancel disabled, reason shown
```

## 5. UI states

| State | Behavior on customer pages |
|---|---|
| Initial | Empty forms; CTAs enabled; no pre-filled session data. |
| Loading | Submit/sign-in/verify buttons disabled; skeleton on P04 while details load. |
| Empty | No saved requests to list; P03 still usable because tracking is explicit. |
| Error | Out-of-scope device, failed verification, stale status → explicit message, no data leak. |
| Validation | Required field empty, invalid phone, missing description → field-level message. |
| Success | Tracking code on P02; allowlisted result on P04; new status after P05/P06. |
| Confirmation | Mandatory on P05 (accept/reject) and P06 (cancel); decline is a no-op. |
| Disabled | Cancel and decision actions disabled outside their allowed statuses. |

## 6. Traceability

| Wireframe | Use case | Requirements |
|---|---|---|
| P01 | UC-01 entry | FR-01 |
| P02 | UC-01 | FR-01–04, BR-01, BR-02, BR-10, NFR-02 |
| P03 | UC-09 | FR-13, BR-06, BR-10 |
| P04 | UC-09 | FR-13, BR-06 |
| P05 | UC-05 | FR-09, BR-04, BR-05 |
| P06 | UC-06 | FR-14, BR-07 |
