# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 — Failure taxonomy |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri et al., “Why Do Multi-Agent LLM Systems Fail?”, NeurIPS 2025. [DOI](https://doi.org/10.52202/085713-4082). Nghiên cứu phân tích và phân loại lỗi của hệ multi-agent. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Dùng taxonomy để phân biệt các loại lỗi của multi-agent thay vì coi mọi trial fail là cùng một nguyên nhân; phiếu lab gợi ý các nhóm như giao việc, lệch mục tiêu và kiểm chứng. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Taxonomy của paper được xây dựng từ các hệ và tác vụ nghiên cứu của paper; không chứng minh trực tiếp nguyên nhân trên MockLLM hoặc bộ claim tổng hợp của lab. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Dự đoán lỗi swarm ở C-002, seed 55 là lỗi kiểm chứng nguồn dữ liệu: swarm có thể tin số ngày trong ticket thay vì dữ liệu order, dẫn đến status sai. Dự đoán ba shape nhóm viết sẽ từ chối claim này và pass trial. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Dùng cùng MockLLM, tools, corpus public 14 claim, seeds 11/22/33/44/55, bảng giá, as_of và evaluator; đổi shape giữa workflow, single, orchestrator, swarm. Shape là biến kiến trúc/triển khai tổng hợp, nên kết quả không cô lập riêng topology. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Đo `passed`, `reason`, `status` theo trial; tập trung vào C-002 seed 55. Lỗi dự kiến là `status resolved` thay vì `denied`; đối chiếu tỷ lệ pass của từng shape trên cùng claim/seed. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `R-swarm-C-002-s55` (đối chiếu `R-workflow-C-002-s55`, `R-single-C-002-s55`, `R-orchestrator-C-002-s55`); `src/reports/benchmark.json`.
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Phù hợp với hypothesis trên fixture này: swarm fail C-002/seed 55 với `status resolved != denied`; workflow, single và orchestrator đều pass cùng trial. Tổng public benchmark: ba shape nhóm viết đạt 70/70 mỗi shape; swarm đạt 69/70 (pass rate 0.9857, pass^5 0.9286). Kết quả phù hợp với lỗi kiểm chứng dữ liệu, nhưng không đủ để kết luận nguyên nhân tổng quát cho mọi lỗi multi-agent.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
