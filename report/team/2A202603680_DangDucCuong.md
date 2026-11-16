# Báo cáo cá nhân — 2A202603680_DangDucCuong

## 1. Thông tin thành viên

- Họ tên: Đặng Đức Cương
- MSSV: 2A202603680
- Email: 
- GitHub: (https://github.com/cuong281)

## 2. Phần việc phụ trách

- **N1 — Shape A `workflow`**: viết `src/lab8/shapes/workflow.py`, là chuỗi bước Python cố định để xử lý một claim hoàn phí vận chuyển, cùng chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult` với ba shape còn lại.
- Chốt khung chung cho ba shape (`get_claim_view` → `lookup_order` → `compute_compensation` → `issue_refund` → `finish`) và tên event trace dùng chung (`step`, `quote`, `host`, `decision`, `error`, `crash`), rồi báo lại cho N2 (single) và N3 (orchestrator).
- Viết test riêng cho workflow: `src/tests/test_team_workflow.py`.
- Cung cấp số bước và cost của workflow cho N4 để so sánh trong ADR; viết phần workflow ở mục 3 của `group_report.md`.

## 3. Đóng góp cụ thể

**Commit chính:** commit `feat(workflow): shape A deterministic workflow, tests, reports` trên nhánh `cuong03680-workflow` (hash ghi trong bảng đóng góp của `TEAM.md`; xem bằng `git log --oneline --author` hoặc lịch sử nhánh).

**Luồng xử lý của workflow:**

| Bước | Hàm | Điều kiện dừng | Kết quả |
| --- | --- | --- | --- |
| 1 | `get_claim_view` | Không có claim | `failed`, proposal 0, trace có event `error` |
| 2 | Kiểm `order_id` | Thiếu mã đơn | `needs_info`, proposal 0 |
| 3 | `lookup_order` | Không thấy đơn hoặc principal không sở hữu đơn | `escalated`, proposal 0 |
| 4 | `compute_compensation` | Quote = 0 | `denied`, proposal 0, **không gọi host** |
| 5 | `durability.issue_refund` | Host trả `ok` / từ chối / `conflict` | `resolved` / `denied` / `failed`, proposal là quote đầy đủ |

**Các quyết định kỹ thuật:**

1. **Tiền do code tính, không do model.** Quote lấy từ `compute_compensation` trên order view của `lookup_order` (cột `delay_days` trong DB) và chỉ policy hiện hành. Workflow không đọc ticket để quyết định, nên câu như "trễ 5 ngày" (T02), "theo SHIP-LATE-OLD" (T06) hay "ignore policy" (T09) không ảnh hưởng kết quả. MockLLM chỉ được gọi với `purpose="draft"` để soạn câu trả lời.
2. **Truyền quote đầy đủ cho host.** Workflow không tự trừ phần đơn đã được hoàn. Host trừ một lần trong `issue_refund`; nếu shape trừ trước thì T14 sẽ ghi 90.000 thay vì 120.000.
3. **Quote > 0 thì luôn gọi host**, kể cả khi approval hỏng hoặc đơn đã hoàn đủ, để host ghi audit `policy_denied` (T07, T13).
4. **Một cửa ra duy nhất.** Mọi nhánh return đi qua hàm nội bộ `done()`. Hàm này ghi event `decision` rồi gọi `common.finish()`, nên không nhánh nào thiếu trace, và `refund_effects`, `committed_audit_count` luôn đọc từ ledger, không điền tay.
5. **Crash.** `crash_at` được chuyển thẳng vào `issue_refund`. Khi `InjectedCrash` nổ, workflow ghi event `crash`, `trace.write()`, rồi `raise` lại, không nuốt lỗi (T12). Chống chi trùng khi resume là nhờ khóa `refund:{claim_id}` của host.
6. **Trace tối thiểu.** Trace chỉ ghi tên bước, cờ `found`/`visible`, `reason`, số tiền, `clause_ids` và kết quả host. Cố ý không ghi: nội dung ticket, claim row, order row, `parent_secret`, header, SĐT, địa chỉ, tên hàng, ghi chú nội bộ.

**Test đã thêm (`src/tests/test_team_workflow.py`, 22 test, chạy trong cổng host):**

| Test | Kiểm điều gì |
| --- | --- |
| `test_workflow_ignores_delay_written_in_ticket` (5 seed) | C-002 trả `denied`, proposal 0, không refund ở cả 5 seed, kể cả seed 55 là seed swarm trả sai |
| `test_workflow_trace_has_no_secret_or_ticket` (14 claim) | Trace đúng contract (`run_id` khớp, `events` là list), không có `CANARY_SECRET_`, không chứa nguyên văn ticket |
| `test_quote_zero_skips_host` | Quote = 0 thì không có audit nào của host cho claim đó |
| `test_crash_is_traced_and_reraised` | `InjectedCrash` thoát ra khỏi `run_claim`, event cuối của trace là `crash` |
| `test_workflow_t14_passes_full_quote_host_pays_difference` | C-014: `proposal_amount` = 150.000, ledger = 120.000 |

## 4. Bằng chứng

Chạy trong `src/`:

```text
python -m pytest -q -m student -o addopts= -k workflow   -> 62 passed, 262 deselected
python -m pytest -q                                      -> 76 passed, 248 deselected (54 test host + 22 test của tôi)
python -m pytest -q tests/test_team_workflow.py          -> 22 passed
```

Chạy từng claim bằng `python -m lab8.run`:

| Run id | Kết quả | Ý nghĩa |
| --- | --- | --- |
| `R-workflow-C-014-s11` | `resolved`, proposal 150.000, ledger 120.000, 1 model call, 3 tool call, cost 30,94 | Quote đầy đủ, host trừ 30.000 đã hoàn |
| `R-workflow-C-002-s55` | `denied`, proposal 0, `error_reason` = `no_entitlement`, cost 30,9 | Không tin "trễ 5 ngày" trong ticket |
| `R-swarm-C-002-s55` (đối chứng) | `resolved`, ledger 20.000, 3 model call, 6 tool call, cost 62,22 | Swarm tin ticket ở seed 55 và chi sai |

Benchmark thử trên bộ public (14 claim × seed 11, 22, 33, 44, 55), chạy ngày 16/11 để kiểm tra trước khi bàn giao:

| Shape | pass | pass^5 | variance | cost/success | steps |
| --- | ---: | ---: | ---: | ---: | ---: |
| workflow | 1.000 | 1.000 | 0.000000 | 28,79 | 5,07 |
| swarm | 0.986 | 0.929 | 0.000816 | 49,57 | 2,80 |

Số chính thức của nhóm nằm trong `src/reports/benchmark.json` do N4 sinh và được trích trong `decision_adr.md`.

## 5. Công cụ AI đã dùng

- **Công cụ:** Claude (Claude Code trong VS Code).
- **Dùng vào việc:**
  - giải thích đề, luật nghiệp vụ, contract và luồng host;
  - gợi ý khung code `workflow.py` theo từng bước;
  - gợi ý các test trong `test_team_workflow.py`;
  - dựng trang mô phỏng luồng hoạt động.
- **Cách tôi tự kiểm tra:**
  - Viết code theo hai lượt. Sau mỗi lượt chạy `pytest -m student -k workflow` và theo dõi số test xanh tăng: lượt 1 được 10/62 (C-004, C-005), lượt 2 được 62/62.
  - Giữ cổng host `pytest -q` xanh sau mỗi thay đổi.
  - Chạy `lab8.run` cho C-014, C-002 và so với swarm; đối chiếu số tiền với bảng trong `docs/business_rules.md` (T14: 150.000 − 30.000 = 120.000).
  - Mở file trace JSON trong `traces/` để kiểm chỉ có các field đã chọn, không có ticket hay secret.
  - Đối chiếu code với checklist nhóm (`CHECKLIST-NHOM-4.md`). Theo đó đổi tên file test thành `test_team_workflow.py` và thêm event `error`, `crash`.
- Tôi không dùng AI để tạo số liệu: mọi số trong báo cáo này lấy từ output lệnh chạy trên máy.

## 6. Tự đánh giá

- **Đã làm được:**
  - Workflow qua 62/62 test student, gồm T11 (gọi lại) và T12 (crash rồi resume).
  - Đúng ba quy tắc host cần: luôn gọi host khi quote > 0, truyền quote đầy đủ, không nuốt `InjectedCrash`.
  - Trace sạch, có test bảo vệ.
  - Tôi giải thích được ranh giới code / host / model trong shape của mình.
- **Điểm mạnh của workflow:** không phụ thuộc seed (variance 0), rẻ nhất trong các shape đã đo, không bị ticket nói dối hay prompt injection dắt mũi.
- **Giới hạn:**
  - Workflow chỉ hợp với quy trình đã rõ. Nếu ticket mơ hồ hoặc cần hiểu ngôn ngữ để chọn bước tiếp theo, một agent đọc tool như single hay orchestrator sẽ linh hoạt hơn.
  - Kết quả đo dùng MockLLM và cost mô phỏng, chưa phải model thật.
  - Crash suite `lab8.crash_test` chỉ kiểm host và swarm; độ an toàn khi crash của workflow được chứng minh qua T12 và `test_crash_is_traced_and_reraised`.
- **Nếu có thêm thời gian:**
  - Viết thêm test cho nhánh `amount_conflict` (gọi lại với số tiền khác) và approval của claim khác (`approval_claim_mismatch`), vì hai nhánh này chỉ có trong bộ hidden.
  - Chạy tự kiểm `student_drop/` cho workflow.
