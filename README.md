**1. Bối cảnh**

Hệ thống vận tải yêu cầu shipper phải đăng nhập để nhận đơn. Tuy nhiên, mỗi lần shipper đóng trình duyệt hoặc load lại trang, giỏ hàng phụ tùng xe lại bị trống rỗng và shipper phải đăng nhập lại từ đầu.

[image](https://rikkeiedu-storage.s3.ap-southeast-2.amazonaws.com/uploads/editor-images/1787706949950_5d77064f-b102-4265-9e03-58ab428c005b.webp)

**2. Yêu cầu**

- **Chức năng 1 (Duy trì trạng thái):** Vận dụng kiến thức về Persistent Cookies để thiết lập cơ chế ghi nhớ phiên làm việc thay vì sử dụng Session Cookies.
- **Chức năng 2 (Bảo mật):** Thiết lập các cờ bảo mật HttpOnly và Secure để ngăn chặn hành vi đánh cắp Session (Session Hijacking).

**3. Ràng buộc**

Phải đảm bảo dữ liệu định danh người dùng không bị mã JavaScript phía Client truy cập trái phép.

**4. Yêu cầu đầu ra**

Phần 1. Phân tích
1. So sánh Cache và Cookies
Tiêu chí	Cache	Cookies
Dung lượng	MB đến GB	Khoảng 4KB/cookie
Mục đích	Lưu hình ảnh, CSS, JavaScript	Lưu thông tin phiên và định danh
Thời gian lưu	Theo chính sách Cache	Theo thời hạn Cookie
Phạm vi	Tăng tốc tải trang	Ghi nhớ trạng thái người dùng

2. Giải pháp
- Sử dụng Persistent Cookies để ghi nhớ phiên đăng nhập ngay cả khi shipper đóng trình duyệt.
- Lưu Token xác thực thay vì lưu trực tiếp mật khẩu.
- Sử dụng Cache để lưu tài nguyên giao diện, không dùng Cache để lưu thông tin đăng nhập.
Phần 2. Triển khai
1. Cấu trúc Cookie

Ví dụ cấu hình Cookie:
Set-Cookie: session_token=abc123xyz;
Max-Age=604800;
Path=/;
HttpOnly;
Secure;
SameSite=Lax


2. Ý nghĩa các thuộc tính
- session_token: Token xác thực phiên đăng nhập của shipper.
- Max-Age=604800: Cookie tồn tại trong 7 ngày.
- Path=/: Cookie được sử dụng trên toàn bộ website.
- HttpOnly: Ngăn JavaScript phía Client đọc Cookie, hạn chế đánh cắp Token qua XSS.
- Secure: Chỉ gửi Cookie qua kết nối HTTPS.
- SameSite=Lax: Hạn chế Cookie được gửi trong các yêu cầu từ website khác, giúp giảm nguy cơ CSRF.
3. Cơ chế hoạt động
1. Shipper đăng nhập thành công.
2. Server tạo Token và gửi Cookie về trình duyệt.
3. Trình duyệt lưu Cookie theo thời hạn đã thiết lập.
4. Khi shipper đóng rồi mở lại trình duyệt, Cookie vẫn tồn tại.
5. Server kiểm tra Token và khôi phục phiên đăng nhập.
Lưu ý: Không lưu mật khẩu hoặc dữ liệu nhạy cảm trực tiếp trong Cookie. Token cần được tạo ngẫu nhiên, có thời hạn và có cơ chế thu hồi khi đăng xuất.
Phần 3. Kiểm thử
Nếu shipper thay đổi giao diện nhưng trình duyệt vẫn hiển thị màu cũ, nguyên nhân có thể do trình duyệt đang sử dụng CSS được lưu trong Cache thay vì tải phiên bản mới.

Cách thực hiện Hard Reload:
- Windows: Nhấn Ctrl + Shift + R hoặc Ctrl + F5.
- MacOS: Nhấn Cmd + Shift + R.
- Có thể mở Developer Tools bằng F12, nhấn giữ nút Reload và chọn Empty Cache and Hard Reload (trên trình duyệt hỗ trợ).
Kết quả: Trình duyệt tải lại tài nguyên giao diện từ Server, bỏ qua Cache cũ và hiển thị phiên bản mới nhất.

Kết luận: Persistent Cookies giúp duy trì đăng nhập, HttpOnly và Secure tăng cường bảo mật Token, còn Hard Reload giúp khắc phục tình trạng giao diện chưa cập nhật do Cache cũ.
