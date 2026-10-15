---
name: hub-improve
description: Cải thiện chính hub avengers-ai. Gồm: xem số đo token và chi phí agent, retro sau một task, audit tìm luật trùng/thừa/mâu thuẫn và chỗ tốn token, áp dụng các đề xuất đã duyệt, đo hiệu quả. Dùng khi người dùng gọi /hub-improve, hoặc hỏi về tối ưu token, điểm thừa, bài học, việc agent tự học hay tự nâng cấp.
argument-hint: "[stats | retro <MÃ> | audit | apply]"
---

# Hub improve

Yêu cầu: $ARGUMENTS

Quy trình và luật khoá: `FLOW.md` mục 13. Tiêu chí phân tích: `.claude/agents/retro.md`. Dữ liệu: `learning/`.

## Chế độ

| Lệnh | Việc |
|---|---|
| (trống) | `stats`, kèm danh sách đề xuất đang chờ, rồi gợi ý bước tiếp |
| `stats` | Orchestrator tự đọc `learning/metrics.csv`, không gọi agent. Xuất các bảng: token theo task; theo agent/model (tổng, trung vị); top 5 lần gọi tốn nhất; tổng token các lượt gửi tiếp (`round` > 1); số task kể từ audit gần nhất |
| `retro <MÃ>` | Kiểm metrics của task đã đủ chưa (thiếu thì bổ sung từ run log). Gọi agent `retro` với `MODE: task`. Đọc file đề xuất rồi chuyển sang `apply` |
| `audit` | Gọi agent `retro` với `MODE: audit`. Cập nhật dòng "Audit gần nhất" trong `lessons.md` rồi chuyển sang `apply` |
| `apply` | Trình và áp dụng đề xuất (bên dưới) |

Brief cho `retro` theo FLOW mục 7, gồm: `HUB`, `MODE`, mã ticket và đường dẫn run log (với `task`), phạm vi.

## Áp dụng đề xuất

1. Đọc file đề xuất mới nhất trong `learning/proposals/` và bảng "Đang đề xuất" trong `lessons.md`. Tự kiểm bằng chứng của từng đề xuất (mở đúng `file:dòng`, dòng metrics). Bằng chứng sai thì loại đề xuất đó, ghi lý do.
2. Loại ngay các đề xuất nới luật khoá (FLOW mục 13).
3. Trình người dùng bằng AskUserQuestion:
   - `multiSelect`, tối đa 4 đề xuất mỗi câu;
   - mỗi lựa chọn ghi: lợi ích · file bị sửa;
   - mục "nên làm" đặt trước.
4. Áp dụng các mục được duyệt bằng Edit, đúng diff trong đề xuất. Sau khi sửa phải kiểm:
   - file còn đúng định dạng (bảng, frontmatter);
   - không sinh luật trùng với nơi khác;
   - `FLOW.md` và `CLAUDE.md` không dài thêm, trừ khi đề xuất ghi rõ lý do.
5. Ghi vào `lessons.md`:
   - mục đã áp dụng chuyển sang "Đang thử", kèm ngày, chỉ số theo dõi và mốc trước;
   - mục bị từ chối chuyển sang "Đã bỏ", kèm lý do một dòng.
6. Ghi trạng thái ở đầu file đề xuất: đã áp dụng / bỏ / chờ.

## Đo hiệu quả

Mục "Đang thử" qua ≥ 3 task thì so chỉ số với mốc trước, dùng `metrics.csv`:
- tốt hơn → chuyển sang "Đã chốt";
- không đổi hoặc tệ hơn → đề xuất hoàn tác, người dùng duyệt như bước 3.

## Ghi số đo (FLOW mục 13, L1)

Mỗi khi một agent xong, thêm một dòng vào `learning/metrics.csv`, lấy số từ `<usage>` của task-notification:

```
date,ticket,flow,step,agent,model,tokens,tool_uses,duration_s,round,outcome,note
2026-10-15,EPB-21073,develop,D6,implementer,sonnet,64257,20,131,1,ok,"FE fix"
```

- `round`: 1 cho lần gọi đầu, tăng dần với mỗi lượt gửi tiếp.
- Mỗi lần gọi một dòng riêng, không gộp. Thiếu `<usage>` thì để trống `tokens` và ghi lý do ở `note`, không ước tính. Việc Orchestrator tự chạy nền (vd test) không cần dòng.
- `outcome`: `ok` | `rework` (phải sửa tiếp) | `fail`.
- `note`: ≤ 10 từ; không ghi dữ liệu khách hàng, token, mật khẩu.
- Việc không có ticket: cột `ticket` ghi slug của run log.
