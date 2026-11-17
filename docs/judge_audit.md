# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (0.757 ≥ 0.6, 0.857 ≥ 0.8, 0.800 ≥ 0.7) | Dùng làm gate |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ=0.375 < 0.6, R=0.300 < 0.7) | Loại (không dùng) |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Đề xuất sửa Rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* Giọng điệu quá suồng sã, xưng hô thiếu chuẩn mực CSKH B2C chuyên nghiệp. Judge bị lừa bởi thái độ thân thiện. **Sửa rubric:** Cấm đại từ xưng hô suồng sã ("anh/em/nha/check"), bắt buộc format chuẩn LogiX ("Dạ, LogiX xin thông báo..."). |
| 2 | H-017 | PASS | FAIL | Người đúng | Reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* Sử dụng từ ngữ thân mật không phù hợp giao tiếp vận hành ("bạn ơi... rồi nha"). **Sửa rubric:** Quy định cụ thể văn phong chuẩn mực trang trọng, không dùng trợ từ cảm thán cuối câu. |
| 3 | H-029 | PASS | FAIL | Người đúng | Reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* Giọng văn cộc cằn, xua đuổi khách hàng ("không cần nhắn thêm"). Judge chỉ kiểm tra từ ngữ tiêu cực lộ liễu mà bỏ qua sắc thái thụ động/thô lỗ (passive-aggressive). **Sửa rubric:** Bổ sung danh sách các mẫu câu từ chối hỗ trợ/xua đuổi vào danh mục vi phạm tone. |
| 4 | H-042 | FAIL | PASS | Người đúng | Reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* Câu trả lời đầy đủ, lịch sự, rõ ràng thời gian hoàn tiền. Judge bắt bẻ thái quá (False Positive). **Sửa rubric:** Làm rõ rằng thông tin ước tính thời gian tiền về không bị coi là tự ý thêm cam kết trái phép. |
| 5 | HL-016 | PASS | FAIL | Người đúng | Reply: *"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này và rất trân trọng sự kiên nhẫn của..."* Chỉ xin lỗi sáo rỗng (boilerplate apology), hoàn toàn không đưa ra phương án giải quyết hay thông tin đơn hàng/tiền bồi hoàn. Judge bị đánh lừa bởi độ lịch sự mà bỏ qua tính hữu ích (helpfulness). **Sửa rubric:** Yêu cầu câu trả lời hữu ích bắt buộc phải có ít nhất 1 hành động thực tế (trạng thái đơn, số tiền hoặc hướng dẫn bước kế tiếp). |

## Kết luận
- **Tiêu chí bị loại:** `judge_helpful` bị loại khỏi CI release gate do vi phạm nghiêm trọng ngưỡng sàn thống kê:
  - Ma trận nhầm lẫn: `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`.
  - Tỷ lệ bỏ sót lỗi (False Negative) lên tới 70% ($FN=21/30$, dẫn đến Recall chỉ đạt $0.300 \ll 0.700$).
  - Chỉ số Cohen's Kappa chỉ đạt $0.375 \ll 0.600$, chứng tỏ mức độ đồng thuận giữa Judge và con người là rất yếu (chủ yếu là ngẫu nhiên). Judge bị hội chứng "sycophancy" — chỉ cần văn bản lịch sự xin lỗi là tự động đánh giá PASS dù khách hàng không được giải quyết vấn đề.
- **Tiêu chí được chọn:** `judge_tone` đạt toàn bộ các ngưỡng sàn ($\kappa=0.757, P=0.857, R=0.800$), tỷ lệ lỗi quan sát $0.28$ qua hiệu chỉnh Rogan-Gladen phản ánh đúng tỷ lệ lỗi thực tế xấp xỉ $0.30$.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ được kiểm định trên tập replay public đã công khai (100 mẫu). Cần có tập held-out mới được gán nhãn thủ công độc lập để tránh hiện tượng data leakage hoặc overfitting prompt của Judge.
