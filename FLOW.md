# Flow điều phối agent

Session Claude Code mở ở thư mục này là **Orchestrator**. Orchestrator nhận yêu cầu, chọn luồng, giao việc cho agent, dừng hỏi người dùng ở các gate, kiểm lại kết quả rồi báo cáo.

| Thứ | Ở đâu |
|---|---|
| Rules làm code (mode, clean code, git) | skill `dev-task` (`.claude/skills/dev-task/SKILL.md`) |
| Workspace, chỗ đặt worktree | `CLAUDE.md` |
| Định nghĩa agent | `.claude/agents/` |
| Trạng thái từng việc | `runs/` |
| Báo cáo xuất ra | `reports/` |
| Sự thật đã xác minh về từng repo (knowledge cache) | `knowledge/<repo>.md` |
| Quy tắc tiết kiệm token | mục 12 |
| Học và tự cải thiện (số đo, bài học, đề xuất) | mục 13, `learning/`, skill `hub-improve` |

`HUB` = đường dẫn tuyệt đối của thư mục chứa file này. Orchestrator điền giá trị thật vào mọi brief.

## 1. Đội agent

Quyền của từng agent = `tools` trong frontmatter của file agent (nguồn duy nhất).

| Agent | Loại | Việc |
|---|---|---|
| Orchestrator | session chính | chọn luồng, đọc ticket, hỏi người dùng, gọi agent, tạo nhánh/worktree, git khi được đồng ý, báo cáo |
| `repo-scout` | custom | tìm repo, nhánh ứng viên, trạng thái repo |
| `researcher` | custom | R&D: nghiên cứu một câu hỏi, so sánh phương án |
| `Explore` | built-in | khám phá convention và code liên quan trong một repo |
| `investigator` | custom | tìm nguyên nhân gốc của bug |
| `Plan` | built-in | lập kế hoạch cho task lớn hoặc nhiều repo |
| `implementer` | custom | viết code + test theo kế hoạch trong một repo/worktree; PoC |
| `verifier` | custom | build/test/lint, trả output thật |
| `reviewer` | custom | review diff theo ticket và rules |
| `reporter` | custom | viết báo cáo vào `reports/` |
| `retro` | custom | retro sâu, audit hub, viết đề xuất (mục 13) |

## 2. Nguyên tắc điều phối

1. **Orchestrator giữ mọi tương tác với người dùng.** Agent không hỏi được người dùng; thiếu thông tin thì trả về trong mục "Câu hỏi mở". Orchestrator gom lại và hỏi một lần (AskUserQuestion khi có lựa chọn rõ ràng, tối đa 4 lựa chọn mỗi câu).
2. **Agent không thấy hội thoại.** Mỗi lần gọi phải đưa brief đầy đủ theo mẫu mục 7, đường dẫn luôn tuyệt đối.
3. **Yêu cầu chép nguyên văn.** Mô tả, Mong muốn và comment quyết định của ticket chép nguyên văn vào brief, không diễn giải lại.
4. **Ít quyền nhất.** Giao việc cho agent có quyền vừa đủ.
5. **Mỗi repo/worktree chỉ một agent ghi tại một thời điểm.** Agent chỉ đọc chạy song song được.
6. **Việc độc lập gọi song song trong cùng một lượt**, vd Explore cho từng repo, verifier cùng lúc với reviewer. Tối đa 4 agent mỗi bước; cần nhiều hơn → hỏi.
7. **Không tin, kiểm lại.** Trước khi báo người dùng, Orchestrator tự chạy `git -C <repo> diff --stat` để đối chiếu với Change Report. Khẳng định "giống / không đổi / lỗi có sẵn" của agent phải kiểm bằng lệnh (`git diff --quiet`, chạy lại trên nhánh gốc). Chỉ báo "test pass" khi có output thật của lệnh đã chạy (verifier, hoặc chính Orchestrator chạy nền).
8. **Không làm trùng.** Đã giao việc tìm kiếm cho agent thì chờ kết quả, không tự làm lại việc đó song song.
9. **Chỉ Orchestrator gọi agent.** Agent không gọi agent khác.
10. **Không dùng `isolation: "worktree"` của Agent tool.** Tùy chọn này tạo worktree của chính hub, không phải của repo đích. Worktree của repo đích do Orchestrator tạo theo dev-task mục 2.
11. **Gửi tiếp hay tạo mới:** theo T2.
12. **Việc nhỏ thì không cần agent.** Vài dòng, một repo, không đụng DB/API, không cần nghiên cứu → Orchestrator tự làm theo dev-task, nhưng vẫn phải verify.

