# Glossary (VN ↔ EN)

Use this as a lightweight, evolving glossary for common terms in our Vietnam-focused work.

## Payments & Commerce

- MoMo — Vietnamese e-wallet provider.
- ZaloPay — Vietnamese e-wallet provider.
- VNPay — Payment gateway & QR ecosystem in Vietnam.
- phí chiết khấu — commission fee.
- thanh toán không tiền mặt — cashless payment.

## Product & Features

- Zalo Mini App — a web-based mini application running inside the Zalo app.
- menu QR / QR menu — QR-based digital menu for scanning at tables.
- đặt món — place order.
- mang đi (take-away) — takeaway; to-go order.
- giao hàng — delivery.
- khuyến mãi — promotion/discount.

## Operations

- bảng điều khiển (dashboard) — admin dashboard.
- mục (item), danh mục (category) — menu item, category.
- tồn kho — inventory.
- thông báo đẩy — push notification.

## Technical Terms

- SKU (Stock Keeping Unit) / mã sản phẩm — unique code for inventory tracking and POS integration.
- display_order / thứ tự hiển thị — numeric field controlling visual sequence of items in menus, categories, and lists.

Feel free to add new entries as terms recur; keep entries brief with one preferred English equivalent.

## QR Session Terms

- session / phiên — a short-lived table-level session created when a customer scans a table QR; used to track item_add/item_remove events until payment (paid/expired/cancelled).
- session_event / sự kiện phiên — an atomic event recorded against a session (item_add, item_remove, payment_attempt, payment_success, refund).
- session_id / id phiên — canonical UUID identifying a session across APIs and analytics (use this for reconciliation and support).
- idempotency_key / khoá idempotency — client-generated token used to deduplicate retried writes (ensures single event persisted per client action).
- event_seq / thứ tự sự kiện — server-assigned monotonic sequence number for events within a session, used as cursor offset for incremental reads.
- payment_match_rate / tỉ lệ khớp thanh toán — metric: matched_payments / total_payments; used to measure reconciliation quality.
- order_capture_rate / tỉ lệ ghi nhận đơn — metric: (sessions_paid + manual_reconciled) / expected_sales; proxy for how much real revenue is captured by QR sessions.
- TTL (time-to-live) / thời gian tồn tại — configured inactivity window after which an active session is marked expired; default 60 minutes for cafe workflows.
- raw_payload / payload thô — unmodified vendor/client payload stored for auditing and migration (keep for 1 year, redact PII as required).
