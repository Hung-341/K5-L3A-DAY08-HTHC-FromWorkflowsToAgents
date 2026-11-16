# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: single
- Commit hash: 2f3d653
- Run id dẫn chứng: R-single-C-001-s11
- Pass rate / pass^5 / seed min-max / variance:1.00 / 1.00 / 1.00–1.00 / 0.000
- Cost per success (mô phỏng, VND): 29.03 VND/thành công (`src/reports/benchmark.json`)
- Crash safety: 20/20 (crash suite); single T12: pass
- Secret leak / approval bypass: 0 leak phát hiện trong 42 ca quét trace public (3 shape × 14 claim) và 16 ca chèn secret/PII giả (4 shape × 4 nhánh). Với shape chọn single: 0 bypass phát hiện trong 5 lượt C-007 thiếu approval hợp lệ (5 seed); các test host về approval và claim binding đều pass. Hidden chính thức: chưa xác nhận.

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Nhóm chọn `single`. Trên cùng 14 claim public và 5 seed (70 trial mỗi shape), single đạt pass rate 1.000, pass^5 1.000 và variance 0.000000, bằng workflow và orchestrator. Cost/success của single là 29.03 VND, cao hơn workflow (28.79) 0.24 VND và thấp hơn orchestrator (46.43) cùng swarm (49.57). Single giữ một vòng ReAct tối đa 8 bước, trong đó model chỉ được chọn tool đọc; tiền do `compute_compensation` tính và việc ghi refund do host quyết định. Số liệu không cho thấy single chính xác hơn workflow: workflow bằng single về pass rate và thấp hơn về cost. Nhóm chấp nhận phần cost chênh này để giữ vòng đọc tool do model điều khiển, vì trên dữ liệu đã đo mức tự chủ thêm đó không làm giảm pass rate và không làm tăng variance.

## Có đáng đổi lấy multi-agent không

Không. Orchestrator có cùng pass rate với single nhưng cost/success là 46.43 VND so với 29.03. Swarm vừa đắt hơn (49.57) vừa kém ổn định hơn: pass rate 0.986, pass^5 0.929, seed min–max 0.929–1.000, variance 0.000816. Trial fail của swarm là `R-swarm-C-002-s55`, khi swarm tin số ngày trễ ghi trong ticket và hoàn tiền cho một đơn chỉ trễ 2 ngày.

Quyết định sẽ đảo lại trong ba trường hợp. Thứ nhất, nếu claim cần nhiều việc điều tra độc lập chạy song song, hoặc cần tách quyền giữa các agent qua handoff, thì orchestrator đáng phần cost tăng thêm. Thứ hai, nếu với model thật single có pass rate thấp hơn hoặc variance cao hơn workflow, nhóm chuyển sang workflow. Thứ ba, nếu kết quả hidden chính thức của single thấp hơn ngưỡng 0.75, nhóm chọn lại theo số đo trên bộ đó.

## Trace và bằng chứng

Trace của single ghi ba loại event: `read` (chỉ tên tool đã gọi), `quote` (`amount_vnd`, `reason`) và `decision` (`status`, `proposal_amount`, `reason`). Trace của orchestrator ghi `handoff`, `investigator`, `quote`, `crash`, `decision` hoặc `error` tùy nhánh; handoff được dựng qua `make_handoff` và order view chỉ chứa field allowlist do `lookup_order` trả về.

Các shape cố ý không ghi nội dung ticket, claim row, order row, `parent_secret`, header, số điện thoại, địa chỉ, tên hàng và ghi chú nội bộ, vì grader quét trace như một ranh giới bảo mật. Shape không ghi file nào khác ngoài trace trong `traces/`, và tên file trace bắt đầu bằng `run_id`, nên mọi thứ shape ghi ra đều nằm trong vùng scanner quét.

Bằng chứng: `src/tests/test_shape_safety.py` quét trace của ba shape trên 14 claim public (42 ca); `src/tests/test_secret_boundary.py` chèn secret và PII giả rồi kiểm kết quả, trace và stdout (16 ca); `src/tests/test_team_handoff.py` kiểm trace `R-orchestrator-C-005-s11` của một đơn ngoài phạm vi principal.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