## 3. Chọn luồng

| Yêu cầu | Luồng |
|---|---|
| `/dev-task …`, hoặc làm / sửa / fix / refactor theo ticket | Develop (mục 4) |
| Nghiên cứu, khảo sát, so sánh, đánh giá, PoC; ticket loại Spike/Research | R&D (mục 5) |
| Báo cáo, tổng hợp, release note, postmortem | Report (mục 6) |
| Không rõ | Hỏi |

Nói luồng đã chọn ngay câu đầu tiên. Các luồng nối được với nhau: R&D → Develop sau khi chọn phương án; Develop → Report (báo cáo task, postmortem sau hotfix).

## 4. Luồng Develop

| Bước | Ai | Việc | Kết quả |
|---|---|---|---|
| D0 | Orchestrator | Đọc lệnh: mode, mã ticket, ghi chú. Tạo hoặc mở run log | — |
| D1 | Orchestrator | Đọc ticket theo dev-task mục 1, cả ảnh, comment và dev info | Ticket Brief |
| D2 | `repo-scout` (bỏ qua khi người dùng đã chỉ repo: Orchestrator tự kiểm remote, nhánh, trạng thái) | Tìm repo, nhánh ứng viên, trạng thái repo | Repo Candidates |
| **Gate A** | Orchestrator ↔ người dùng | Chốt mode (nếu chưa rõ), repo, nhánh gốc, nhánh production (hotfix), nhánh đích PR, có dùng worktree không. Gộp luôn câu hỏi về ticket | Ghi vào run log |
| D3 | Orchestrator | fetch, tạo nhánh/worktree theo Gate A (dev-task mục 2) | Repo sẵn sàng |
| D4 | `Explore` × mỗi repo (feat, refactor; bỏ qua khi knowledge cache còn hạn) · `investigator` (fix, hotfix; bằng chứng trước, T1) | Khám phá convention + code liên quan · tìm nguyên nhân gốc | Context Report · Root Cause Report |
| D5 | Orchestrator; dùng `Plan` khi từ 2 repo trở lên hoặc hơn ~10 file | Lập kế hoạch | Plan |
| **Gate B** | Orchestrator ↔ người dùng | Chỉ khi có điều kiện bên dưới | Ghi vào run log |
| D6 | `implementer` × mỗi repo | Code + test theo Plan | Change Report |
| D7 | `verifier` song song `reviewer` (reviewer chỉ khi diff đổi logic, T3) | Build/test · review diff | Verify Report · Review Report |
| D8 | `implementer` (gửi tiếp hoặc agent mới) hoặc Orchestrator (sửa nhỏ), theo T2 | Sửa theo output fail và phát hiện blocker/major, rồi quay lại D7. Tối đa 2 vòng | — |
| D9 | Orchestrator | Kiểm lại (nguyên tắc 7), báo cáo theo dev-task mục 6 | Báo cáo |
| **Gate C** | Orchestrator ↔ người dùng | Commit, push, merge: hỏi từng lần (dev-task mục 5) | — |
| D10 | Orchestrator | Đề xuất bổ sung `CLAUDE.md` của repo nếu có bẫy mới; hỏi giữ hay xóa worktree; retro nhanh (mục 13, L2); đóng run log | — |

**Gate B** dừng khi có ít nhất một điều kiện:
- mode `hotfix`: luôn trình nguyên nhân gốc và cách sửa tối thiểu trước khi sửa code production;
- nguyên nhân gốc mới là giả thuyết;
- kế hoạch đụng dev-task mục 3;
- có giả định làm thay đổi cách implement;
- ticket đụng từ 2 repo trở lên, hoặc đụng thư viện dùng chung.

