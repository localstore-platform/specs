QR Session Quickstart — Hướng dẫn nhanh cho chủ quán (VN)

Mục tiêu: Hỗ trợ chủ quán nhanh chóng in mã QR cho từng bàn, hiểu luồng đơn hàng và cách đối soát khi khách thanh toán (MoMo/VNPay/ZaloPay hoặc tiền mặt).

Tóm tắt ngắn (English): This quickstart explains how QR-per-table sessions work: customers scan table QR → session is created → customers add items → they pay (or staff record cash). The platform records item-level events for analytics.

1) Chuẩn bị
   - Tạo tài khoản trên nền tảng và cấu hình cửa hàng (location).
   - In mã QR cho mỗi bàn. QR chứa session code (ví dụ: `store.example.com/s/abcd1234`).
   - Thiết lập phương thức thanh toán: MoMo, VNPay, ZaloPay (tùy chọn). Nếu không dùng e-wallet, staff sẽ cần ghi nhận thanh toán tiền mặt trong giao diện quản trị.

2) Quy trình phục vụ
   - Khách quét QR trên bàn → mở menu dành cho bàn đó.
   - Khách thêm món vào giỏ, sửa số lượng, chọn modifiers.
   - Khách bấm "Thanh toán" và chọn ví điện tử hoặc quét mã QR thanh toán.
   - Khi thanh toán thành công qua cổng, hệ thống tự xác nhận và đóng phiên (session status = paid).
   - Nếu khách trả tiền mặt, nhân viên vào giao diện quản trị → tìm session → bấm "Ghi nhận thanh toán tiền mặt" và nhập số tiền và mã nhân viên.

3) Hướng xử lý tình huống
   - Khách bỏ giỏ (không thanh toán): hệ thống sẽ tự động hết hạn phiên sau 60 phút không hoạt động. Dữ liệu vẫn được lưu để phân tích hành vi.
   - Khách chuyển bàn: nhân viên có thể chuyển session sang mã bàn mới trong giao diện quản trị.
   - Tách hóa đơn: hỗ trợ nhiều lần thanh toán vào cùng session bằng cách ghi nhận nhiều `payment` (ví dụ: chia tiền mặt + ví điện tử).
   - Hoàn tiền: tạo sự kiện `refund` trong phần quản trị, hệ thống sẽ cập nhật báo cáo doanh thu.

4) Đối soát & báo cáo
   - Hệ thống tự động đối soát các giao dịch ví điện tử bằng webhook; bạn chỉ cần kiểm tra các mục "Chưa đối soát" trong dashboard mỗi ngày.
   - Đối với tiền mặt, nhân viên cần ghi nhận thanh toán vào session để đảm bảo báo cáo chính xác.
   - Hệ thống có báo cáo "mismatch" nếu tổng session không khớp với tổng thanh toán — người quản lý cần xác minh (nhân viên, lỗi nhập, gian lận).

5) Mẹo vận hành
   - Huấn luyện nhân viên: 10–15 phút để biết cách tìm session, ghi nhận thanh toán tiền mặt, và xử lý chuyển bàn/tách hoá đơn.
   - Khuyến nghị TTL session: 60 phút cho quán cafe; 90–120 phút cho nhà hàng ăn tối nơi khách ngồi lâu.
   - Khi bắt đầu pilot, khuyến nghị bật thông báo hàng ngày cho quản lý để xem các phiên chưa đối soát.

6) Hỗ trợ & liên hệ
   - Nếu gặp lỗi thanh toán ví điện tử, hãy gửi chi tiết `payment_reference` và thời gian giao dịch đến đội hỗ trợ.
   - Tài liệu chi tiết kĩ thuật cho đội IT: `architecture/api-specification.md` (QR Sessions & Table Ordering).

----

Short English summary (for internal use):

- Print QR per table, configure e-wallets for instant payment. Staff can manually record cash payments. System keeps session events for analytics and supports merging/transfer/refund. TTL default 60 minutes. See API doc for engineers.
