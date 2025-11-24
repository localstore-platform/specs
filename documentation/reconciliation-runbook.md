Reconciliation Runbook — For Store Managers (VN)

Mục đích: Hướng dẫn hàng ngày để xác nhận rằng doanh thu trên nền tảng khớp với thực tế (giao dịch ví điện tử + tiền mặt).

Tóm tắt nhanh:

- Kiểm tra mục "Đối soát" trong dashboard mỗi sáng sau ca làm việc.
- Nếu có "mismatch" (khác số tiền hoặc chưa đối soát), thực hiện 1 trong các thao tác: ghép giao dịch, ghi nhận tiền mặt, hoặc bỏ qua với ghi chú.

Hướng dẫn chi tiết (VN)

1) Mở Dashboard → Reconciliation → chọn ngày hôm qua.
2) Xem tổng số session marked `paid` và tổng số tiền. Hệ thống sẽ hiển thị số các vấn đề (open issues).
3) Với mỗi issue:
   - Xem chi tiết: session_id, thời gian tạo, danh sách items, tổng session, candidate payments.
   - Nếu có candidate payment khớp: chọn "Match" → hệ thống sẽ liên kết payment với session.
   - Nếu khách trả tiền mặt: chọn "Mark as cash" và điền tên nhân viên + ghi chú.
   - Nếu là giao dịch bị hoàn/duplicate: chọn "Resolve as refund/duplicate" và ghi chú.
4) Sau xử lý, nhấn "Resolved". Ghi chú phải mô tả lý do (ví dụ: "Khách trả tiền mặt — NV A ghi nhận").
5) Nếu có nhiều lệnh chưa đối soát, export CSV và gửi cho kế toán nếu cần.

Tình huống phổ biến & xử lý

- Giao dịch ví không xuất hiện: kiểm tra payment_reference và thời gian, liên hệ support với payment_reference.
- Giao dịch ngoại lệ (partial payments): chọn nhiều payment để ghép (partial match).
- Phiên hết hạn nhưng có thanh toán: tìm session bằng `session_code` trong công cụ tìm kiếm và ghép thủ công.

Ghi chú vận hành

- Khuyến nghị: nhân viên ca cuối kiểm tra và ghi nhận mọi thanh toán tiền mặt mỗi ngày trước khi đóng ca.
- Nếu có hành vi gian lận đáng ngờ (thường xuyên mismatch cao), liên hệ đội hỗ trợ và tạm dừng nhân viên liên quan.

Quick English checklist (for HQ)

- Check reconciliation dashboard daily.
- For each open issue: match candidate, mark cash, or resolve with reason.
- Export CSV for accounting if multiple unresolved issues.
- Escalate repeated mismatches to support.

Support

- Provide payment_reference and timestamps when contacting support.
- For merchant onboarding, schedule a 15-minute session to walk managers through the runbook.

# End