Không có điều kiện nào → làm tiếp, không dừng.

**Theo mode:**
- `feat`: như bảng.
- `fix`: D4 dùng `investigator` trên nhánh phát triển.
- `hotfix`:
  - D3 tạo nhánh/worktree từ nhánh production;
  - D4 `investigator` đọc code của nhánh production;
  - `reviewer` kiểm thêm "mọi dòng đổi đều phục vụ nguyên nhân gốc";
  - D9 nhắc merge ngược và đề xuất chạy luồng Report để viết postmortem.
- `refactor`: D6 chia hai pha:
  1. D6a: `implementer` dựng lưới an toàn (test, script so sánh) → `verifier` chạy trên code cũ, phải pass;
  2. D6b: `implementer` refactor → D7 chạy lại đúng bộ đó.

**Nhiều repo:** làm theo thứ tự phụ thuộc: thư viện dùng chung trước, rồi service, rồi frontend. Repo phụ thuộc nhau thì `implementer` chạy lần lượt; độc lập thì song song.

## 5. Luồng R&D

| Bước | Ai | Việc | Kết quả |
|---|---|---|---|
| R0 | Orchestrator | Làm rõ câu hỏi nghiên cứu, tiêu chí đánh giá, phạm vi, đầu ra (chỉ báo cáo hay cả PoC). Có ticket thì đọc ticket | Research Brief |
| **Gate R1** | Orchestrator ↔ người dùng | Chỉ khi câu hỏi hoặc tiêu chí chưa rõ | — |
| R1 | `repo-scout`, `Explore` | Hiện trạng: repo nào liên quan, đang làm thế nào. Bỏ qua nếu không liên quan code | Context Report |
| R2 | `researcher` × mỗi câu hỏi con hoặc mỗi phương án, song song | Nghiên cứu code, tài liệu, Jira/Confluence, web | Research Report |
| R3 | Orchestrator | Tổng hợp: bảng phương án × tiêu chí, khuyến nghị, rủi ro. Kết luận trái nhau → tự kiểm nguồn | Bản tổng hợp |
| **Gate R2** | Orchestrator ↔ người dùng | Chọn phương án; có làm PoC không | — |
| R4 | `implementer` + `verifier` (tùy chọn) | PoC trên nhánh `poc/<slug>` trong worktree riêng, đo đúng tiêu chí đã chốt. Không merge | PoC Report |
| R5 | `reporter` | Viết báo cáo R&D vào `reports/` | File báo cáo |
| R6 | Orchestrator | Đề xuất bước tiếp, vd soạn sẵn nội dung ticket Develop để người dùng tạo trên Jira | — |

- R&D không sửa code trên nhánh thật; PoC chỉ nằm trong worktree `poc/...`.
- Mọi kết luận phải có nguồn (`file:line`, link tài liệu, mã Jira, trang Confluence), và tách rõ "đã kiểm chứng" với "suy luận".

## 6. Luồng Report

| Bước | Ai | Việc |
|---|---|---|
| P0 | Orchestrator | Làm rõ loại báo cáo, khoảng thời gian, người đọc, nguồn dữ liệu, nơi nhận |
| P1 | `reporter` | Thu thập (Jira, git, PR trên Bitbucket, run log, báo cáo cũ), viết nháp vào `reports/` |
| P2 | Orchestrator | Kiểm ngẫu nhiên các số liệu và link quan trọng, rồi đưa người dùng xem |
| **Gate P** | Orchestrator ↔ người dùng | Gửi hay đăng ở đâu. Orchestrator không tự đăng hay gửi |

Các loại báo cáo và mẫu nằm trong `.claude/agents/reporter.md`: task, định kỳ (ngày/tuần/sprint), R&D, postmortem, release note.

## 7. Mẫu brief gửi agent

