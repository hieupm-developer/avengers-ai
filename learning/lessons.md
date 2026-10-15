# Bài học và thay đổi của hub

Quy trình: FLOW.md mục 13 · Thủ tục: skill `hub-improve`.
Audit gần nhất: 2026-10-15 (`proposals/2026-10-15-audit-hub.md`) · Task kể từ audit: 2 (webhook-transhub-sang-easyposwebhook, EPB-22980)

Bài học chỉ ghi ở dạng chung, không gắn repo. Sự thật riêng của repo ghi vào `knowledge/<repo>.md`.

## Đang đề xuất

| ID | Ngày · nguồn | Bài học | Đặt vào | Chỉ số theo dõi |
|---|---|---|---|---|
| L-018 | 2026-10-15 · EPB-22860, người dùng chỉ ra | Gỡ conflict cho PR mà nhánh nguồn đã từng merge vào nhánh tích hợp (dev/test): chạy `merge-tree` mô phỏng cả chuỗi tiếp theo (PR → đích → nhánh tích hợp). Nếu còn conflict thì đề xuất gỡ luôn phía nhánh tích hợp ngay trong cùng gate, đừng chỉ cảnh báo. | dev-task mục 5 (cạnh L-008) | Lần người dùng phải nhắc gỡ conflict phía sau · mốc 1 |
| L-017 | 2026-10-15 · xoá worktree EPB-21073 | Nhiều session dùng chung workspace. Worktree task có lúc bị session khác dùng lại cho nhánh khác, và worktree tạm trùng tên (`<repo>-<MÃ>-dev`) bị tạo lại. Trước khi xoá worktree: kiểm nhánh/HEAD hiện tại, đã push chưa, đã nằm trong nhánh đích chưa, `git status` sạch chưa. Đặt tên worktree tạm có hậu tố giờ (`-dev-HHmm`) để tránh trùng giữa các session. | references/conventions.md mục Worktree | Lần xoá nhầm hoặc trùng worktree · mốc 0 (đã bắt kịp trước khi xoá) |

## Đang thử (đã áp dụng, chờ đo)

Áp dụng ngày 2026-10-15, người dùng duyệt. Đo lại sau 3 task, tức ở audit hoặc retro sau task thứ 3.

| ID | Thay đổi | Ở đâu | Chỉ số · mốc trước |
|---|---|---|---|
| L-001 | Chỉ gửi tiếp khi agent ≤ ~100k token. Sửa ≤ ~40 dòng thì Orchestrator tự sửa; còn lại tạo agent mới | FLOW T2, D8, nguyên tắc 11 | Token mỗi dòng `round` > 1 · mốc 177–183k/lượt (EPB-21073) |
| L-002 | Giải conflict theo khối; cấm `checkout --theirs/--ours <file>` | dev-task mục 5 | Lỗi merge phát hiện sau · mốc 1 (EPB-21073) |
| L-003 | Quy tắc nghiệp vụ trong brief phải có `file:line`; suy luận thì ghi rõ | FLOW mục 7 | Vòng sửa do brief sai · mốc 1 |
| L-004 + L-006 | Kiểm khẳng định của agent bằng lệnh. Test đã biết lệnh thì Orchestrator chạy nền, không gọi verifier | FLOW nguyên tắc 7, T8 | Số lần gọi verifier mỗi task; khẳng định sai bị bắt · mốc 0 verifier ở EPB-21073 |
| L-005 | Field định danh từ client phải lấy từ token hoặc đối chiếu bản ghi gốc | references/data-pitfalls.md | Major tenant của reviewer · mốc 1 |
| L-007 | Worktree: chạy bước chuẩn bị hook; gỡ junction trước khi xoá; worktree tạm chỉ gỡ khi người dùng đã đồng ý | references/conventions.md | Commit lỗi do hook · mốc 1 |
| L-008 | Chạy `merge-tree` và báo trước conflict khi hỏi merge | dev-task mục 5 | Hỏi lại do conflict bất ngờ |
| L-009 | Bỏ 2 sơ đồ mermaid (P-1) | FLOW mục 1, 4 | Số từ FLOW · mốc 3.615 → 3.493 sau cả đợt |
| L-010 | Mỗi luật một nơi chuẩn: nguyên tắc 11, T1, T4, đoạn số đo (P-2) | FLOW mục 2, 7, 12 | Số từ FLOW |
| L-012 | Sau mỗi gate viết lại dòng trạng thái và "Đang chờ" của run log (P-4) | FLOW mục 10 | Dòng 2 của run log khớp trạng thái thật · mốc 0/3 |
| L-013 | Metrics: mỗi lần gọi một dòng, không gộp, không ước tính (P-5) | hub-improve "Ghi số đo" | Tổng CSV ≈ run log (lệch < 5%) |
| L-014 | Bảng agent bỏ cột quyền; quyền chỉ ghi ở frontmatter (P-6) | FLOW mục 1 | Số từ FLOW |
| L-016 | Knowledge: bỏ quyền push một lần ghi như quyền lâu dài; sự thật FE về đúng file FE; cập nhật baseline (P-8, Orchestrator tự ghi) | knowledge/* | Mục sai chỗ hoặc thiếu mốc = 0 |

## Đã chốt

## Đã bỏ

| ID | Ngày | Bài học | Lý do |
|---|---|---|---|
| L-011 | 2026-10-15 | Gọn mục 11 "Thêm agent mới" và Context Report (P-3) | Người dùng không chọn |
| L-015 | 2026-10-15 | D5: chỉ gọi `Plan` khi thứ tự phụ thuộc chưa rõ (P-7) | Người dùng không chọn; giữ luật cũ, đo số lần gọi Plan ở audit sau |
