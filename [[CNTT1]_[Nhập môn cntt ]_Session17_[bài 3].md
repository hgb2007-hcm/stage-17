# Xử lý Sự cố Giao diện Website Doanh nghiệp

## ***Phần 1: Báo cáo phân tích (I/O) & Sơ đồ Box Model thực tế***

Khi mở tab Computed trong Chrome DevTools lên để soi phần tử div#lesson-content theo ảnh chụp thực tế, mình thu được cấu trúc Box Model hiện tại như sau:   

- Margin: Trái 28px (hoặc 10px tùy tính toán của engine), Phải -20px, Trên/Dưới 0px. Việc xuất hiện margin-right: -20px chính là "thủ phạm" kéo dịch phần tử sang phải, gây chồng lấn.   

- Border: 2px (được set bằng border: 2px solid red).   

- Padding: 15px ở các phía (hoặc hiển thị 12px tùy vùng nén nội dung).   

- Content: Phần lõi hiển thị nội dung bên trong (ví dụ vùng chứa văn bản và sơ đồ khoảng 840 x 600px).   

***Nguyên nhân gốc rễ (Root Cause):***

- Nhìn vào bảng Styles bên cạnh, cả hai phần tử div#lesson-content và div#video-list đều đang được gán cứng width: 65% !important kết hợp với float: left và margin-right: -20px. Khi tổng chiều rộng của hai khối lên tới hơn 130% cộng thêm các giá trị margin âm, trình duyệt bị quá tải không gian hiển thị, dẫn đến việc khung nội dung bị trượt và đè thẳng lên khung danh sách video.

## ***Phần 2: Thiết kế giải pháp khắc phục***

Để tách biệt hoàn toàn hai khung nội dung mà không làm vỡ tổng thể trang web, mình đề xuất điều chỉnh lại các thông số tại bảng Styles cho sạch sẽ như sau:

1. Để tách biệt hoàn toàn hai khung nội dung mà không làm vỡ tổng thể trang web, mình đề xuất điều chỉnh lại các thông số tại bảng Styles cho sạch sẽ như sau:

2. Loại bỏ Margin âm: Xóa bỏ hoặc đưa margin-right về 0px (hoặc một khoảng cách dương hợp lý như 2% để tạo khe hở gutter giữa hai cột).

3. Áp dụng Box-Sizing: Thêm thuộc tính box-sizing: border-box; cho toàn trang hoặc các phần tử này để trình duyệt tự động gom padding và border vào trong width, tránh việc bị phình to vỡ khung do cơ chế tính toán hình học mặc định (content-box).