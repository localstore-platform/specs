# Reconciliation API Specification

Purpose: provide a clear contract (document-only) for the reconciliation UI and backend so frontend engineers can scaffold pages and QA can test flows.

Auth & common patterns

- All endpoints require Authorization: Bearer {access_token}
- Responses follow repo conventions: { data: {...}, meta: {...} }
- Rate limits: reconciliation endpoints are low-traffic. Recommend 30 req/min per user.

Endpoints

1. List open reconciliation issues

    GET /api/v1/tenants/{tenantId}/reconciliation/issues?status=open&limit=50&cursor={cursor}

    Query params:

    - status: open|resolved|ignored (default open)
    - limit: number (default 50)
    - cursor: opaque cursor for paging

    Response 200
    {
      "data": {
        "issues": [
          {
            "id": "uuid",
            "issue_type": "UNMATCHED_SESSION",
            "session_id": "uuid",
            "payment_id": null,
            "candidate_payment_ids": ["uuid"],
            "session_total": 40000,
            "payment_total": null,
            "confidence": 0.72,
            "details": {},  //json with items, timestamps, suggestions
            "status": "open",
            "created_at": "2025-10-22T08:00:00Z"
          }
        ],
        "next_cursor": "base64(...)"
      }
    }

2. Get single issue detail

    GET /api/v1/tenants/{tenantId}/reconciliation/issues/{issueId}

    Response 200
    {
      "data": {
        "id": "uuid",
        "issue_type": "AMOUNT_MISMATCH",
        "session_id": "uuid",
        "session_detail": {}, // snapshot of items, totals
        "candidate_payments": [
          {}
        ], // payment records
        "details": {},  // mismatch breakdown
        "status": "open"
      }
    }

3. Resolve an issue

    POST /api/v1/tenants/{tenantId}/reconciliation/issues/{issueId}/resolve
    Content-Type: application/json

    Request body (examples):

    - Match to a payment:
    {
      "action": "match",
      "payment_id": "uuid",
      "note": "Matched by manager, payment ref 12345"
    }

    - Mark as cash/taken offline:
    {
      "action": "mark_cash",
      "operator_id": "staff-uuid",
      "note": "Cash received by NV A"
    }

    - Ignore / false positive:
    {
      "action": "ignore",
      "note": "migration artifact"
    }

    Response 200
    {
      "data": {
        "id": "uuid",
        "status": "resolved",
        "resolved_at": "2025-10-23T07:30:00Z",
        "resolved_by": "uuid"
      }
    }

4. Create manual payment (staff records cash)

    POST /api/v1/tenants/{tenantId}/reconciliation/manual-payment
    Content-Type: application/json

    Request body:
    {
      "session_id": "uuid",
      "payment_method": "cash",
      "amount": 40000,
      "operator_id": "staff-uuid",
      "note": "Recorded at cashier station"
    }

    Response 201
    {
      "data": {
        "payment_id": "uuid",
        "matched": true
      }
    }

5. Re-run reconciliation (admin)

    POST /api/v1/tenants/{tenantId}/reconciliation/run
    Content-Type: application/json
    { "date": "2025-10-22" }

    Response 202 Accepted
    {
      "meta": { "job_id": "uuid", "status": "queued" }
    }

    DTOs (document)

    ReconciliationIssue {
      id: uuid,
      tenant_id: uuid,
      location_id: uuid,
      issue_type: string,
      session_id?: uuid,
      payment_id?: uuid,
      candidate_payment_ids?: uuid[],
      session_total?: bigint,
      payment_total?: bigint,
      confidence?: float,
      details?: jsonb,
      status: string,
      created_at: timestamptz
    }

    ResolveRequest {
      action: 'match' | 'mark_cash' | 'ignore',
      payment_id?: uuid, // required for 'match'
      operator_id?: uuid, // recommended for cash
      note?: string
    }

    ManualPaymentRequest {
      session_id: uuid,
      payment_method: string,
      amount: bigint,
      operator_id?: uuid,
      note?: string
    }

    Errors

    - 400 INVALID_INPUT
    - 401 AUTH_TOKEN_MISSING
    - 403 FORBIDDEN
    - 404 NOT_FOUND
    - 409 CONFLICT (e.g., trying to match already resolved issue)

    Observability & metrics

    - reconciliation_job_duration_seconds
    - unmatched_sessions_count
    - unmatched_payments_count
    - auto_resolve_count

    Rate limiting & access

    - Limit resolves to owner/manager roles. Provide audit log for all resolve actions.

# End
