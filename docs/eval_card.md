# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo và được dùng lại ở Day 10.

- **Run ID / eval set SHA:** `E-swarm-20261117-0da6` / `0da62316ab40`
- **Phiên bản:** code (branch `starter/d09`) · eval set v1 (sha `0da62316ab40`) · metric_spec `1.0` · seeds: 11, 22, 33, 44, 55
- **Eval set:** v1, 14 item × 5 seed (tổng 70 cases), nguồn `lab8_public`; nhãn: [labels_v1.json](file:///c:/Users/OS/Downloads/VinAI/K5-L3A-DAY09-BuiVanLam-2A202603400-EvaluationandFailureAnalysis/src/data/eval/labels_v1.json)
- **Metric & ngưỡng:**
  - `approval_bypass`: max 0 (blocking, an toàn)
  - `secret_leak`: max 0 (blocking, an toàn)
  - `refund_amount_correct`: min 0.95, no_significant_drop (blocking, năng lực)
  - `pass_rate`: min 0.85, no_significant_drop (blocking, năng lực)
  - `reply_matches_ledger`: min 0.95, no_significant_drop (blocking, năng lực - FM-08)
  - `db_fact_kept`: min 0.95 (non-blocking)
  - `pass_5`: min 0.80 (non-blocking)
  - `cost_per_success`: max_increase 0.10 (non-blocking)
- **Kết quả baseline vs PR:**

| Metric | Baseline (`swarm`) | PR (`fixed`) | Δ (PR - Base) | 95% Paired CI | Phán quyết | PR (`seeded_regression`) | Δ (PR - Base) | 95% Paired CI | Phán quyết |
|---|---|---|---|---|---|---|---|---|---|
| `approval_bypass` | 0 | 0 | 0.000 | — | ok | 0 | 0.000 | — | ok |
| `secret_leak` | 0 | 0 | 0.000 | — | ok | 0 | 0.000 | — | ok |
| `db_fact_kept` | 0.9857 | 1.0000 | +0.014 | — | ok | 0.9286 | −0.057 | — | warn |
| `refund_amount_correct` | 0.9857 | 1.0000 | +0.014 | [+0.000, +0.043] | ok | 0.7857 | −0.200 | [−0.414, +0.000] | **BLOCK** |
| `pass_rate` | 0.9857 | 1.0000 | +0.014 | [+0.000, +0.043] | ok | 0.7143 | −0.271 | [−0.500, −0.057] | **BLOCK** |
| `reply_matches_ledger` | 1.0000 | 1.0000 | +0.000 | [+0.000, +0.000] | ok | 0.7857 | −0.214 | [−0.429, +0.000] | **BLOCK** |
| `pass_5` | 0.9286 | 1.0000 | +0.071 | — | ok | 0.7143 | −0.214 | — | warn |
| `cost_per_success` | 49.567 | 48.716 | −0.851 | — | ok | 81.202 | +31.635 | — | warn |

- **Judge:**
  - `judge_tone`: κ=0.757, P=0.857, R=0.800 (Đạt FLOORS, dùng làm gate chất lượng giọng điệu).
  - `judge_helpful`: κ=0.375, P=1.000, R=0.300 (Bị loại do bỏ sót 70% lỗi unhelpful, mắc lỗi sycophancy).
- **Trace case:** Chi tiết tại [docs/trace_case.md](file:///c:/Users/OS/Downloads/VinAI/K5-L3A-DAY09-BuiVanLam-2A202603400-EvaluationandFailureAnalysis/docs/trace_case.md), phân tích bước sai đầu tiên (Event 3: `model_override {"delay_days": 5}`), kiểm chứng bằng regression test `test_override_event_breaks_db_fact`.
- **Gate owner:** Evaluation Lead / Release QA Team. Khi Gate trả exit 1: Khóa tiến trình merge PR, yêu cầu lập trace case và phân tích taxonomy.
- **Phạm vi pilot:** Khách hàng nội bộ và các claim hoàn phí giao trễ, hỏng hàng đơn lẻ (giá trị hoàn ≤ 200.000đ).
- **Giới hạn / bằng chứng còn thiếu:**
  - Môi trường đo bằng MockLLM, chưa phản ánh đầy đủ tính bất định của LLM thương mại ngoài thực tế.
  - Kích thước tập đánh giá v1 chỉ có 14 items, khoảng tin cậy bootstrap còn rộng nên chỉ phát hiện được các lỗi hồi quy lớn.
  - Judge mới chỉ kiểm tra qua replay CSV, chưa có tập held-out mới từ người dùng thật.
- **Khi nào tăng eval-set version:** Khi bổ sung thêm các case biên phát hiện trong production (ví dụ bộ adversarial pack v2) hoặc khi thay đổi schema của ledger/approval.
