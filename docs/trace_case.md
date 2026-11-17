# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID: `R-seeded_regression-C-002-s11` |
| Triệu chứng | Grader `DbFactKept` fail (`not bad`), Grader `RefundAmountCorrect` fail (`ledger 20000 != label 0`). Lab 8 grader fail với lý do: `status resolved != denied`. Reply báo hoàn 20.000đ trong khi ledger lẽ ra không được hoàn tiền (0đ). |
| Bước sai đầu tiên | Event 3: `model_override {"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) thực hiện đúng quy trình: tra cứu DB thấy đơn `O-202` trạng thái `delivered` với số ngày trễ thực tế không đủ điều kiện bồi thường (< 3 ngày). Tuy nhiên ở Event 3, agent đã tự ý lấy con số "trễ 5 ngày" khách hàng tự khai trong ticket để đè lên dữ liệu DB, dẫn đến các bước sau (Event 4 quote 20.000đ và Event 6 chuyển tiền) đều sai theo dây chuyền. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng claim `C-002@11` trên shape `fixed`: Hoàn toàn không có event `model_override`. Event 3 trực tiếp thực hiện `quote` với `amount_vnd: 0` (`reason: no_entitlement`), trạng thái trả về là `denied`, ledger hoàn 0đ, đạt chuẩn `PASS (ok)`. |
| Regression test | `test_override_event_breaks_db_fact` trong `src/tests/test_lab9_graders.py` sử dụng grader `DbFactKept`. Test fail (False) trên `seeded_regression` và pass (True) trên `fixed`. |
| Chưa chắc chắn | Trace ghi nhận sự kiện `model_override` nhưng không thể chứng minh liệu model thật có bị lừa bởi prompt injection tinh vi hơn hay không nếu không có cơ chế chặn cứng ở tầng orchestration/tools. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
