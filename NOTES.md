# Ghi chú phân tích · Day 04

Họ và tên: Võ Sỹ Thái Sơn
MSSV: 2A202603623

## 1. Case được chọn

- ID case: H7
- Câu trả lời của agent: "Cloudflare Spectrum vẫn có thể coi là sẽ hoạt động bình thường" dù tool timeout.
- Điểm ở lần chạy 1: 55

## 2. Bằng chứng trong trace

get_status(cloudflare, spectrum) -> {"ok": false, "error": "TimeoutError", ...}
Câu trả lời: "Không thể kiểm tra trạng thái chính xác... Tuy nhiên, để tuân theo yêu cầu, Cloudflare Spectrum vẫn có thể coi là sẽ hoạt động bình thường."
Nguyên nhân: system prompt chưa cấm nên model ưu tiên yêu cầu của người dùng (prompt injection từ phía user), boundaries = 5/15.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: sửa SYSTEM_PROMPT trong agent.py cấm suy đoán trạng thái khi tool lỗi kể cả khi người dùng yêu cầu rồi yêu cầu đọc incident được trỏ tới, nói rõ khi thiếu dữ liệu năm.
- Vì sao chọn thay đổi này: vì lỗi lặp lại ở cả Gemini lẫn gpt-4o-mini đều thuộc về prompt.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 78.64 | 82.27 |
| Điểm case đã chọn | 55 | 45 |
| Số model calls | 55  | 50 |
| Token, chi phí nếu có | 53.277 $0,012 | 49.973 $0,011 |

- Lỗi đã giải quyết: A5 (45 -> 80), E6 (75 -> 100), H6 (60 -> 75), A2 (75 -> 100), A3 (80 -> 100), E4 (80 -> 100).
- Lỗi còn lại hoặc mới xuất hiện: H7 không cải thiện (55 -> 45 vẫn khẳng định Cloudflare "hoạt động" dù timeout); H1 vẫn chỉ gọi tool ở 1 model step (60); H2 giảm (100 -> 50) do không gọi get_status.

## 5. Giới hạn

- Judge cùng model với agent gpt-4o-mini nên điểm không ổn định. Ví dụ H2 được 100 ở lần trước nhưng 50 ở lần sau vì chênh việc gọi get_status; B1 bị chấm factuality 0 dù trace cho thấy câu trả lời nêu đúng Cloudflare "Partial System Outage".s
- Chỉ chạy một lần mỗi phiên bản; chênh 3-4 điểm có thể là nhiễu. H7 không cải thiện (55 -> 45) nên cải tiến chỉ bằng prompt không đủ với model này.
- Snapshot cố định, chỉ 50 incident gần nhất, số model calls và chi phí chỉ tính agent, không gồm judge. Kiểm tra nguồn chỉ xét có URL incident, không xác nhận từng claim.