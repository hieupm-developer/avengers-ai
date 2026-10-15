# Clean code

Đọc khi: viết code (implementer) hoặc review diff (reviewer).

Linter của repo thường không kiểm tra hết các rule dưới đây → trước khi báo xong, **tự soát diff theo từng mục**.

Code xung quanh vi phạm rule → **code mới vẫn theo rule**, code cũ ngoài phạm vi task thì không sửa. Chỗ nào nguy hiểm thì ghi vào "Phát hiện thêm".

## Phạm vi áp dụng theo mode
- **feat**: code mới phải đạt toàn bộ file này.
- **fix / hotfix**: chỉ áp dụng cho các dòng đang sửa. **Không dọn code xung quanh.**
- **refactor**: đây chính là mục tiêu, nhưng hành vi phải giữ nguyên (`modes.md`, refactor).

## Đặt tên
- Tên nói lên ý định: `findExpiredShops`, `calculateNetRevenue`. Không dùng `process`, `handle2`, `data`, `tmp`, `list1`.
- Boolean bắt đầu bằng `is`/`has`/`can`; collection đặt tên số nhiều.
- Một khái niệm nghiệp vụ chỉ một tên. Dùng tên đã có trong code, không đặt tên mới đồng nghĩa.

## Hàm
- Một hàm làm một việc. Dài hơn ~40 dòng hoặc lồng sâu quá 3 cấp → tách.
- Dùng guard clause và return sớm thay cho if-else lồng nhau.
- Tối đa 4 tham số; nhiều hơn thì gom vào object. Không dùng tham số boolean để rẽ nhánh hành vi (flag argument) → tách thành 2 hàm.
- Không có side effect ẩn: hàm `get*`/`find*`/`is*` không ghi DB, không gửi message, không ghi cache.

## Hằng số và kiểu dữ liệu
- Không dùng magic number/string → đưa vào nơi khai báo hằng sẵn có của repo. Tìm kỹ trước khi tạo, **không tạo thêm nơi khai báo hằng trùng nghĩa**.
- Code mới dùng enum cho tập giá trị cố định thay vì chuỗi rời.

## Lỗi và null
- Không nuốt exception (catch rỗng, hoặc log rồi chạy tiếp mà không có lý do). Catch exception cụ thể. Chỉ catch exception chung ở biên (listener, job, lời gọi bên ngoài), và log kèm định danh tenant + id đối tượng + stacktrace.
- Chọn exception theo đúng HTTP status: lỗi do client/đầu vào → 4xx, lỗi hệ thống → 5xx. Kiểm tra class exception của repo trả status nào trước khi dùng.
- Trả về collection rỗng thay vì `null`.
- Không che lỗi bằng null-check khi chưa hiểu vì sao giá trị null.

## Cấu trúc
- Trùng lặp: logic cùng ý nghĩa xuất hiện lần thứ 3 → tách hàm; mới 2 lần thì cân nhắc. Không gộp 2 đoạn chỉ giống nhau về hình thức.
- Không gọi DB/API trong vòng lặp (N+1) → xử lý theo lô.
- Không trả entity/model DB ra API, dùng DTO.
- Tầng nhận request không chứa logic nghiệp vụ, không gọi thẳng tầng truy cập dữ liệu.
- Không để lại code comment-out, import thừa, biến không dùng, log debug, `TODO` không kèm mã ticket.
- Comment giải thích **vì sao** (quyết định, bẫy dữ liệu, số đo), không diễn lại code làm gì.
- Không import class từ package shaded/relocated của thư viện khác. Không viết tên package đầy đủ inline khi import được.

## Riêng Java
- So sánh wrapper (`Integer`, `Long`) bằng `Objects.equals`, không dùng `==` (sai khi giá trị > 127, NPE khi null). So sánh `BigDecimal` bằng `compareTo`.
- Class mới dùng constructor injection.
- Dùng `Optional` cho kết quả có thể không có. Không dùng `printStackTrace()`, `System.out`.
