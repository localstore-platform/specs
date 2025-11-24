Merchant FAQ — Hỏi đáp nhanh cho chủ quán (VN)

1) Làm sao để khách bắt đầu gọi món từ mã QR trên bàn?
   - Khách quét mã QR → trình duyệt mở trang menu dành cho bàn đó. Hệ thống tự tạo `session` cho bàn và mọi thao tác thêm món/sửa giỏ đều gắn vào session này.

2) Khách có thể chia hóa đơn không?
   - Có. Hệ thống hỗ trợ nhiều lần thanh toán trong cùng một session. Nhân viên có thể ghi nhận nhiều `payment` (ví dụ: tiền mặt + ví điện tử).

3) Khách chuyển bàn thì sao?
   - Nhân viên có thể chuyển session sang bàn mới bằng tính năng "Chuyển bàn" trong giao diện quản trị. Lịch sử đơn hàng vẫn được giữ nguyên.

4) Khách quét QR nhưng không thanh toán (bỏ giỏ) — dữ liệu này có được lưu?
   - Có. Phiên sẽ tự hết hạn sau TTL (mặc định 60 phút). Dữ liệu sự kiện vẫn được lưu để phân tích hành vi (không ảnh hưởng tới doanh thu).

5) Khách thanh toán bằng MoMo/VNPay/ZaloPay thì có tự cập nhật không?
   - Có. Khi cổng thanh toán trả webhook xác nhận, nền tảng tự động ghi nhận `payment_success` và đóng session (status = paid). Bạn không cần thao tác thủ công cho ví điện tử.

6) Khách trả tiền mặt — nhân viên cần làm gì?
   - Nhân viên vào phần Reconciliation → tìm session → bấm "Ghi nhận thanh toán tiền mặt" và nhập số tiền + chọn nhân viên. Session sẽ được đánh dấu `paid`.

7) Có thể in hóa đơn/biên nhận không?
   - Phiên bản MVP hỗ trợ xuất PDF/CSV; in hóa đơn trực tiếp (kết nối máy in nhiệt) là tính năng "soft POS" giai đoạn tiếp theo.

8) Làm sao để xử lý hoàn tiền (refund)?
   - Trong UI quản trị, chọn session (hoặc payment) và chọn `refund`. Hệ thống ghi một sự kiện `refund` và cập nhật báo cáo doanh thu.

9) Mã QR có thể bị người khác thao tác (mở cùng bàn) — có rủi ro không?
   - Phiên gắn với mã QR của bàn; mọi ai quét mã sẽ xem cùng session. Để tránh xung đột, khi đang checkout hệ thống có cơ chế lock-by-device và sẽ hiển thị cảnh báo khi có hành vi xung đột.

10) Dữ liệu khách hàng có được lưu không?
    - Chúng tôi chỉ lưu những trường cần thiết (ví dụ: phone hashed nếu khách cung cấp), và tuân thủ chính sách bảo mật. Không lưu thông tin thẻ thanh toán; sử dụng token/payment_reference từ gateway.

11) Nếu hệ thống mất mạng thì sao?
    - Ứng dụng web/Flutter có cơ chế retry: client sẽ lưu sự kiện tạm thời và gửi lại khi có mạng. Nếu khách đã thanh toán nhưng webhook chưa đến, nhân viên có thể dùng chức năng tìm kiếm `payment_reference` và đối soát thủ công.

12) Tôi muốn xem báo cáo hàng ngày — ở đâu?
    - Trên Dashboard → Metrics (hoặc nhận báo cáo hàng ngày qua email). Nếu có vấn đề đối soát, hệ thống sẽ gửi cảnh báo để bạn kiểm tra mục Reconciliation.

13) Làm thế nào để liên hệ hỗ trợ?
    - Gửi yêu cầu kèm `session_id` hoặc `payment_reference` tới đội support. Thông tin liên hệ sẽ có trong trang Manage → Support.

Short English notes (internal):

- QR session = session per table; supports multi-payments and manual cash entry.
- Staff should be trained to record cash payments daily to avoid mismatches.
- For disputes, `session_id` and `payment_reference` are the primary artifacts for support.

# End
