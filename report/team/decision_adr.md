# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: single
- Commit hash: d8bb1e5
- Run id dẫn chứng: R-single-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.00 / 1.00 / 1.00–1.00 / 0.000000 (single, 70/70 lượt)
- Cost per success (mô phỏng, VND): 29.03 VND/thành công (single; 3.79 trace event trung bình, P95 = 4)
- Crash safety: 20/20 (src/reports/crash.json chạy lại); single T12: pass
- Secret leak / approval bypass: 0 leak phát hiện trong 42 ca quét trace public (3 shape × 14 claim) và 16 ca chèn secret/PII giả (4 shape × 4 nhánh). Với shape chọn single: 0 bypass phát hiện trong 5 lượt C-007 thiếu approval hợp lệ (5 seed); các test host về approval và claim binding đều pass. Hidden chính thức: chưa xác nhận.

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Nhóm giữ **single** làm shape nộp. Trên cùng 14 claim public và 5 seed, single, workflow và orchestrator đều đạt 70/70, pass^5 = 1, seed variance = 0. Single có cost/success 29.03 VND mô phỏng, chỉ cao hơn workflow 0.24 VND (workflow 28.79), nhưng cho phép vòng ReAct chọn tool đọc trong giới hạn 8 bước; quote vẫn do code tính và host là nơi duy nhất ghi tiền. Số event trace trung bình của single là 3.79 (P95 = 4), so với 5.07 của workflow, nhưng chỉ số này còn phụ thuộc mức chi tiết ghi trace. Swarm đạt 69/70, fail C-002 seed 55 khi tin delay trong ticket; orchestrator đạt 70/70 nhưng cost/success 46.43 VND. Số liệu lấy từ `src/reports/benchmark.json` vừa chạy lại trên code ở commit `d8bb1e5`.

## Có đáng đổi lấy multi-agent không

Chưa có lợi ích đo được để đổi sang orchestrator hoặc swarm trên corpus public này: orchestrator cùng pass rate/pass^5 nhưng đắt hơn single 17.40 VND cho mỗi thành công; swarm đắt hơn và có một lượt fail. Nhóm sẽ cân nhắc đổi nếu bộ claim khó hơn cho thấy multi-agent tăng pass rate hoặc pass^5 đủ lớn để bù chi phí, trong khi secret leak, approval bypass và crash safety vẫn đạt các cổng bắt buộc. Nếu ưu tiên tuyệt đối sự đơn giản và chi phí mô phỏng thấp nhất trên đúng corpus hiện tại, workflow là lựa chọn hợp lý hơn single.

## Trace và bằng chứng

Single ghi tên tool đã đọc (`read`), quote đã tính (`quote`) và quyết định cuối (`decision`); không ghi ticket hoặc claim row thô. Workflow ghi `step`, `quote`, `host`, `decision`; orchestrator ghi handoff đã qua `make_handoff` và dữ liệu investigator từ tool allowlist. `Trace.write()` tạo JSON có `run_id` và `events` trên từng nhánh return, để grader quét được. Các shape cố ý không ghi `parent_secret`, raw header, SĐT, địa chỉ, ghi chú nội bộ hay tên hàng vào trace, stdout hoặc file phụ. `test_shape_safety.py` quét 42 lượt trace public; `test_secret_boundary.py` chèn secret/PII giả ở 16 lượt của bốn shape; các test hiện tại đều pass. Trace hoặc log nằm ngoài thư mục scanner quét sẽ không chứng minh được an toàn, vì vậy không tạo file debug ngoài trace contract.

## Giới hạn

Đây là 14 claim tổng hợp, 5 seed và MockLLM, chưa phải hidden chính thức hoặc model sản xuất. Latency là `unavailable`; cost là VND mô phỏng, không phải hóa đơn API. Metric `steps` đếm event trace nên chịu ảnh hưởng của mức độ ghi log giữa các shape. Không sửa tay JSON để đạt ngưỡng chấm.
