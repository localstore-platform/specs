# Reconciliation UI — Wireframe & Interaction Notes (Document-only)

## Purpose

Provide a clear, low-fidelity wireframe and interaction guide for the merchant reconciliation UI used to review and resolve nightly `reconciliation_issues`. This document is intentionally text-based so frontend teams can implement quickly without visual assets.

## Pages / Components

1. Reconciliation Dashboard (Landing)

   - Purpose: quick summary for store manager each morning
   - Top-level items:
     - Date selector (default: yesterday)
     - KPIs: open_issues_count, unmatched_sessions_count, unmatched_payments_count, total_mismatch_amount
     - Quick actions: "Run reconciliation now", "Export CSV", "Mark all as reviewed"
     - List preview: show first 10 open issues (issue type badge: UNMATCHED_SESSION / UNMATCHED_PAYMENT / AMOUNT_MISMATCH / PARTIAL_MATCH)

   - List row fields (compact view):
     - Issue ID (clickable)
     - Type (badge)
     - Location / Table (if applicable)
     - Session time
     - Session total (VND)
     - Payment total (VND or hyphen)
     - Confidence (0–100%)
     - Quick actions: Match | Mark cash | Ignore

2. Issue Detail (primary screen for resolving)

   - Header: Issue ID, status (open/resolved/ignored), created_at, location
   - Left column: Session snapshot
     - Session code, table_id, created_at, last_activity_at
     - Itemized cart: [name, qty, modifiers, unit_price, line_total]
     - Totals: subtotal, tax, discount, total
   - Right column: Candidate payments & suggestions
     - List of candidate payments with payment_method, reference, amount, timestamp, confidence
     - Action buttons per candidate: Match (bind this payment), View payment raw payload
     - If no candidate: "Record manual payment" button
   - Bottom: Resolution controls
     - Resolve as: [Match to payment (requires selection)] [Mark as cash] [Ignore]
     - Input: note (required when resolving)
     - Operator selector (if resolving as cash) to record staff id
     - Resolve button + Cancel

3. Manual Payment Modal

   - Fields:
     - Session_id (readonly)
     - Payment method (dropdown: cash, momo, vnpay, zalopay, other)
     - Amount (prefilled with session total, editable)
     - Operator (staff) id (dropdown)
     - Note
   - Actions: Save & Match (creates session_payment and links it)

4. Bulk Actions & Export

   - Bulk selection in list view; actions: Match (select payment for each), Mark as cash, Ignore, Export selected to CSV
   - Export format: CSV with columns [issue_id, session_id, created_at, session_total, payment_total, suggested_match_ids, status]

5. Activity Log & Audit Trail

   - For each issue show timeline of actions: created_by (system or job), candidate_matches_generated, manual_payment_created, resolved_by, resolution_note.
   - UI element: small "History" accordion in Issue Detail.

## Behavior & Edge Cases

- If user clicks "Match" and payment has changed status (e.g., payment now refunded), show warning and prevent match unless operator confirms.
- Partial payments: allow selecting multiple payments to sum to session total; UI shows running total and leftover amount.
- Offline/manual fixes: staff can create manual payment; it will appear in candidate list and be auto-matched if amounts equal.
- Concurrency: lock issue for editing for 2 minutes by default; show "Editing by X" if lock held by another operator.
- Undo: allow "Re-open" action for 24 hours after resolution to revert incorrect resolves (logs audit trail).

## API integration notes

- Use `architecture/reconciliation-api-spec.md` endpoints:
  - List: GET /tenants/{tenantId}/reconciliation/issues
  - Detail: GET /tenants/{tenantId}/reconciliation/issues/{issueId}
  - Resolve: POST /tenants/{tenantId}/reconciliation/issues/{issueId}/resolve
  - Manual payment: POST /tenants/{tenantId}/reconciliation/manual-payment
- Polling vs live updates: UI should poll `issues` list every 30s during active session; staff dashboard can optionally use SSE when available.

## UI Accessibility & Localisation

- All labels and help text must be available in Vietnamese (primary) and English (secondary).
- Large tap targets for mobile management; table list supports pagination and search (by session_code, payment_reference).

## Minimal CSS/Design notes (for frontend)

- Compact list with left badge color coding: red=critical, amber=warning, blue=info.
- Use Vietnamese locale for currency display: format as ₫ with thousand separators.
- Provide keyboard shortcuts: J/K to navigate list, Enter to open issue, R to open Resolve dialog.

## Acceptance criteria (for docs-only handoff)

- Frontend team can implement screens using the API spec without additional design assets.
- All operations must be auditable with `resolved_by` and `resolution_note` stored server-side.
- Reconciliation UI must support locale VN, and default to local timezone per location.
