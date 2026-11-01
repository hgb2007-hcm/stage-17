# Gỡ Lỗi Luồng Yêu Cầu Kết Quả Xét Nghiệm Và Sai Lệch Mã Trạng Thái HTTP

## ***PHẦN 1: PHÂN TÍCH VẤN ĐỀ***

***1. Bốn điểm sai lệch chí mạng trong bản giải trình của thực tập sinh:***

- Sai về DNS: Ổng bảo trình duyệt gửi thẳng URL tới Web Server mà không cần DNS. Thực tế: Trình duyệt làm sao biết Web Server nằm ở chỗ quái nào nếu không có DNS (hệ thống phân giải tên miền) tra cứu tên miền carepulse.vn thành một địa chỉ IP cụ thể.

- Sai về TCP/TLS: Ổng bảo HTTPS tự động mở kết nối không cần bắt tay. Thực tế: Muốn có HTTPS bảo mật thì bắt buộc phải có quá trình TCP Handshake (bắt tay 3 bước mở kênh mạng) rồi tiếp tục TLS Handshake (trao đổi chứng chỉ mã hóa) thì dữ liệu mới truyền đi an toàn được. Không thể có chuyện nhảy cóc như thế.

- Sai về mã lỗi HTTP (404 vs 500): Sập server database mà lại đi trả về mã 404 Not Found và bảo bệnh nhân "hồ sơ đã bị xóa vĩnh viễn". Đây là một pha "quay xe" cực kỳ tai hại. 404 nghĩa là gõ sai đường dẫn URL không tồn tại, còn sập backend/database thì phải là 500 Internal Server Error.

- Sai về DevTools (F12): Đỉnh cao sự ngáo ngơ là nghĩ gõ sửa chữ "Âm tính" bằng Tab Elements trên trình duyệt máy mình thì database trên server của bệnh viện tự cập nhật luôn. Đúng là ảo tưởng sức mạnh lập trình!

***2. Phân tích mã 500 và hậu quả tâm lý khi nhầm thành 404:***

- Tại sao phải là 500? Khi máy chủ cơ sở dữ liệu sập nguồn hoặc lỗi code backend truy vấn dữ liệu, server vẫn nhận được yêu cầu (Request) từ bệnh nhân, nhưng nó bất lực không lấy được dữ liệu ra để xử lý tiếp. Lúc này, server phải trả về mã 500 Internal Server Error để báo hiệu lỗi từ phía hệ thống nội bộ của họ.

- Hậu quả tai hại nếu báo lỗi 404: Mã 404 ám chỉ tài nguyên không tìm thấy. Nếu nhân viên CSKH phán với bệnh nhân rằng "Hồ sơ bệnh án của anh đã bị xóa vĩnh viễn khỏi hệ thống", người bệnh nghe xong chắc chắn sẽ hoảng loạn tột độ, tưởng bệnh viện làm mất sạch kết quả xét nghiệm quan trọng, thậm chí kiện ra tòa vì tắc trách làm lộ hoặc mất dữ liệu y tế cá nhân.

## ***PHẦN 2: SỬA LỖI & HOÀN THIỆN GIẢI PHÁP***

***3. Viết lại Biên bản giải trình kỹ thuật chuẩn xác theo 4 bước vòng đời HTTP Request:***

***[BIÊN BẢN SỰ CỐ KỸ THUẬT CAREPULSE - CA BỆNH ÁN #8821 - ĐÃ HIỆU CHỈNH]***

- Bước 1 (DNS Lookup): Khi bệnh nhân Nguyễn Văn A nhấn xem kết quả, trình duyệt gửi tên miền carepulse.vn đến máy chủ phân giải tên miền (DNS Server) để dịch thành địa chỉ IP chính xác của máy chủ hệ thống CarePulse.

- Bước 2 (TCP/TLS Handshake): Sau khi có IP, máy tính thực hiện quá trình bắt tay 3 bước qua giao thức TCP, tiếp tục thiết lập bảo mật qua TLS Handshake để tạo kênh truyền tải mã hóa HTTPS an toàn cho dữ liệu y tế nhạy cảm.

- Bước 3 (HTTP Request & Response): Trình duyệt gửi gói tin HTTP GET /records/patient8821 lên server. Tại thời điểm này, dịch vụ backend cố gắng truy vấn cơ sở dữ liệu nhưng hệ thống DB gặp sự cố quá tải/sập nguồn. Server không thể xử lý và trả về mã lỗi 500 Internal Server Error.

- Bước 4 (Rendering Pipeline): Do nhận được phản hồi lỗi 500 thay vì dữ liệu JSON hợp lệ, trình duyệt không thể tạo lập cây DOM và CSSOM để hiển thị kết quả, dẫn đến việc màn hình hiển thị bảng trống trơn kèm thông báo lỗi hệ thống.

***4. Giải thích nguyên lý bộ nhớ: Tại sao sửa giao diện trên Tab Elements không thay đổi được Server Database?***

- Bản chất của trình duyệt khi nhận phản hồi từ server là tải toàn bộ mã nguồn về lưu tạm vào bộ nhớ RAM của máy cục bộ (Client-side RAM) và dựng thành cây DOM hiển thị lên màn hình.

- Thao tác dùng công cụ F12 > Tab Elements của ông thực tập sinh chỉ tác động trực tiếp lên cây DOM đang nằm trên RAM của chiếc máy tính cá nhân đó, hoàn toàn giống như việc mình dùng bút xóa viết đè lên một tờ giấy nháp.

- Dữ liệu gốc, cốt lõi của toàn bộ bệnh nhân vẫn nằm an toàn trên Server-side Database đặt tại trung tâm dữ liệu của hệ thống. Ngay khi người dùng nhấn F5 (Reload) hoặc đóng tab mở lại, trình duyệt sẽ xóa sạch RAM cũ, gửi một request mới tinh lên server để xin lại dữ liệu thật. Lúc này, mọi chữ "Âm tính" tự chế trên màn hình sẽ biến mất không dấu vết, trả lại hiện trạng lỗi ban đầu.