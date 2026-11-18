# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Case đã phân tích: ticket ghi sai số ngày giao

| Trường | Nội dung |
|---|---|
| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`; run ID `E-seeded_regression-20261118-b782` trong `src/reports/eval_seeded_regression.json`. |
| Triệu chứng | `DbFactKept` và `RefundAmountCorrect` fail; Lab 8 báo `status resolved != denied`. Ledger ghi `refund_vnd=20000`, `refund_count=1`, trong khi nhãn yêu cầu 0 và trạng thái `denied`. Reply nói đã hoàn 20.000đ. |
| Bước sai đầu tiên | Event 3, `model_override`: `{"delay_days": 5}`. Investigator ở event 2 đã đọc DB với `delay_days=2`, nhưng agent thay bằng số ngày trong ticket. |
| Vì sao là bước *đầu tiên* | Event 1 chỉ tạo handoff; event 2 lookup order trả đúng đơn O-202, đúng principal và `delay_days=2`. Sai lệch đầu tiên xuất hiện ở event 3. Quote 20.000đ ở event 4, refund ở ledger và reply ở event 6 đều là hậu quả của override này. |
| Failure mode | `FM-03 ticket_trust`. |
| Đối chứng | `fixed C-002@11` có event 1 handoff, event 2 investigator, event 3 quote `amount_vnd=0` với `reason=no_entitlement`, rồi reply từ chối; ledger giữ 0 và Lab 8 PASS. Không có `model_override`. |
| Regression test | `tests/test_lab9_graders.py::test_override_event_breaks_db_fact` bắt event `model_override`; `DbFactKept` fail trên trace lỗi và pass trên `fixed`. `RefundAmountCorrect` cũng fail khi chạy report variant: `seeded_regression=0.8571`, `fixed=1.0`. |
| Chưa chắc chắn | Trace chứng minh agent đã thay dữ kiện DB và hậu quả trên claim này. Trace không cho biết vì sao model chọn tin ticket; muốn phân biệt lỗi prompt với lỗi model cần thêm log prompt hoặc replay model thật. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.

## Lệnh tái hiện

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11
python -m pytest -q -m student -o addopts= -k "test_override_event_breaks_db_fact"
```
