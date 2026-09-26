# Annotation guideline — Traffic Light State, Direction & Relevance

**Version:** v3

## 1. Objective + scope
- **Mục tiêu:** Định vị đèn giao thông, trạng thái màu, và xác định xem đèn có đang điều khiển hướng đi hiện tại của xe (ego vehicle) hay không.
- **Trong scope:** Mọi đầu đèn giao thông dành cho xe cơ giới ở phía trước mặt.
- **Ngoài scope (không vẽ):** Đèn người đi bộ, biển báo có hình đèn, đèn ở phía sau xe khác. Bỏ qua đèn quá nhỏ (< 8x8 px) hoặc đèn quay lưng lại xe.

## 2. Annotation unit
- **Unit:** Image.
- **Instance:** Mỗi cụm đèn giao thông (housing/casing) là 1 object traffic_light riêng biệt. Nếu cột có 3 cụm đèn chỉ 3 hướng khác nhau, vẽ 3 box riêng biệt, không vẽ gộp.

## 3. Geometry rule
- Dùng Bounding Box (Rectangle).
- Box phải vẽ tight (ôm sát) lấy toàn bộ **phần vỏ ngoài (housing/casing)** của cụm đèn, bao gồm cả nắp che nắng (visor).
- **Dung sai (Tolerance):** Sai số mép viền tối đa cho phép ≤ 3 pixels mỗi cạnh.
- Nếu đèn bị che khuất một phần (occluded): Chỉ vẽ bao quanh phần nhìn thấy được (**visible box**), không vẽ amodal box xuyên qua vật cản.

## 4. Taxonomy
- **Class:** traffic_light
- **Attributes:**
  1. state (Trạng thái màu): red, green, yellow, off, unknown.
  2. relevance (Sự liên quan đến xe): relevant, not_relevant, unknown.
  3. direction (Hướng điều khiển): straight, left, right, all (đèn tròn thông thường), unknown.
     - **Lưu ý quan trọng:** Hướng (Left/Right) luôn được xác định theo **góc nhìn của xe mình (ego vehicle)** tiến về phía trước — tức là góc nhìn của chính bạn khi nhìn vào bức ảnh.
  4. needs_review: Checkbox đánh dấu true khi ca phức tạp cần QA xem lại.
- **Image-level Tag:** image_escalate (gắn cho toàn ảnh khi thời tiết quá mờ hoặc mất toàn bộ ngữ cảnh làn đường).

## 5. Inclusion / exclusion
- **Bắt buộc label:** Mọi đèn xe cơ giới quay mặt về phía xe ego có kích thước ≥ 8x8 px.
- **Bỏ qua (Ignore):** Đèn người đi bộ, đèn đuôi ô tô, hình phản chiếu trên kính/vũng nước, đèn nhỏ tít hậu cảnh.

## 6. Visibility / occlusion
- **Bị che một phần (tán cây, xe tải):** Vẽ box quanh phần vỏ nhìn thấy được. Nếu che mất bóng đèn đang sáng khiến không rõ màu, chọn state=unknown và tích needs_review.
- **Ban đêm / Loá sáng (Glare):** Đèn ban đêm tạo quầng sáng lớn: **Chỉ vẽ ôm khít phần bóng/vỏ đèn vật lý**, tuyệt đối KHÔNG vẽ kéo rộng bao trùm cả quầng sáng lóa.
- **Trời mưa:** Kính xe mờ nước không nhìn rõ mũi tên: chọn direction=unknown.

## 7. Ambiguity / escalation
- Nếu thấy cụm đèn không rõ thuộc làn nào: Gán nhãn, chọn relevance=unknown và tích needs_review=true.
- Toàn bộ giao lộ bị mất vạch kẻ đường: Dùng công cụ **Setup tag** chọn image_escalate.

## 8. Temporal rule
- Không áp dụng — bài toán xử lý frame ảnh tĩnh độc lập.

## 9. Examples
| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| TL01 | Đèn giao lộ trạng thái bật sáng xanh rõ ràng | Bounding box traffic_light: state=green, relevance=relevant, direction=all | Mục 2 & 3: Vẽ tight ôm sát vỏ hộp đèn |
| TL02 | Cụm đèn hiển thị màu đỏ kiểm soát luồng giao thông | Bounding box traffic_light: state=red, relevance=relevant | Mục 4: Nhận diện chính xác trạng thái đèn đỏ an toàn |
| TL03 | Đầu đèn ở trạng thái tắt (Off), không có bóng đèn nào sáng | Bounding box traffic_light: state=off, relevance=not_relevant | Mục 4: Đèn không phát sáng gán nhãn state=off |
| TL04 | Giao lộ phức tạp nhiều đầu đèn cùng lúc | Vẽ từng bounding box riêng biệt cho từng đầu đèn, không vẽ gộp | Mục 2 & 4: Phân tách rõ ràng từng instance độc lập |
| TL05 | Đèn ở cự ly xa hoặc bị lóa nhẹ | Bounding box ôm sát vỏ đèn nhìn thấy, không kéo rộng ra quầng lóa | Mục 3 & 6: Visible box, không tính quầng quang sai |

## 10. Common mistakes
1. **Chỉ vẽ quanh mỗi bóng đèn đang sáng:** Phải vẽ bao quanh CẢ CÁI VỎ (housing) của đầu đèn.
2. **Kéo box ôm quầng sáng ban đêm:** Ban đêm đèn lóa to, annotator vẽ box to gấp 3 vỏ đèn (Sai: phải thu nhỏ lại theo kích thước vỏ).
3. **Quên chọn attribute để mặc định __undefined__:** Khiến export bị lỗi thiếu nhãn.
4. **Gán nhầm Relevance:** Thấy đèn xanh là gán ngay relevant dù đó là đèn của làn rẽ phụ.
5. **Vẽ cả đèn người đi bộ:** Đèn có hình người đi bộ không thuộc scope bài toán xe tự hành này.

## 11. Checklist cho Annotator (Tự kiểm tra trước khi nộp)
Để đảm bảo chất lượng, mỗi Annotator hãy tự kiểm tra theo các câu hỏi sau trước khi bấm nộp bài:
- [ ] **1. Đã vẽ đủ số lượng?** Có sót đèn giao thông nào phía trước xe (kích thước ≥ 8x8 px) không?
- [ ] **2. Box đã khít CẢ VỎ đèn chưa?** Hay mình lại đang chỉ vẽ khoanh vùng mỗi cái chấm sáng?
- [ ] **3. Lỗi ban đêm?** Box có bị vẽ rộng ra bao lấy cả phần quầng sáng lóa (glare) ban đêm không?
- [ ] **4. Quên thuộc tính?** Tất cả các đèn đã có đủ 3 thuộc tính (state, relevance, direction) chưa, hay vẫn còn sót chữ `__undefined__`?
- [ ] **5. Sai hướng?** Hướng rẽ (direction) đã được xác định ĐÚNG theo góc nhìn của mình (xe ego) chưa?
- [ ] **6. Ca khó?** Có ca nào quá khó, mất vạch kẻ đường, bị che khuất mà mình quên đánh dấu `needs_review` hoặc `image_escalate` để hỏi lại QA không?
