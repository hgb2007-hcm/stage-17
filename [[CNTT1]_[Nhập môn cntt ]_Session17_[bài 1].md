# Hệ thống Giám sát Sự cố Website Doanh nghiệp

## ***Phần 1: Phân tích lỗi sai trong lập luận của Nam***

***1. Sai lầm về DNS và mã trạng thái 500:***

- Nam nghĩ gì: Nam thấy trang web xoay vòng lâu rồi hiện lỗi, tưởng là do DNS không tìm thấy địa chỉ IP nên mới đòi đổi sang Google DNS (8.8.8.8).

- Thực tế phũ phàng: Nếu DNS thực sự tạch (không phân giải được tên miền lms.ptit.edu.vn), trình duyệt của Nam sẽ ném ra mặt ngay lập tức các thông báo kiểu như ERR_NAME_NOT_RESOLVED hoặc DNS_PROBE_FINISHED_NXDOMAIN. Trình duyệt thậm chí còn chẳng thể kết nối đi đâu được chứ đừng nói là nhận về mã 500.

- Bản chất mã 500: Mã 500 Internal Server Error có nghĩa là trình duyệt đã tìm thấy server, kết nối thành công, gửi request đi ngon lành, nhưng thằng Server bên kia đang gặp sự cố nội bộ (ví dụ: sập database, lỗi code backend, quá tải...). Lỗi này nằm hoàn toàn ở phía Server, mạng nhà Nam hay DNS hoàn toàn vô tội. Cố gọi thợ sửa mạng hay đổi DNS lúc này chỉ tốn tiền oan thôi

***2. Đối chiếu nhanh mã 500 với các mã khác cho dễ hình dung:***

- 200 OK (Mọi thứ bình thường): Nam gõ địa chỉ, server nhận bài, xử lý mượt mà và trả về giao diện LMS đầy đủ để điểm danh, nộp bài.

- 404 Not Found (Lỗi phía Client/Đường dẫn): Nam gõ nhầm đường dẫn kiểu lms.ptit.edu.vn/abcxyz. Server vẫn sống và trả lời rằng: "Tao tìm khắp nơi rồi nhưng không thấy cái trang ông vừa gõ đâu cả". Lỗi này do người dùng nhập sai link.

- 500 Internal Server Error (Lỗi phía Server): Nam gõ đúng link, kết nối tới tận cửa nhà server rồi, nhưng đúng lúc đó bếp nhà server bị cháy (lỗi code/hệ thống). Server chỉ biết ôm đầu chịu trận và hét lên mã 500 chứ không trả về được dữ liệu gì sất.

## ***Phần 2: Hoàn thiện luồng dữ liệu (Tại sao Nam nhận được mã 500?)***

**Để Nam hiểu tại sao mình nhận được cái lỗi 500 đấy thay vì lỗi mạng, luồng đi của gói tin từ lúc Nam bấm Enter diễn ra như sau:**

1. DNS Lookup (Phân giải tên miền):
  - Trình duyệt của Nam hỏi hệ thống xem lms.ptit.edu.vn ứng với địa chỉ IP nào.

  - Quá trình này thành công tốt đẹp, DNS trả về địa chỉ IP của máy chủ LMS trường PTIT. (Nếu đoạn này hỏng thì Nam đã tạch từ vòng gửi xe như đã phân tích ở Phần 1).

2. Thiết lập kết nối (TCP & TLS Handshake):

- Máy tính của Nam thực hiện bắt tay 3 bước (SYN, SYN-ACK, ACK) với địa chỉ IP của server trường để mở một kênh kết nối an toàn. Kết nối được thiết lập thành công.

3. Gửi HTTP Request:

- Trình duyệt đóng gói yêu cầu (HTTP Request) kiểu như: "Ê server, cho tao xin trang chủ LMS nhé" rồi gửi qua đường cáp mạng phóng thẳng đến server của trường.

4. Server xử lý và gặp nạn (Nguồn gốc lỗi 500):

- Server trường PTIT nhận được request của Nam.

- Tuy nhiên, trong quá trình backend xử lý (ví dụ: câu lệnh truy vấn cơ sở dữ liệu bị lỗi cú pháp, hoặc lượng truy cập quá đông làm server quá tải cúm cục bộ), chương trình bị crash/exception không lường trước.

- Vì không thể xử lý tiếp để trả về trang web, server đành bất lực trả về mã trạng thái 500 Internal Server Error.

5. Server Response & Hiển thị trên trình duyệt:

- Phản hồi mang mã 500 bay ngược về máy Nam.

- Trình duyệt nhận được mã 500, dừng ngay việc xoay vòng và hiển thị cái màn hình lỗi to đùng cho Nam thấy.

***Kết luận: Bảo Nam cứ bình tĩnh, đừng gọi thợ mạng hay đổi DNS nữa, việc cần làm lúc này là... đi ngủ hoặc đi chơi đợi bao giờ mấy anh quản trị hệ thống của trường fix xong lỗi server thì vào lại bình thường***