# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Chưa được cung cấp; thành viên và vai trò theo `TEAM.md`.
- Thành viên: Đặng Đức Cương, Nguyễn Thế Hoàng, Lê Gia Hưng, Hà Minh Thắng.
- Commit chốt (trùng `report/team/decision_adr.md`): Chưa có; cần cập nhật sau khi tích hợp và commit trên fork nhóm.

## 2. Mục tiêu và cách tiếp cận

Nhóm xử lý cùng một bài toán claim hoàn phí theo bốn kiến trúc để so sánh trên cùng dữ liệu, seed và bảng giá. Code quyết định eligibility, quote, approval và ghi tiền; model chỉ hỗ trợ diễn giải hoặc điều phối trong phạm vi được cấp. Vai trò được chia theo `TEAM.md`: N1 workflow, N2 single agent, N3 orchestrator/handoff/tích hợp, N4 ledger và đánh giá. Tại thời điểm ghi báo cáo, phần orchestrator đã hoàn thiện và kiểm tra; workflow và single vẫn cần bàn giao/tích hợp trước khi có thể kết luận toàn nhóm.

...

## 3. Giải pháp

### Orchestrator (N3)

Shape C đọc claim qua `get_claim_view`, rồi xử lý sớm claim không tồn tại hoặc thiếu `order_id`. Với claim đủ dữ kiện, orchestrator tạo `handoff.v1` bằng `make_handoff`; investigator chỉ được cấp `lookup_order`, `compute_compensation` và `lookup_refund_history`. Investigator không có quyền ghi tiền. Nếu đơn không hiển thị trong phạm vi principal, shape trả `escalated` với proposal bằng 0. Nếu có quote dương, orchestrator chuyển nguyên quote cùng `crash_at` tới `issue_refund`; host chịu trách nhiệm approval, trần theo đơn, idempotency và ghi ledger.

Trace dùng các event `handoff`, `investigator`, `quote`, `crash`, `decision` hoặc `error` tùy nhánh. Handoff được tạo qua allowlist; order view chỉ gồm field được `lookup_order` công bố. Trace không ghi parent context, ticket thô, secret, header, số điện thoại, địa chỉ, tên hàng hoặc ghi chú nội bộ. Test trace ngoài principal scope dùng run id `R-orchestrator-C-005-s11`; kết quả có event `handoff` và `investigator`, không lộ các chuỗi cấm của fixture.

Phần mô tả workflow và single cần được N1/N2 cập nhật sau khi bàn giao code.

...

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

...

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

N3 đã hoàn thiện shape orchestrator, thêm `src/tests/test_team_handoff.py` và xác nhận test shape (62 passed), test handoff (3 passed) cùng cổng host (57 passed). N1, N2 và N4 được phân công theo `TEAM.md`; nội dung đóng góp và commit của họ cần được cập nhật bởi từng người sau khi bàn giao. Bảng đóng góp cuối cùng cần ghi commit thật, không dùng hash dự kiến.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
