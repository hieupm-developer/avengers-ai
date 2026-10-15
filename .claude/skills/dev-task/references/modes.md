# Quy trình theo mode

Chỉ đọc phần của mode đang làm. "SKILL.md mục N" là các mục trong file `SKILL.md` của skill này.

## feat

1. **Làm rõ**: dùng bản tóm tắt ticket (SKILL.md mục 1), liệt kê giả định. Ticket không có AC rõ ràng → tự đề xuất AC và đánh dấu "đề xuất". **Có giả định nào làm thay đổi cách implement thì dừng lại**, gửi tóm tắt + kế hoạch cho người dùng xác nhận trước khi code. Có thể gộp vào câu hỏi repo/nhánh (SKILL.md mục 2).
2. **Khảo sát**: khám phá convention (`conventions.md`); tìm tính năng tương tự để bám pattern; liệt kê thứ sẽ tái dùng.
3. **Kế hoạch**:
   - các file tạo/sửa;
   - API contract (endpoint, request, response mẫu);
   - thay đổi DB;
   - rủi ro hiệu năng.

   Kế hoạch đụng SKILL.md mục 3 → dừng lại hỏi.
4. **Implement** từ trong ra ngoài: hằng/DTO → truy cập dữ liệu → nghiệp vụ → API.
5. **Test**:
   - Logic trong code (tính toán, mapping, validation, gom nhóm) → unit test theo framework của repo. Case biên: rỗng, null, chia 0, sai định dạng đầu vào.
   - Logic nằm trong SQL (lọc tenant, không double count, lọc trạng thái) mà repo không có test chạy với DB → viết **script SELECT kiểm chứng** kèm kết quả mong đợi, để người dùng chạy trên dev/staging.
6. **Verify** (`conventions.md`).
7. Feat có API mới cho client khác dùng → ghi hợp đồng API vào docs theo cách repo đang làm (hỏi nếu repo chưa có chỗ nào).

**Done khi**: đạt hết AC, test pass, script kiểm chứng đã viết, không còn TODO, đã báo cáo.

## fix / hotfix

**Nguyên tắc**: sửa nhỏ nhất và sửa đúng nguyên nhân gốc. Không refactor, không format lại, không đổi tên, không thêm dependency, không đổi schema (trừ khi không còn cách nào khác → hỏi).

Khác nhau giữa hai mode:
- `hotfix`: tách nhánh từ nhánh production đã xác nhận ở SKILL.md mục 2, và phải merge ngược về nhánh đang phát triển.
- `fix`: tách nhánh từ nhánh phát triển đã xác nhận ở SKILL.md mục 2, không có bước merge ngược.

1. **Bằng chứng trước, đọc code sau**: lấy từ ticket trước (mô tả, ảnh lỗi, comment): log, stacktrace, tenant, id đối tượng, thời điểm. Lỗi đến từ hệ thống ngoài hoặc DB → xin dữ liệu thật (log, bản ghi request/response) **trước** khi đọc code sâu; câu SQL chỉ lấy đúng cột/trường cần. Ticket vẫn thiếu thì hỏi đúng thứ đang thiếu, không đoán.
2. **Nguyên nhân gốc**: chỉ ra `file:line` và chuỗi nguyên nhân. Ghi rõ đâu là **đã xác nhận**, đâu là **giả thuyết**. Không sửa triệu chứng (try-catch nuốt lỗi, null-check che lỗi). Điều tra không ra nguyên nhân → báo những gì đã loại trừ, không sửa mò.
   - Điều tra lỗi production thì đọc code ở **nhánh production** (`git -C <repo> show origin/<nhánh>:<file>`), không đọc ở nhánh đang checkout.
3. **Phạm vi ảnh hưởng**: tenant/khoảng thời gian bị ảnh hưởng; dữ liệu trong DB đã sai chưa. Nếu đã sai → viết script sửa dữ liệu riêng (SELECT kiểm tra trước, UPDATE trong transaction, idempotent) và **không tự chạy**.
4. **Sửa**.
5. **Regression test** tái hiện đúng lỗi (fail trước khi sửa, pass sau khi sửa) nếu lỗi nằm ở logic code. Lỗi nằm ở SQL → viết script SELECT tái hiện cho người dùng chạy.
6. **Verify** (`conventions.md`).
7. Hotfix: nhắc người dùng merge ngược về nhánh đang phát triển sau khi lên production.

## refactor

**Nguyên tắc**: hành vi bên ngoài không đổi. Giữ nguyên:
- API: endpoint, request/response, HTTP status;
- schema DB;
- format message queue;
- cache key và cấu trúc value;
- định dạng log mà monitoring/tra cứu đang dựa vào.

Nếu chính mục tiêu refactor là đổi một trong các thứ này → đưa phương án tương thích ngược và hỏi.

1. **Mục tiêu**: lấy từ ticket — vì sao refactor, và sau refactor được gì (bớt trùng lặp, tách lớp, query nhanh hơn — có số đo).
2. **Ảnh hưởng**: graph (impact radius, callers, tests) **cộng Grep** (`conventions.md`). Liệt kê toàn bộ nơi gọi. Thư viện dùng chung thì tìm cả ở các repo khác trong workspace đang dùng nó.
3. **Lưới an toàn**, dựng **trước khi sửa**:
   - Logic code chưa có test → viết characterization test chốt hành vi hiện tại.
   - SQL (đổi điều kiện lọc, đổi cột ngày, đổi JOIN) → viết script chạy query cũ và query mới trên cùng tenant + khoảng ngày, so sánh số dòng, tổng số liệu và `EXCEPT` theo cả 2 chiều, để người dùng chạy trên staging. Kết quả phải giống nhau.
   - Hành vi trông như bug vẫn giữ nguyên và ghi vào "Phát hiện thêm".
4. **Từng bước nhỏ**, bước nào cũng build được. Đổi tên dùng công cụ rename có xem trước (vd `refactor_tool` mode rename).
5. **Xóa code chết** chỉ khi chắc chắn không còn ai gọi. Kiểm tra bằng Grep chứ không chỉ graph, gồm cả reflection, job định kỳ, listener, chuỗi SQL, config, service khác gọi qua API.
6. **Verify** (`conventions.md`): test cũ và test mới đều pass.
