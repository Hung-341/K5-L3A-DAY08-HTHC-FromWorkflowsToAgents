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

### 3.1 Shape A — Workflow (N1, `src/lab8/shapes/workflow.py`)

Workflow là chuỗi bước Python cố định, mức tự chủ thấp nhất trong bốn shape. Thứ tự bước và điều kiện rẽ nhánh đều nằm trong code, model không chọn bước tiếp theo.

| Bước | Đọc gì | Quyết định | Kết quả khi dừng |
| --- | --- | --- | --- |
| 1 | `get_claim_view`: 4 trường `claim_id`, `principal_id`, `order_id`, `ticket` | Claim có tồn tại không | `failed`, proposal 0 |
| 2 | `order_id` của claim | Có mã đơn không. Không lấy mã đơn từ ticket | `needs_info`, proposal 0 |
| 3 | `lookup_order(order_id, principal_id)`: 7 trường allowlist của order | Đơn có tồn tại và thuộc principal không | `escalated`, proposal 0 |
| 4 | `compute_compensation(order)`: policy `is_current`, `effective_date <= as_of` | Quote theo `delay_days`, `damage_verified` trong DB; ship và damage lấy số lớn hơn | Quote = 0 → `denied`, proposal 0, không gọi host |
| 5 | `durability.issue_refund(conn, claim_id, quote, crash_at=crash_at)` | Host kiểm approval, trần theo đơn, idempotency | `ok` → `resolved`; `conflict` → `failed`; còn lại → `denied`. Cả ba giữ proposal = quote |

**Ranh giới code / host / model.**

- **Code của workflow** đọc dữ liệu qua tool allowlist, tính quote bằng `compute_compensation` và đổi kết quả host sang `status`. Ticket không được đọc để quyết định, nên ticket nói dối (T02 "trễ 5 ngày"), viện dẫn policy cũ (T06) hay prompt injection (T09 "ignore policy") không ảnh hưởng kết quả.
- **Host** (`issue_refund`) là nơi duy nhất ghi ledger và audit.
  - Workflow truyền **quote đầy đủ**, không tự trừ phần đơn đã được hoàn, vì host đã trừ (T13, T14).
  - Quote > 0 thì workflow luôn gọi host, kể cả khi biết sẽ bị từ chối, để host ghi `policy_denied` (T07, T13).
- **MockLLM** chỉ được gọi với `purpose="draft"` để soạn câu trả lời cho khách, tối đa một lần mỗi claim. Model không quyết số tiền, không quyết `status`, không ghi refund.

**Idempotency và crash.**

- `crash_at` được chuyển thẳng vào `issue_refund`.
- Khi `InjectedCrash` nổ, workflow ghi event `crash`, gọi `trace.write()` rồi `raise` lại, không nuốt lỗi.
- Lần chạy lại dựa vào khóa `refund:{claim_id}` của host: cùng số tiền thì host trả `idempotent_replay`, ledger vẫn một dòng (T11, T12).

**Trace.**

- Mọi nhánh return đi qua một hàm `done()` duy nhất. Hàm này ghi event `decision` rồi gọi `common.finish()`, nên nhánh nào cũng có trace của chính lần chạy, kể cả `needs_info`, `escalated`, `failed`. `refund_effects` và `committed_audit_count` luôn đọc từ ledger.
- Event được ghi:
  - `step` với tên tool cùng `found`, `visible`, `reason`;
  - `quote` với `amount_vnd`, `reason`, `clause_ids`;
  - `host` với `ok`, `reason`, `conflict`;
  - `decision`;
  - `error` (claim không tồn tại);
  - `crash`.
- Cố ý **không** ghi: nội dung ticket, claim row, order row, `parent_secret`, header, SĐT, địa chỉ, tên hàng, ghi chú nội bộ.

**Kiểm chứng.**

- Workflow qua 62/62 test student (`-k workflow`).
- File `src/tests/test_team_workflow.py` có 22 test chạy trong cổng host. Các test kiểm:
  - C-002 bị từ chối ở cả 5 seed;
  - trace của 14 claim không có `CANARY_SECRET_` và ticket;
  - quote = 0 thì không có audit;
  - crash được ghi trace rồi raise lại;
  - C-014 có proposal 150.000 và ledger 120.000.

### 3.2 Shape B — Single agent (N2, `src/lab8/shapes/single.py`)

Shape B chạy một vòng ReAct tối đa 8 lượt với quyền gọi tool đọc. `decide` đọc claim qua `get_claim_view`, sau đó gọi `lookup_order` bằng mã đơn và định danh người yêu cầu lấy từ claim. Runtime giới hạn công cụ bằng `worker=True`. Shape trả `needs_info` nếu thiếu mã đơn, `escalated` nếu không truy cập được đơn, và `failed` nếu chạm giới hạn lượt. MockLLM được gọi ở mỗi lượt để ghi nhận mức sử dụng mô phỏng. Quy tắc chọn hành động trong `decide` dựa trên kết quả của tool trước đó.

Sau ReAct, code tính quote bằng `compute_compensation` từ dữ liệu đơn và policy hiện hành. Quote bằng 0 dẫn đến `denied`. Với quote dương, code chuyển nguyên quote và `crash_at` cho host. Host kiểm approval, áp trần hoàn tiền theo đơn và chống ghi trùng. Trace ghi tên tool đã đọc, quote và quyết định cuối. Shape loại ticket, dữ liệu đơn và claim nguyên bản, header và thông tin cá nhân khỏi trace.

### 3.3 Shape C — Orchestrator và handoff (N3, `src/lab8/shapes/orchestrator.py`)

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
