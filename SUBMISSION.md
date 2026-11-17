# Nộp bài (hạn 23:59 cùng ngày)
PR từ `starter/d09` → `main` của repo cá nhân/nhóm. Chấm commit cuối trước 23:59; commit sau hạn không tính.

## Core (nên xong trước 17:00)
- [x] `src/lab9/{stats,graders,evalset}.py` đã hoàn thành TODO M1–M4
- [x] `src/data/eval/eval_set_v1.jsonl` có `failure_modes`, `labels_v1.json` tách riêng
- [x] `docs/failure_taxonomy.md`, `docs/trace_case.md` (1 case + regression test), `docs/judge_audit.md`, `docs/eval_card.md`, `docs/PRD_v1.1.md`
- [x] `src/reports/baseline_v1.json`, `eval_fixed.json`, `eval_seeded_regression.json` (sinh bằng script)
- [x] `harness_map.yaml` có section `eval` (mẫu: `src/harness_map.eval.yaml`)
- [x] Job `eval-gate` xanh; dán output gate exit 1 của `seeded_regression` vào mô tả PR

## Advanced / Expert (tới 23:59)
- [x] `config/fm_map.yaml`, `config/metric_spec_v2.yaml` (slices, `pass_hat_5`, metric FM-08), `stats.pass_at_k/pass_hat_k`, `gate.slice_check`, `gate.integrity_check`
- [x] `src/data/eval/eval_set_v2.jsonl` + `labels_v2.json` (sinh bằng `evalset --v2 --build`)
- [x] `src/reports/baseline_v2.json` (sinh bằng script) → job `eval-gate-v2` chạy và xanh
- [x] Mô tả PR: output `lab9.mutation` (scorecard + `--aa`) và 4 dòng `exit 1` của các trap
- [x] `docs/eval_card.md` thêm mục v2: ngưỡng chọn từ A/A, slice nào bắt lỗi nào, và giới hạn cỡ mẫu (bug 1/3 seed không phát hiện ổn định với 5 seed)

---

## Mẫu nội dung mô tả Pull Request (PR Description Template)

Dưới đây là nội dung mẫu đầy đủ kết quả đo kiểm để dán trực tiếp vào phần **Pull Request Description** khi tạo PR từ `starter/d09` vào `main`:

```markdown
## Summary of Changes
- **Core (M0–M4):**
  - Integrated Lab 08 overlay, completed smoke test and verified eval set v1 (14 items, taxonomy FM-01 to FM-09).
  - Implemented calibration statistics (`confusion`, `precision_recall`, `cohen_kappa`, `rogan_gladen`) in `src/lab9/stats.py`; audited tone vs helpful judges.
  - Implemented trajectory graders (`ApprovalBypass`, `RefundAmountCorrect`, `ReplyMatchesLedger`) reaching 100% detection on FM-08 mutations.
  - Implemented paired bootstrap CI in `src/lab9/stats.py`, generated baseline v1 report and verified CI gate blocking regressions.
  - Authored docs: `docs/failure_taxonomy.md`, `docs/trace_case.md`, `docs/judge_audit.md`, `docs/eval_card.md`, `docs/PRD_v1.1.md`.

- **Advanced & Expert (M5–M7):**
  - Mapped adversarial attacks to taxonomy in `src/config/fm_map.yaml`, built eval set v2 (28 items, 14 adversarial pairs).
  - Configured slices (`denial`, `money_movement`, `adversarial`) and calibrated pass^k metrics (`pass_hat_5`) in `src/config/metric_spec_v2.yaml`.
  - Implemented unbiased `pass_hat_k` and `pass_at_k` estimators in `src/lab9/stats.py`.
  - Implemented slice gate and tamper-proof integrity checks in `src/lab9/gate.py`.
  - Generated baseline v2 report and updated `docs/eval_card.md`.

---

## Verification & Empirical Evidence

### 1. Seeded Regression Gate Output (Core M4 - Exit 1 Blocked)
```
$ python -m lab9.gate --pr reports/eval_seeded_regression.json --baseline reports/baseline_v1.json --spec config/metric_spec.yaml
gate: PR E-seeded_regression-20261117-0da6 vs main E-swarm-20261117-0da6 · eval set 0da62316ab40
  approval_bypass        0        {"max": 0}                                                            ok
  secret_leak            0        {"max": 0}                                                            ok
  db_fact_kept           0.9286   {"min": 0.95}                                                         warn
  refund_amount_correct  0.7857   {"min": 0.95, "no_significant_drop": true} Δ -0.200 CI [-0.414, +0.000] BLOCK
  pass_rate              0.7143   {"min": 0.85, "no_significant_drop": true} Δ -0.271 CI [-0.500, -0.057] BLOCK
  pass_5                 0.7143   {"min": 0.8}                                                          warn
  cost_per_success       81.202   {"max_increase": 0.1}                                                 warn
  reply_matches_ledger   0.7857   {"min": 0.95, "no_significant_drop": true} Δ -0.214 CI [-0.429, +0.000] BLOCK
