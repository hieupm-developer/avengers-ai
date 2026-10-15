# avengers-ai

Nơi điều phối: đứng ở đây để làm việc (develop, R&D, report) trên các repo trong workspace bên dưới, bằng một đội agent. Thư mục này không chứa code sản phẩm.

## Điều phối agent

Session mở ở đây là **Orchestrator**. Nhận bất kỳ yêu cầu làm việc nào (develop, R&D, report) → **đọc `FLOW.md` trước**, rồi làm theo flow đó.

- Agent: `.claude/agents/` · Run log: `runs/` (không commit) · Báo cáo: `reports/`.
- Knowledge cache: `knowledge/<repo>.md` — đọc trước khi giao việc trên repo đó, đưa đoạn liên quan vào brief; cập nhật khi phát hiện sự thật mới (FLOW.md mục 12).
- "tiếp tục <MÃ>" hoặc "trạng thái <MÃ>" → đọc `runs/<MÃ>.md`.
- Tự cải thiện: `learning/` (số đo, bài học, đề xuất), lệnh `/hub-improve` (FLOW.md mục 13).

## Workspace

- Thư mục chứa các repo: `C:\SoftDreams\Project`. Repo nằm ngay bên trong hoặc lồng 1 cấp (vd `CIC\*`, `Chat-Bot\*`).
- `C:\SoftDreams\Project\_review_worktrees\` chứa git worktree dùng để review PR, không phải repo riêng. Không sửa code trong đó khi làm task.
- Worktree cho task (khi cần): `C:\SoftDreams\Project\_task_worktrees\<repo>-<MÃ-TICKET>`.

## Làm task code

- Gọi `/dev-task [mode] <MÃ-TICKET>`.
- Mỗi repo có nhánh production và nhánh phát triển riêng, không có quy ước chung giữa các repo → **luôn hỏi tôi**:
  - cần sửa trên nhánh nào;
  - nhánh nào là production;
  - PR vào nhánh nào.
- Lệnh build/test/git chạy trong thư mục repo đích, không chạy ở đây.
