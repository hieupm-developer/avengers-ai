# Nguyên tắc SOLID

Đọc khi: thiết kế lớp/service mới, tách hoặc gộp lớp, refactor, hoặc review diff có thêm abstraction.

Áp dụng **vừa đủ**:
- SOLID dùng để giảm chi phí khi thay đổi, không phải để thêm tầng.
- Chưa có nhu cầu thay đổi thật thì không tạo interface hay abstraction "phòng sau này". Đó là YAGNI.
- Theo pattern của repo đích trước (`conventions.md`). Repo không dùng một nguyên tắc nào đó thì không ép vào trong một task fix/hotfix.

## S — Single Responsibility (một lý do để thay đổi)

- Mỗi lớp/hàm chỉ phục vụ **một tác nhân thay đổi**. Ví dụ: luật nghiệp vụ, định dạng output, cách lưu trữ, cách gọi API ngoài là 4 tác nhân khác nhau.
- Dấu hiệu vi phạm:
  - tên lớp có "And" / "Manager" / "Util" chung chung;
  - một service vừa gọi HTTP, vừa map DTO, vừa tính tiền, vừa ghi log nghiệp vụ;
  - một thay đổi nhỏ bắt sửa nhiều chỗ không liên quan trong cùng file.
- Cách làm:
  - tách theo trách nhiệm: Controller (HTTP), Service (nghiệp vụ), Repository (dữ liệu), Client (hệ thống ngoài), Mapper;
  - hàm dài → tách hàm con có tên nói rõ việc.
- Không tách quá nhỏ: 3 dòng dùng một lần thì không cần lớp riêng.

## O — Open/Closed (mở để mở rộng, đóng để sửa)

- Thêm một biến thể mới (hãng vận chuyển, sàn TMĐT, loại hoá đơn…) nên là **thêm code mới**, không phải sửa `if/else`/`switch` rải ở nhiều nơi.
- Cách làm:
  - strategy hoặc registry theo khoá, ví dụ một registry tra processor theo tên platform;
  - cấu hình thay cho hằng cứng khi giá trị thay đổi theo môi trường.
- Dấu hiệu vi phạm: cùng một `switch (platform)` lặp ở ≥ 2 nơi; thêm một hãng phải sửa 5 file.
- Chỉ áp dụng khi đã có (hoặc chắc chắn sắp có) ≥ 2 biến thể.

## L — Liskov Substitution (lớp con thay được lớp cha)

- Lớp con/implementation phải dùng được ở mọi chỗ đang dùng lớp cha/interface mà **không đổi hành vi mong đợi**.
- Dấu hiệu vi phạm:
  - implementation ném `UnsupportedOperationException` cho method của interface;
  - lớp con chặt điều kiện đầu vào hoặc nới điều kiện đầu ra (trả null khi cha hứa không null);
  - caller phải `instanceof` để xử lý riêng.
- Cách làm: interface nhỏ hơn (xem I), hoặc dùng composition thay cho kế thừa.

## I — Interface Segregation (interface nhỏ, đúng nhu cầu)

- Client không bị buộc phụ thuộc vào method nó không dùng.
- Dấu hiệu vi phạm: interface 20 method mà mỗi implementation chỉ làm thật 5 method, còn lại để trống hoặc ném lỗi.
- Cách làm: tách theo vai trò, ví dụ `Pricing`, `Tracking`, `Webhook` thay cho một `DeliveryPartner` ôm hết.
- Không tách interface khi chỉ có một implementation và một client.

## D — Dependency Inversion (phụ thuộc vào abstraction)

- Nghiệp vụ không phụ thuộc trực tiếp vào chi tiết hạ tầng (HTTP client cụ thể, Kafka, Redis, SDK của sàn). Nó phụ thuộc vào interface do tầng nghiệp vụ định nghĩa.
- Cách làm:
  - inject qua constructor, không `new` dependency trong lớp nghiệp vụ;
  - không gọi static/singleton ẩn (`Common.getX()` đọc context toàn cục) ở nơi cần test;
  - truyền giá trị vào thay vì đọc từ bên trong.
- Lợi ích kiểm chứng được: test nghiệp vụ bằng mock/fake, không cần DB, Kafka hay API thật. Đây cũng là lý do repo không có mockito-inline/`mockStatic` vẫn test được.

## Khi review / viết code

| Câu hỏi | Nếu "có" |
|---|---|
| Lớp này thay đổi vì ≥ 2 lý do khác nhau? | Cân nhắc tách (S) |
| Thêm một loại mới phải sửa nhiều `switch`/`if` có sẵn? | Registry / strategy (O) |
| Có implementation "không hỗ trợ" method của interface? | Tách interface hoặc bỏ kế thừa (L, I) |
| Nghiệp vụ `new` client/kết nối hạ tầng, hoặc gọi static khó mock? | Inject qua constructor (D) |
| Abstraction mới chỉ có 1 implementation, không có kế hoạch thêm? | Bỏ abstraction, viết thẳng (YAGNI) |

- Fix/hotfix: **không** refactor theo SOLID lẫn vào. Ghi vào "Phát hiện thêm" (dev-task mục 0).
- Refactor theo SOLID: theo `modes.md` phần refactor, nghĩa là dựng lưới an toàn trước rồi đi từng bước nhỏ.
