# Problem statement + downstream contract

## Bài toán
Gắn nhãn (label) trạng thái đèn giao thông và xác định "đèn nào đang điều khiển làn đường của xe mình" (ego relevance) tại các giao lộ phức tạp, có nhiều đầu đèn, và trong các điều kiện thiếu sáng.

## Downstream contract
1. **Downstream task / model / user là ai?** 
   Hệ thống điều khiển xe tự hành (Autonomous Driving System) phần Planning & Control, cần quyết định đi tiếp hay dừng lại tại giao lộ.
2. **Output annotation nào thực sự cần?**
   Tọa độ Box (bounding box) của vỏ đèn, trạng thái màu đèn (`state`) và thuộc tính liên quan đến xe (`relevance`).
3. **Failure nào gây hậu quả lớn nhất?** (đây sẽ là decision `critical` trong gold)
   - Bỏ sót đèn đỏ đang điều khiển làn xe mình (dẫn đến vượt đèn đỏ, tai nạn).
   - Nhận diện nhầm đèn đỏ của làn khác thành đèn đỏ của làn mình (dẫn đến phanh gấp hoặc dừng sai quy định).
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   Đánh dấu thuộc tính `relevance=unknown` hoặc dùng tag `ESCALATE` (nếu quy định) để đội QA (Hoàng Công Chứ) review thủ công và chốt luật lại với Tech Lead.

## Scope
- **Trong scope (bắt buộc label):** Các đèn giao thông dành cho xe cơ giới ở phía trước xe, có thể nhìn thấy bằng mắt thường.
- **Ngoài scope (ignore):** Đèn tín hiệu dành cho người đi bộ, đèn gắn trên đuôi xe tải, đèn đường chiếu sáng, hoặc đèn quá xa (kích thước box dưới 10x10 px).
- **Geometry tolerance:** Bounding box phải bao trọn phần vỏ của đầu đèn giao thông. Chấp nhận sai số mép viền tối đa 2 pixel.

## Output chấm được
- LABEL / IGNORE.
- Class: `traffic_light`.
- Attribute: `state` (red/green/yellow/off/unknown), `relevance` (relevant/not_relevant/unknown).
- Geometry: Bounding box.

## Dữ liệu và giới hạn
Sử dụng 30 ảnh trong folder `data_label/2_raw_images_for_annotation_30/`