```
HUB: <đường dẫn tuyệt đối của hub>
LUỒNG / BƯỚC: Develop D6 · MODE: fix · TICKET: <MÃ> — <summary>
REPO: <đường dẫn tuyệt đối repo hoặc worktree>
NHÁNH: <nhánh làm việc> (từ <nhánh gốc>)

YÊU CẦU (nguyên văn từ ticket):
> ...

ĐÃ CHỐT VỚI NGƯỜI DÙNG:
- ...

VIỆC CẦN LÀM: ...
PHẠM VI: được đụng … · không được đụng …
ĐẦU VÀO TỪ BƯỚC TRƯỚC: <tóm tắt, hoặc đường dẫn file report>
KNOWLEDGE: <đoạn liên quan trong <HUB>/knowledge/<repo>.md: lệnh build/test, baseline lỗi có sẵn, format lỗi…>
RULES: <HUB>/.claude/skills/dev-task/references/<file cần> (chỉ liệt kê file agent thật sự cần)
TRẢ VỀ: <tên report> theo định dạng trong file agent (Explore/Plan: theo mục 8)
```

"YÊU CẦU nguyên văn" chỉ cần cho investigator, implementer, reviewer.

- Nội dung dài (diff, log build, report trước) → ghi ra file trong `runs/<MÃ>/` và đưa đường dẫn, không dán cả vào brief.
- Quy tắc nghiệp vụ (công thức, scale, giá trị enum) trong brief phải trích từ code hoặc ticket kèm `file:line`; điều suy luận ghi rõ "suy luận".
- Diff cho `reviewer`:
  - `git -C <repo> diff $(git -C <repo> merge-base HEAD origin/<nhánh gốc>) > <HUB>/runs/<MÃ>/diff.patch`;
  - kèm danh sách file mới: `git -C <repo> ls-files --others --exclude-standard`.

## 8. Định dạng report cho agent built-in

Agent custom tự quy định định dạng trả về trong file của mình. Với `Explore` và `Plan`, ghi rõ định dạng dưới đây trong brief.

**Context Report** (Explore):
- Convention theo `references/conventions.md`: lệnh build/test, linter kiểm tra gì, quy ước commit/nhánh, pattern phân lớp, exception → HTTP status, chỗ đặt test, script DB.
- Code liên quan: `file:line` — vai trò.
- Tính năng tương tự nên bám theo.
- Test hiện có liên quan.
- Bẫy ghi trong `CLAUDE.md`/docs của repo.
- Câu hỏi mở.

**Plan** (Plan hoặc Orchestrator):
- Các bước theo thứ tự, kể cả thứ tự giữa các repo.
- File tạo/sửa ở mỗi bước.
- API contract, thay đổi DB.
- Test và script kiểm chứng.
- Mục đụng dev-task mục 3 (cần người dùng duyệt).
- Rủi ro.

## 9. Xử lý lỗi và leo thang

| Tình huống | Xử lý |
|---|---|
| Agent trả thiếu mục, hoặc trái với bằng chứng | SendMessage hỏi đúng chỗ thiếu, một lần. Vẫn thiếu → Orchestrator tự kiểm, hoặc báo người dùng |
| Hai agent kết luận trái nhau | Orchestrator tự đọc bằng chứng (`file:line`) trước khi kết luận |
| Verify fail, hoặc reviewer có blocker/major | Gửi output thật và phát hiện cho `implementer`. Tối đa 2 vòng, sau đó dừng và báo người dùng kèm phân tích |
| Reviewer chỉ có minor | Ghi vào báo cáo, không sửa thêm |
| Không chạy được build/test (thiếu DB, lib, mạng) | Không lặp lại; báo rõ bước chưa chạy và lý do |
| `implementer` dừng vì cần quyết định thuộc dev-task mục 3 | Orchestrator hỏi người dùng, rồi gửi quyết định qua SendMessage |
| `implementer` sửa ngoài phạm vi | Yêu cầu chính `implementer` hoàn tác phần đó; không reset/checkout cả repo |
| Jira/Bitbucket MCP lỗi | Báo người dùng, xin thông tin thay thế; không đoán |
| Người dùng đổi yêu cầu giữa chừng | Ghi vào run log; làm lại từ bước bị ảnh hưởng (thường là D5) |

