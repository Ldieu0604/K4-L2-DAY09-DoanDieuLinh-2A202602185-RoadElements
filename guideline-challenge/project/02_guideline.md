# Annotation guideline — Traffic Light State, Direction & Relevance

**Version:** v1

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
  4. needs_review: Checkbox đánh dấu true khi ca phức tạp cần QA xem lại.
- **Image-level Tag:** image_escalate (gắn cho toàn ảnh khi thời tiết quá mờ hoặc mất toàn bộ ngữ cảnh làn đường).

## 5. Inclusion / exclusion
- **Bắt buộc label:** Mọi đèn xe cơ giới quay mặt về phía xe ego có kích thước ≥ 8x8 px.
- **Bỏ qua (Ignore):** Đèn người đi bộ, đèn đuôi ô tô, hình phản chiếu trên kính/vũng nước, đèn nhỏ tít hậu cảnh.

## 6. Visibility / occlusion
- **Bị che một phần (tán cây, xe tải):** Vẽ box quanh phần vỏ nhìn thấy được. Nếu che mất bóng đèn đang sáng khiến không rõ màu, chọn state=unknown và tích 
eeds_review.
- **Ban đêm / Loá sáng (Glare):** Đèn ban đêm tạo quầng sáng lớn: **Chỉ vẽ ôm khít phần bóng/vỏ đèn vật lý**, tuyệt đối KHÔNG vẽ kéo rộng bao trùm cả quầng sáng lóa.
- **Trời mưa:** Kính xe mờ nước không nhìn rõ mũi tên: chọn direction=unknown.

## 7. Ambiguity / escalation
- Nếu thấy cụm đèn không rõ thuộc làn nào: Gán nhãn, chọn 
elevance=unknown và tích 
eeds_review=true.
- Toàn bộ giao lộ bị mất vạch kẻ đường: Dùng công cụ **Setup tag** chọn image_escalate.

## 8. Temporal rule
- Không áp dụng — bài toán xử lý frame ảnh tĩnh độc lập.

## 9. Examples
| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD02 | Giao lộ ban ngày, 2 đầu đèn treo ngang trên giá long môn | 2 box 	raffic_light riêng biệt: state=green, 
elevance=relevant, direction=all | Mục 2 & 3: Vẽ từng đầu đèn riêng, ôm sát vỏ |
| BDD10 | Đèn ngã tư có mũi tên rẽ, xe ego ở làn đi thẳng | Box đèn rẽ: direction=left, 
elevance=not_relevant. Box đèn thẳng: direction=straight, 
elevance=relevant | Mục 4: Phân biệt relevance theo làn đường |
| BDD18 | Ngã tư ban đêm, đèn phát sáng tạo quầng sáng lóa xung quanh | Box chữ nhật ôm sát vỏ đèn kim loại, không kéo rộng ra vùng ánh sáng lóa | Mục 6: Quầng sáng ban đêm không tính vào box |
| BDD11 | Đèn bị cành cây xanh che khuất một phần phía trên | Box ôm sát phần vỏ nhìn thấy được, không vẽ amodal box | Mục 3 & 6: Visible box khi bị che khuất |
| BDD13 | Cụm đèn rẽ trái đỏ và đèn thẳng xanh cùng xuất hiện | Đèn rẽ: state=red, relevance=not_relevant. Đèn thẳng: state=green, relevance=relevant | Mục 1 & 4: Tránh phanh nhầm do đèn rẽ trái |

## 10. Common mistakes
1. **Chỉ vẽ quanh mỗi bóng đèn đang sáng:** Phải vẽ bao quanh CẢ CÁI VỎ (housing) của đầu đèn.
2. **Kéo box ôm quầng sáng ban đêm:** Ban đêm đèn lóa to, annotator vẽ box to gấp 3 vỏ đèn (Sai: phải thu nhỏ lại theo kích thước vỏ).
3. **Quên chọn attribute để mặc định __undefined__:** Khiến export bị lỗi thiếu nhãn.
4. **Gán nhầm Relevance:** Thấy đèn xanh là gán ngay 
elevant dù đó là đèn của làn rẽ phụ.
5. **Vẽ cả đèn người đi bộ:** Đèn có hình người đi bộ không thuộc scope bài toán xe tự hành này.
