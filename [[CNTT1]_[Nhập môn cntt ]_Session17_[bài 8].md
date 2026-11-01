# Vá Lỗ Hổng Bảo Mật Cookie Bệnh Án Và Khắc Phục Lỗi Cache Thuốc Cũ

## ***Phần 1: Phân tích lỗ hổng & nguyên nhân***

**1. Tại sao danh mục thuốc vẫn cũ rích dù Server đã cập nhật?**

- Thủ phạm: Browser Cache (Bộ nhớ đệm của trình duyệt).

- Cơ chế: Khi trình duyệt tải các tệp tĩnh (medicine_catalog.css, pricing_data.json) từ tối hôm trước, nó thông minh (nhưng đôi khi quá đà) tự động lưu bản sao của các tệp này vào Memory Cache (trên RAM) hoặc Disk Cache (trên ổ cứng). Khi bác sĩ bấm F5 bình thường sáng nay, trình duyệt tưởng trang web không đổi nên lấy ngay bản cũ trong máy ra dùng thay vì gửi request lên Server xin tệp mới. Đó là lý do cột Size hiển thị (from disk cache) và thời gian nạp siêu tốc 2ms.

**2. Hiểm họa Session Hijacking do thiếu cờ HttpOnly và Secure:**

- Mã độc lấy cắp Cookie bằng cách nào: Nếu website dính lỗ hổng XSS (cho phép kẻ xấu chèn mã độc JavaScript), hacker chỉ cần gọi câu lệnh đơn giản sau trên trình duyệt nạn nhân:

- Hậu quả Session Hijacking: Do chuỗi Cookie carepulse_doctor_session=TOKEN_XYZ883921983 đại diện cho quyền định danh của bác sĩ, khi bị hacker cướp được, hắn có thể "giả mạo" y hệt bác sĩ để đăng nhập vào hệ thống CarePulse mà không cần mật khẩu. Hacker hoàn toàn có thể xem trộm hồ sơ bệnh án nhạy cảm, sửa đổi kết quả xét nghiệm hoặc kê đơn thuốc sai lệch, vi phạm nghiêm trọng quy định an toàn thông tin bệnh nhân của Bộ Y tế.

## ***Phần 2: Hoàn thiện giải pháp & Thao tác DevTools***

**3. Hướng dẫn làm mới cưỡng bức (Hard Reload) để cập nhật danh mục thuốc:**

Để xóa bỏ bảng giá cũ mà không ảnh hưởng đến lịch sử duyệt web khác, các bác sĩ làm theo các cách sau:

- Cách nhanh nhất (Phím tắt): Nhấn tổ hợp phím Ctrl + Shift + R (trên Windows) hoặc Cmd + Shift + R (trên Mac). Thao tác này ra lệnh cho trình duyệt bỏ qua toàn bộ cache cũ và tải lại 100% tệp mới từ Server.

- Cách thủ công qua F12:
  - Nhấn F12 để mở công cụ Developer Tools.
  - Nhấn và giữ chuột phải vào nút Reload (biểu tượng mũi tên xoay vòng ở góc trên cùng bên trái trình duyệt).
  - Chọn dòng "Empty Cache and Hard Reload" (Xóa bộ nhớ đệm và tải lại cứng).
  - Verification (Kiểm tra thành công): Cột Size trong tab Network sẽ hiển thị dung lượng tệp thực tế (ví dụ: 15.2 KB) thay vì dòng chữ (from disk cache) và bảng giá mới xuất hiện ngay lập tức.

**4. Dòng lệnh Set-Cookie chuẩn an ninh y tế:***

Viết lại dòng lệnh `Set-Cookie`: **Set-Cookie: carepulse_doctor_session=TOKEN_XYZ883921983; Domain=carepulse.vn; Path=/; Expires=Wed, 21 Oct 2026 07:00:00 GMT; HttpOnly; Secure; SameSite=Strict***

**5. 3 bước kiểm tra Cookie và vô hiệu hóa Cache tạm thời trên Chrome DevTools:**

- Mở DevTools: Nhấn phím F12 trên bàn phím.

- Kiểm tra cờ bảo mật Cookie: Chuyển sang tab Application $\rightarrow$ Ở cột bên trái, tìm mục Cookies $\rightarrow$ Chọn tên miền carepulse.vn. Tại đây, kiểm tra xem các cột HttpOnly và Secure đã được tích chọn hay chưa.

- Vô hiệu hóa Cache tạm thời khi đang fix lỗi: Chuyển sang tab Network $\rightarrow$ Tích chọn vào ô Disable cache (nằm ở thanh thực đơn phía trên). Chừng nào ô này còn được tích và bảng F12 còn mở, trình duyệt sẽ không bao giờ lưu cache tệp tĩnh nữa, giúp kỹ sư code test giao diện mượt mà không lo bị kẹt dữ liệu cũ.
