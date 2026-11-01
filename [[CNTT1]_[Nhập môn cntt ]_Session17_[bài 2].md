# Xử lý Sự cố Cache và Phiên Đăng nhập trong Doanh nghiệp

## ***Phần 1: Phân tích nguyên nhân***

**1. Tại sao sửa CSS mà nút thanh toán vẫn xanh lè? (Browser Cache)**

- Thủ phạm: Do Browser Cache (Bộ nhớ đệm của trình duyệt).

- Giải thích: Để tối ưu tốc độ tải trang và đỡ tốn băng thông, trình duyệt có thói quen "nhớ dai" mấy cái file tĩnh như CSS, hình ảnh, JS. Khi An ấn F5 (Normal Refresh), trình duyệt tưởng bở là trang web cũ vẫn thế nên nó lôi ngay cái file CSS cũ rích đang nằm sẵn trong máy ra xài, thèm gì đi hỏi server xem có gì mới đâu. Thế nên nút thanh toán vẫn cứng đầu hiện màu xanh là vì thế.

**2. Tại sao cứ đóng trình duyệt là mất phiên đăng nhập? (Session Cookie vs. Persistent Cookie)**

- Thủ phạm: Do cấu hình cookie thiếu thời gian sống, khiến nó biến thành Session Cookie.

- Giải thích:

   - Khi An đăng nhập mà code backend không gán thêm thuộc tính Max-Age hoặc Expires, trình duyệt sẽ hiểu đây là Session Cookie (Cookie phiên làm việc). Đặc tính của thằng này là "sinh ra cùng tab, mất đi cùng trình duyệt" — nghĩa là cứ tắt trình duyệt hay đóng cửa sổ là nó tự động bốc hơi khỏi RAM.

   - Ngược lại, nếu muốn nó nhớ lâu hơn ngay cả khi tắt máy đi ngủ, An phải dùng Persistent Cookie bằng cách định nghĩa rõ thời gian sống (Max-Age hoặc Expires) để trình duyệt lưu nó xuống ổ cứng.

## ***Phần 2: Sửa lỗi nhanh gọn lẹ***

Để An không phải xóa sạch sành sanh lịch sử web và giữ được người yêu... à nhầm, giữ được trạng thái đăng nhập, áp dụng ngay 2 chiêu này:

***1. Đập tan bộ nhớ đệm để cập nhật giao diện ngay lập tức:***

- Cách làm: Thay vì dùng F5 thông thường, An chơi chiêu Hard Reload bằng tổ hợp phím Ctrl + F5 (trên Windows) hoặc Ctrl + Shift + R.

- Mẹo nâng cao: Nếu vẫn không ăn thua, bấm F12 để mở Developer Tools, sau đó nhấn và giữ chuột phải vào cái nút Reload (vòng tròn xoay ở góc trái trên cùng) rồi chọn dòng "Empty Cache and Hard Reload". Cách này ép trình duyệt xóa sạch cache cũ của riêng trang đó và tải mới 100% từ server mà không làm ảnh hưởng gì đến lịch sử lướt web khác của An.

***2. Giữ phiên đăng nhập sống dai:***

- Cách làm: Sửa lại đoạn code set cookie ở phía server khi user đăng nhập thành công. Bổ sung thêm thuộc tính Max-Age (tính bằng giây) hoặc Expires.