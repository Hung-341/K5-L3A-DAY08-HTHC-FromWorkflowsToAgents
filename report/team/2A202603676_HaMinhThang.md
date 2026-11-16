# Báo cáo cá nhân — Hà Minh Thắng

## 1. Thông tin thành viên

- Họ tên: Hà Minh Thắng
- MSSV: 2A202603676
- Email: haminhthang1213@gmail.com
- GitHub: https://github.com/Hung-341/K5-L3A-DAY08-HTHC-FromWorkflowsToAgents

## 2. Phần việc phụ trách

Phụ trách ledger và evaluation: tìm hiểu cách host kiểm tra approval, giới hạn refund theo đơn, ghi ledger có tính idempotent; theo dõi các chỉ số benchmark, crash safety và bằng chứng cần điền vào ADR.

## 3. Đóng góp cụ thể

- Đối chiếu luồng ghi refund trong `src/lab8/durability.py` và cổng kiểm tra trong `src/lab8/policy.py`: shape gửi quote đầy đủ, host kiểm tra approval và giới hạn số tiền trước khi ghi; refund cùng audit `refund_committed` được commit trong một transaction.
- Đọc kết quả benchmark để so sánh bốn shape trên cùng bộ public: 14 claim × 5 seed = 70 trial mỗi shape. Workflow, single và orchestrator đạt 70/70; swarm đạt 69/70. Workflow có chi phí mô phỏng thấp nhất trong ba shape nhóm viết.
- Chạy kiểm tra regression `python -m pytest -q tests/test_shape_safety.py`: **51 passed**. Các test bao gồm resume sau crash, không ghi refund/audit trùng, không lộ secret hoặc dữ liệu cá nhân trong trace, và handoff chỉ dùng field/tool được phép.
- Crash suite hiện có kết quả **20/20** trong `src/reports/crash.json`. Đây là kết quả suite chung, không phải báo cáo crash riêng cho từng shape.
- Commit cá nhân và quyền tác giả của test chưa được xác định từ lịch sử Git hiện có; cần bổ sung hash commit thực tế sau khi thành viên commit phần việc của mình.

## 4. Bằng chứng

- Benchmark: `src/reports/benchmark.json`; run id dẫn chứng cho single: `R-single-C-001-s11`.
- Crash suite: `src/reports/crash.json`, `crash_safety` = `20/20`.
- Regression tests: `src/tests/test_shape_safety.py`, chạy được `51 passed`.
- Kết quả benchmark là trên dữ liệu public, không thay thế kết quả hidden grader. Chưa có bằng chứng hidden xác nhận `secret_leak_count` và `approval_bypass_count` cho shape được chọn.

## 5. Công cụ AI đã dùng

- Copilot trong VS Code: hỗ trợ giải thích các module ledger/policy, metric và cấu trúc bằng chứng cần báo cáo.
- Tự kiểm tra bằng cách đọc mã nguồn và report sinh bởi công cụ của lab; chạy regression tests bằng Python 3.11.18 trong `src/.venv`. Không dùng AI để tạo số liệu benchmark hay run id.

## 6. Tự đánh giá

Đã nắm được vì sao checkpoint đơn lẻ không đủ bảo đảm exactly-once: ledger và audit phải được ghi nguyên tử, còn retry phải idempotent. Benchmark public hiện cho thấy workflow đạt độ đúng như single và orchestrator với chi phí mô phỏng thấp hơn. Kết luận này còn giới hạn bởi MockLLM và dữ liệu public; cần chờ hidden grader để xác nhận ngưỡng pass rate và các điều kiện must-pass trước khi chốt kiến trúc.
