# Convention, tìm code, verify

Đọc khi: chuẩn bị code (implementer), chạy build/test (verifier), hoặc làm task một mình.

## Khám phá convention của repo trước khi code

Brief đã kèm đoạn knowledge cache của repo (`<HUB>/knowledge/<repo>.md`) → dùng luôn, chỉ kiểm lại dòng nào nghi đã cũ. Còn lại tự tìm từ chính repo đích:

| Cần biết | Tìm ở đâu |
|---|---|
| Rules, bẫy domain, lệnh build riêng | `CLAUDE.md`, `README`, `docs/` |
| Lệnh build / test / lint | `pom.xml`, `build.gradle`, `package.json`, `Makefile`, CI config |
| Linter thực sự kiểm tra gì | file config của linter (có linter chỉ kiểm tra một luật hẹp) |
| Quy ước commit message | `git log --oneline -30` |
| Quy ước tên nhánh | `git branch -a`, các merge commit gần đây |
| Phân lớp, response wrapper, đặt tên, logging, kiểu inject | 1–2 tính năng tương tự đã có trong repo |
| Class exception và HTTP status mỗi class trả về | thư mục errors/exception + exception handler chung |
| Framework test, chỗ đặt test | thư mục test hiện có |
| Script DB: chỗ đặt, định dạng, migration tool | thư mục changelog/migration hiện có |

Ghi lại các convention đã tìm được (3–6 dòng) trong phần kế hoạch. Có hai cách làm cùng tồn tại trong repo → theo cách ở code mới hơn hoặc phổ biến hơn, và nói rõ đã chọn cách nào. Không chắc → hỏi.

Tái dùng util, hằng số, DTO sẵn có. **Không tự nghĩ công thức nghiệp vụ mới** khi repo đã có.

## Công cụ tìm kiếm code

- Có code-review-graph thì dùng trước (`semantic_search_nodes_tool`, `query_graph_tool` callers_of/tests_for, `get_impact_radius_tool`), Grep/Read sau.
- **Graph/semantic search không đủ làm bằng chứng "không có ai gọi"**: lời gọi qua interface, reflection, chuỗi SQL, config, framework annotation thường không được resolve. Trước khi đổi tên, đổi signature hay xóa thứ gì phải Grep thêm tên đó trong toàn repo, kể cả file SQL/YAML/properties.
- Kiểm tra graph còn mới không (`list_graph_stats_tool`). Graph cũ hơn HEAD → cập nhật graph hoặc dựa vào Grep.

## Verify

Dùng đúng lệnh build/test/lint của repo đích, chạy trong thư mục repo/worktree đích, theo thứ tự: build → test liên quan → lint. Bước nào fail thì sửa rồi mới sang bước sau. Ví dụ:

| Build tool | Build | Chạy test cụ thể |
|---|---|---|
| Maven | `./mvnw -ntp compile -q` | `./mvnw -ntp test -Dtest=<Class>` |
| Gradle | `./gradlew compileJava` | `./gradlew test --tests <Class>` |
| npm | `npm run build` | `npm test -- <pattern>` |

Windows PowerShell dùng `mvnw.cmd` / `gradlew.bat`.

- Vòng sửa giữa chừng: chỉ chạy test liên quan. Toàn bộ test suite chạy **một lần** trước commit/merge cuối, so với baseline lỗi có sẵn (lấy từ knowledge cache nếu có).
- Bước nào không chạy được (thiếu DB, thiếu lib nội bộ, lỗi mạng) → nói rõ bước nào và vì sao. **Không bao giờ báo "đã pass" cho bước chưa chạy.**
- Tự đọc lại `git diff`:
  - soát theo `clean-code.md`;
  - không còn log debug, code comment-out, file ngoài phạm vi task.
- Đối chiếu lại từng ý AC của ticket: đã đạt, chưa đạt, hay không áp dụng.

## Worktree

- Worktree mới của repo có hook (husky, pre-commit…) hoặc cần dependency: chạy bước chuẩn bị của repo (script `prepare`, cài hook) trước khi commit; lệnh cụ thể lấy từ knowledge của repo.
- Worktree có link dependency (junction/symlink): gỡ link trước (chỉ gỡ link, không xoá đệ quy), rồi mới xoá worktree.
- Worktree tạm Orchestrator tạo cho việc merge: nói trong câu hỏi merge/push rằng sẽ gỡ nó sau khi xong; chỉ gỡ khi người dùng đồng ý câu hỏi đó. Worktree của task: không tự xoá (dev-task mục 2).
