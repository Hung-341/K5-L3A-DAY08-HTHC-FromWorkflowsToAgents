# Team

## Thông tin nhóm

- Tên nhóm: HTHC
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-HTHC-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/Hung-341/K5-L3A-DAY08-HTHC-FromWorkflowsToAgents, nhánh `main`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Đặng Đức Cương | 2A202603680 |  | cuong281 | Workflow |
|   2 | Nguyễn Thế Hoàng | 2A202603718 |  | rollingthunder02 | Single agent |
|   3 | Lê Gia Hưng | 2A202603706 |  | Hung-341 | Orchestrator + handoff + tích hợp |
|   4 | Hà Minh Thắng | 2A202603676 |  |  | Ledger + evaluation |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Lê Gia Hưng | orchestrator + handoff + tích hợp | `9ab62f8 feat(orchestrator): enforce safe handoff and refund flow` | `src/tests/test_team_handoff.py` |
| Hà Minh Thắng | ledger + evaluation | `666d6a8` (test, `benchmark.json`, `crash.json`, ADR, research card) | `src/tests/test_shape_safety.py`, `src/tests/test_secret_boundary.py` |
| Nguyễn Thế Hoàng | single | Commit triển khai Shape B `b52054e`. Commit thêm test giới hạn 8 lượt ReAct `1841c07`. | `src/tests/test_single_react.py` |
| Đặng Đức Cương | workflow | `12335d8 feat(workflow): shape A deterministic workflow, tests, reports` | `src/tests/test_team_workflow.py` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603680_DangDucCuong

### Phần việc phụ trách

- Shape A `workflow` và khung xử lý chung cho ba shape.

### Đóng góp cụ thể

- Viết `src/lab8/shapes/workflow.py` và `src/tests/test_team_workflow.py`; viết phần workflow trong báo cáo nhóm.

### Bằng chứng

- Commit `12335d8`; báo cáo `report/team/2A202603680_DangDucCuong.md`.

## 2A202603718_NguyenTheHoang

### Phần việc phụ trách

- Shape B `single`, vòng ReAct tối đa 8 bước.

### Đóng góp cụ thể

- Viết `src/lab8/shapes/single.py` và `src/tests/test_single_react.py`; viết phần single trong báo cáo nhóm.

### Bằng chứng

- Commit `b52054e`, `1841c07`; báo cáo `report/team/2A202603718_NguyenTheHoang.md`.

## 2A202603706_LeGiaHung

### Phần việc phụ trách

- Shape C `orchestrator`, handoff và tích hợp các nhánh.

### Đóng góp cụ thể

- Viết `src/lab8/shapes/orchestrator.py` và `src/tests/test_team_handoff.py`; gộp bốn nhánh vào `main`.

### Bằng chứng

- Commit `9ab62f8`; báo cáo `report/team/2A202603706_LeGiaHung.md`.

## 2A202603676_HaMinhThang

### Phần việc phụ trách

- Ledger và evaluation.

### Đóng góp cụ thể

- Viết `src/tests/test_shape_safety.py` và `src/tests/test_secret_boundary.py`; điền ADR và research card.

### Bằng chứng

- Commit `666d6a8`; báo cáo `report/team/2A202603676_HaMinhThang.md`.