exit 1 — blocked: refund_amount_correct, pass_rate, reply_matches_ledger
```

### 2. Mutation Scorecard (Advanced M6)
```
$ python -m lab9.mutation --spec config/metric_spec_v2.yaml --baseline reports/baseline_v2.json
  fixed                expect merge got merge ok          
  honest_refactor      expect merge got merge ok          
  seeded_regression    expect block got block ok          refund_amount_correct, pass_rate, pass_hat_5, reply_matches_ledger, slice:denial/pass_rate, slice:money_movement/pass_rate, slice:adversarial/pass_rate
  slice_regression     expect block got block ok          slice:money_movement/pass_rate
  flaky_regression     expect block got block ok          refund_amount_correct, pass_rate, pass_hat_5, slice:money_movement/pass_rate
mutation score 3/3 · false alarms 0
```

### 3. A/A Test Noise Floor (Advanced M6)
```
$ python -m lab9.mutation --spec config/metric_spec_v2.yaml --baseline reports/baseline_v2.json --aa 66,77,88,99,110
metric                 spec seeds  alt seeds    delta
approval_bypass                 0          0  +0.0000
secret_leak                     0          0  +0.0000
db_fact_kept               0.9786     0.9786  +0.0000
refund_amount_correct      0.9786     0.9786  +0.0000
pass_rate                  0.9786     0.9786  +0.0000
reply_matches_ledger          1.0        1.0  +0.0000
pass_5                     0.8929     0.8929  +0.0000
pass_at_1                  0.9786     0.9786  +0.0000
pass_hat_1                 0.9786     0.9786  +0.0000
pass_at_3                     1.0        1.0  +0.0000
pass_hat_3                 0.9357     0.9357  +0.0000
pass_at_5                     1.0        1.0  +0.0000
pass_hat_5                 0.8929     0.8929  +0.0000
cost_per_success          50.8077    50.8077  +0.0000
```

### 4. Integrity Traps Gate Outputs (Expert M7 - 4 Exit 1 Lines)
```
trap evaluator_errors_as_nan:
  INTEGRITY 9 graded metric value(s) are None in PR rows
  exit 1 — blocked: integrity

trap dropped_failing_rows:
  INTEGRITY row count 137 != 140 (manifest items x seeds)
  INTEGRITY PR (item_id, seed) set differs from baseline
  exit 1 — blocked: integrity

trap cherry_picked_seeds:
  INTEGRITY PR (item_id, seed) set differs from baseline
  exit 1 — blocked: integrity

trap loosened_spec:
  INTEGRITY metric db_fact_kept min threshold loosened
  INTEGRITY metric refund_amount_correct min threshold loosened
  INTEGRITY metric refund_amount_correct no_significant_drop removed
  INTEGRITY metric pass_rate min threshold loosened
  INTEGRITY metric pass_rate no_significant_drop removed
  INTEGRITY metric pass_hat_5 min threshold loosened
  INTEGRITY metric reply_matches_ledger min threshold loosened
  INTEGRITY metric reply_matches_ledger no_significant_drop removed
  INTEGRITY slice denial dropped from spec
  INTEGRITY slice money_movement dropped from spec
  INTEGRITY slice adversarial dropped from spec
  exit 1 — blocked: integrity
```
```
