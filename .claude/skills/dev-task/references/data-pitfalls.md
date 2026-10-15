# Bẫy dữ liệu và hệ thống thường gặp

Đọc khi: task đụng tới query, dữ liệu, tiền, ngày, transaction hoặc lời gọi hệ thống ngoài.

- **Multi-tenant**: mọi query phải lọc theo khóa tenant (công ty/cửa hàng/user). Bảng con không có khóa tenant → JOIN bảng cha để lọc. Field định danh nhận từ client (công ty, shop, platform…) phải lấy từ token hoặc đối chiếu với bản ghi gốc, không tin giá trị trong body.
- **Double count**: JOIN 1-n rồi `SUM`/`COUNT` cột của bảng phía 1 là sai.
- **Giá trị trạng thái/enum**:
  - DB có thể chứa biến thể (chính tả khác nhau, hoa/thường, giá trị cũ) → xem `SELECT DISTINCT` hoặc code đang lọc thế nào, đừng chỉ tin tên hằng;
  - tên hằng có thể khác giá trị → validate đầu vào với tập giá trị hợp lệ, không tự map đoán.
- **Cùng một giá trị nhưng khác kiểu** khi đi qua config/cache/DB (vd chuỗi `"4"` với số `4`) → so sánh sai mà không báo lỗi.
- **Ngày**: phân biệt ngày nghiệp vụ với ngày tạo bản ghi (lúc job chạy). Cẩn thận timezone ở biên ngày.
- **Tiền**: kiểu decimal chính xác, làm tròn ở tầng trả về chứ không làm giữa các phép cộng; chia cho 0 → trả 0, không ném lỗi.
- **Transaction không bao lời gọi HTTP/IO ra ngoài**: lỗi bên ngoài có thể đánh dấu rollback cả transaction, và giữ DB connection suốt lúc chờ.
- **Bảng lớn**: query mới phải dùng được index sẵn có (cột WHERE/JOIN, cột INCLUDE). Nghi full scan → báo trước, kèm script index đề xuất.
- **Cache lệch DB**: dữ liệu cache có thể cũ hoặc khác kiểu với DB. Xác định nguồn nào là chuẩn.
- **Phản hồi của hệ thống ngoài**: "thành công" ở tầng HTTP chưa chắc là thành công nghiệp vụ (vd lỗi nằm trong danh sách lỗi của response). Không đưa nguyên câu lỗi của hệ thống ngoài cho người dùng; không log trường chứa dữ liệu cá nhân.

Bẫy riêng của repo nằm trong `CLAUDE.md`/docs của repo, hoặc knowledge cache của hub. Phát hiện bẫy mới → ghi vào knowledge cache của hub; muốn ghi vào `CLAUDE.md` của repo thì hỏi trước. **Không ghi vào skill này.**
