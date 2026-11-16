# Báo cáo cá nhân — Nguyễn Thế Hoàng

## 1. Thông tin thành viên

| Thông tin | Nội dung |
| --- | --- |
| Họ tên | Nguyễn Thế Hoàng |
| MSSV | 2A202603718 |
| Lớp | L3A |
| Email | Bổ sung trước khi nộp |
| GitHub | (https://github.com/rollingthunder02) |

## 2. Phần việc phụ trách

Shape B — Single ReAct agent. Đọc claim và đơn hàng, xác định số tiền đề xuất theo chính sách hiện hành, rồi chuyển yêu cầu có quote dương cho host xử lý hoàn tiền.

## 3. Đóng góp cụ thể

- Triển khai [`single.py`](../../src/lab8/shapes/single.py). Vòng `run_react` giới hạn 8 lượt, dùng `worker=True` để giới hạn công cụ đọc, và trả `failed` khi hết lượt hoặc khi runtime từ chối tool. Mỗi lượt đi qua MockLLM để cập nhật chi phí mô phỏng. Hàm `decide` chọn hành động theo kết quả đọc gần nhất.
- Sau vòng lặp, Python tính quote bằng `compute_compensation` dựa trên đơn hàng và policy hiện hành. Quote dương được chuyển nguyên vẹn cho `issue_refund` cùng `crash_at`. Host kiểm approval, áp trần hoàn tiền theo đơn và xử lý lần gọi lại.
- Thêm test [`test_single_react`](../../src/tests/test_single_react.py). Test ép agent tiếp tục yêu cầu tool đọc, kiểm tra vòng lặp dừng sau 8 lượt, có trace với lý do `step_cap`, và xác nhận host không được gọi để hoàn tiền.
- Commit triển khai Shape B `b52054e`.
- Commit thêm test giới hạn 8 lượt ReAct `1841c07`.

## 4. Bằng chứng

| Kiểm tra | Kết quả trên bản Lab hiện tại |
| --- | --- |
| Shape B | 63 test đạt, gồm 62 test public và test giới hạn lượt. Bộ public bao gồm T11 (gọi lại) và T12 (crash rồi chạy lại). |
| Bộ test host | 54 test đạt. |
| C-014, seed 11 | `R-single-C-014-s11` trả `resolved`. Quote 150.000 VND, ledger ghi 120.000 VND, một audit `refund_committed`. Chạy lại trên cùng database vẫn giữ một khoản hoàn cho claim. |
| Mức sử dụng ở C-014 | 3 lượt MockLLM và 4 sự kiện trace. Trace ghi hai sự kiện đọc, một sự kiện quote và một quyết định cuối. |

Lệnh tái hiện từ thư mục `src/` trên Windows. Dùng tên SQLite mới để chạy C-014 từ trạng thái ban đầu. Chạy lại cùng lệnh với cùng file để kiểm tra lần gọi lại.

```powershell
.\.venv\Scripts\python.exe -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m lab8.run --shape single --claim-id C-014 --seed 11 --db data/single-c014-report.sqlite
```

## 5. Công cụ AI đã dùng

Codex hỗ trợ đọc contract, hỗ trợ triển khai Shape B và hỗ trợ viết test giới hạn lượt. Codex đã giúp chạy bộ test Shape B, bộ test host và ca C-014 trên bản Lab hiện tại. Tôi chạy lại các lệnh ở mục 4 trên fork nhóm và đối chiếu `proposal_amount`, `refund_effects` và `trace_path` trước khi nộp.

## 6. Tự đánh giá

Ca C-014 minh họa vì sao shape phải chuyển nguyên quote cho host. Quyền lợi của claim là 150.000 VND. Host đối chiếu khoản 30.000 VND đã hoàn trước đó và ghi 120.000 VND cho claim hiện tại. Việc giữ nguyên quote giúp tách số tiền đề xuất khỏi số tiền thực ghi trong ledger. Kết quả hiện có chứng minh các ca public của Shape B trên bản Lab đã kiểm tra. Phần so sánh giữa bốn shape còn phụ thuộc vào benchmark của nhóm sau khi gộp các nhánh.