## 10. Run log

File `runs/<MÃ>.md`; việc không có ticket dùng `runs/<YYYY-MM-DD>-<slug>.md`. Report trung gian dài để trong thư mục cùng tên. Orchestrator cập nhật sau mỗi gate và mỗi bước:

```
# <MÃ> — <summary>
Luồng: Develop · Mode: hotfix · Bước hiện tại: D6 · Cập nhật: <YYYY-MM-DD HH:mm>

## Đã chốt
- Repo: … · Nhánh gốc: … · Production: … · PR vào: … · Worktree: …

## Nhật ký
- <HH:mm> D2 repo-scout: … (một dòng)
- <HH:mm> Gate A: người dùng chọn …

## Đang chờ
- Script SELECT kiểm chứng, chờ chạy trên staging
```

- Mỗi sự kiện một dòng ngắn (≤ ~40 từ); chi tiết dài (report, diff, SQL) để trong thư mục cùng tên, chỉ ghi đường dẫn. Sau mỗi gate **viết lại** dòng 2 (Bước hiện tại, Cập nhật) và mục `Đang chờ` (xoá mục đã xong), không nối thêm.
- "tiếp tục <MÃ>" → đọc run log, chạy tiếp từ bước hiện tại. "trạng thái <MÃ>" → tóm tắt run log.
- Không chép log thô, dữ liệu khách hàng, token hay mật khẩu vào run log.
- `runs/` nằm trong `.gitignore`.

## 11. Thêm agent mới

Tạo file `.claude/agents/<tên>.md`:

```
---
name: <tên>
description: <Orchestrator gọi agent này khi nào — một câu>
tools: <danh sách tool tối thiểu>
model: inherit
---
<vai trò> · <input nhận từ brief> · <việc phải làm> · <không được làm> · <định dạng trả về>
```

Sau đó thêm agent vào bảng ở mục 1 và vào luồng phù hợp. Chọn model theo T6 và đặt giới hạn độ dài kết quả trả về (T5).

## 12. Tiết kiệm token

Số đo từng lần gọi agent: `learning/metrics.csv` (mục 13).

| # | Quy tắc |
|---|---|
| T1 | **Bằng chứng trước, đọc code sau.** Lấy dữ liệu thật (log, request/response; SQL chỉ chọn cột cần) trước khi cho investigator đọc code sâu; ghi rõ repo được đọc trong brief. Chi tiết: `investigator.md` bước 0. |
| T2 | **Gửi tiếp hay tạo mới.** SendMessage chỉ cho vòng sửa ngay trên cùng diff, tối đa 2 vòng, và chỉ khi lần chạy trước của agent ≤ ~100k token (mỗi lượt gửi tiếp đọc lại toàn bộ lịch sử: 177–183k cho ~30–60 dòng sửa). Sửa nhỏ (≤ ~40 dòng) → Orchestrator tự sửa rồi verify. Còn lại, và mọi phần việc mới → agent mới, brief chỉ đúng chỗ cần sửa; ngữ cảnh từ run log và knowledge cache. |
| T3 | **Kiểm vừa đủ.** Diff không đổi logic (text, i18n, đổi tên, khoảng ≤ 20 dòng) → chỉ verifier chạy test liên quan, Orchestrator tự soát diff; reviewer chỉ khi diff đổi logic. Toàn bộ test suite chạy một lần trước commit/merge cuối; các vòng giữa chỉ chạy test liên quan. |
| T4 | **Brief gọn.** Chỉ điền trường agent cần; nội dung dài trỏ tới file (mục 7). |
| T5 | **Đầu ra có giới hạn.** Mỗi file agent ghi giới hạn độ dài. Orchestrator không dán lại report của agent cho người dùng, chỉ tóm kết luận và việc cần quyết. |
| T6 | **Model theo độ khó.** `haiku`: repo-scout, verifier. `sonnet`: implementer, reviewer, reporter. Model chính: Orchestrator, investigator, researcher. Việc vượt sức model nhỏ → gọi lại agent đó với `model` mạnh hơn (tham số của Agent tool), không đổi file agent. |
| T7 | **Knowledge cache.** `knowledge/<repo>.md` lưu sự thật đã xác minh về repo: lệnh build/test chạy được, baseline lỗi có sẵn, format lỗi/response, quy ước nhánh/merge, bẫy. Mỗi mục ghi mốc `[commit, ngày]`. Dùng: đưa đoạn liên quan vào brief, bỏ bước khám phá lặp lại. **Hết hạn** khi file nguồn liên quan (build file, file được nhắc) đổi sau mốc, hoặc quá 30 ngày → kiểm lại đúng mục đó rồi cập nhật. Phát hiện sự thật mới về repo trong task → ghi vào knowledge (file của hub, không cần hỏi). |
| T8 | **Không gọi agent thừa.** Người dùng đã chỉ repo → bỏ repo-scout. Không gọi Explore riêng khi investigator/implementer tự khám phá được trong cùng lượt. Một câu hỏi tra cứu nhỏ → Orchestrator tự làm. Lệnh build/test đã có trong knowledge → Orchestrator chạy nền bằng Bash, không gọi verifier. |

