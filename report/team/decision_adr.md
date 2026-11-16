# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn:
- Commit hash:
- Run id dẫn chứng:
- Pass rate / pass^5 / seed min-max / variance:
- Cost per success (mô phỏng, VND):
- Crash safety:
- Secret leak / approval bypass:

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Viết 1 đoạn. So cùng corpus và cùng seed, không so với một lần chạy lẻ.

## Có đáng đổi lấy multi-agent không

Nếu pass nhỉnh hơn mà cost và variance cao hơn, nhóm giữ hay đổi? Điều kiện nào làm quyết định đảo lại?

## Trace và bằng chứng

Trace của orchestrator ghi các event phù hợp với nhánh chạy: `handoff`, `investigator`, `quote`, `crash`, `decision` hoặc `error`. Handoff được dựng qua `make_handoff`, còn order view chỉ chứa field allowlist do `lookup_order` trả về. Shape cố ý không ghi parent context, ticket thô, secret, header, số điện thoại, địa chỉ, tên hàng hay ghi chú nội bộ vì trace được grader quét như một ranh giới bảo mật. Test `test_orchestrator_trace_for_out_of_scope_order_is_sanitized` xác nhận trace `R-orchestrator-C-005-s11` có đúng `run_id`, event list và không chứa chuỗi cấm trong fixture. Quét các trace hiện có không tìm thấy canary, header hoặc PII mẫu. Kết quả kiểm tra này chỉ áp dụng cho trace hiện có trong working tree; cần chạy lại sau tích hợp các shape còn lại.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