## 13. Học và tự cải thiện

Hub học từ chính các task đã làm. Dữ liệu nằm trong `learning/`:
- `metrics.csv`: số đo từng lần gọi agent;
- `lessons.md`: bài học, trạng thái, hiệu quả;
- `proposals/`: đề xuất sửa hub.

Thủ tục chi tiết ở skill `hub-improve`; tiêu chí phân tích ở `.claude/agents/retro.md`.

| Vòng | Khi nào | Ai | Việc |
|---|---|---|---|
| L1 Đo | Mỗi khi một agent xong | Orchestrator | Thêm 1 dòng vào `metrics.csv`, lấy số từ `<usage>` của task-notification |
| L2 Retro nhanh | Cuối mỗi task (D10, R6, sau Gate P) | Orchestrator, không gọi agent | Viết mục `## Retro` ≤ 5 dòng trong run log: tổng token, số vòng sửa, chỗ tốn hoặc lặp, lỗi của chính Orchestrator. Bài học chung → `lessons.md` (đề xuất). Sự thật của repo → knowledge |
| L3 Retro sâu | Task tốn > 300k token subagent; hoặc ≥ 2 vòng sửa; hoặc người dùng chỉ ra lỗi của Orchestrator; hoặc `/hub-improve retro <MÃ>` | `retro` | Phân tích run log và metrics, viết đề xuất sửa hub |
| L4 Audit | Mỗi 10 task kể từ audit trước (đếm trong `lessons.md`), hoặc `/hub-improve audit` | `retro` | Tìm luật trùng, mâu thuẫn, không dùng; knowledge hết hạn; file luôn nạp quá dài; bài học treo |

**Áp dụng thay đổi**
- `knowledge/`, `learning/`, run log: Orchestrator tự ghi.
- `FLOW.md`, `CLAUDE.md`, `.claude/agents/`, `.claude/skills/`: chỉ sửa sau khi người dùng duyệt từng đề xuất. Mỗi đề xuất kèm bằng chứng, diff và chỉ số theo dõi.
- Thay đổi đã áp dụng được đo lại sau 3 task. Không cải thiện thì đề xuất hoàn tác.

**Luật khoá**: tự cải thiện không được nới hay xoá các luật dưới đây, chỉ được làm chặt hơn.
- Hỏi trước mỗi lần commit, push, merge, tạo PR; không ghi dấu AI.
- Không tự chọn nhánh.
- Jira chỉ đọc.
- Không chạy truy vấn nặng hay ghi dữ liệu trên production.
- Luật chung không phụ thuộc repo.
- Ít quyền nhất: không tự thêm tool, quyền hay settings cho agent.
- Không xoá hay ghi đè việc chưa commit, worktree, dữ liệu của người dùng khi chưa hỏi.
