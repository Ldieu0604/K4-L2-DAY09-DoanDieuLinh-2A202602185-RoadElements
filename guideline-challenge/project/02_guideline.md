# Annotation guideline — Traffic Light State & Relevance

**Version:** v1

## 1. Objective + scope
- **Mục tiêu:** Định vị đèn giao thông, trạng thái màu, và xác định xem đèn có đang điều khiển hướng đi hiện tại của xe (ego vehicle) hay không.
- **Trong scope:** Mọi đầu đèn giao thông dành cho xe cơ giới ở phía trước mặt.
- **Ngoài scope (không vẽ):** Đèn người đi bộ, biển báo có hình đèn, đèn ở phía sau xe khác (với đèn cảnh báo). Bỏ qua đèn quá nhỏ (< 10x10 px).

## 2. Annotation unit
- **Unit:** Image.
- **Instance:** Mỗi cụm đèn giao thông (housing/casing) là 1 object `traffic_light` riêng biệt. Nếu cột có 3 cụm đèn chỉ 3 hướng khác nhau, vẽ 3 box riêng biệt.

## 3. Geometry rule
- Dùng Bounding Box (Rectangle).
- Box phải vẽ tight (ôm sát) lấy toàn bộ **phần vỏ ngoài (housing/casing)** của cụm đèn, không chỉ khoanh vùng bóng đèn đang sáng.
- Nếu đèn lọt vào khung ảnh nhưng bị che khuất một phần (occluded), chỉ vẽ phần nhìn thấy được (visible box).

## 4. Taxonomy
- **Class:** `traffic_light`
- **Attributes:**
  1. `state` (Trạng thái màu): 
     - `red`: Đỏ
     - `green`: Xanh
     - `yellow`: Vàng
     - `off`: Không sáng
     - `unknown`: Sáng nhưng lóa không rõ màu.
  2. `relevance` (Sự liên quan đến xe):
     - `relevant`: Đèn điều khiển làn đường mà xe có khả năng đang đi thẳng hoặc rẽ (nếu đang ở làn rẽ).
     - `not_relevant`: Đèn của hướng đi ngược chiều, hoặc đèn dành rõ ràng cho làn rẽ trong khi xe đang ở làn đi thẳng.
     - `unknown`: Không thể xác định xe đang ở làn nào để kết luận.

## 5. Inclusion / exclusion
- **Bắt buộc:** Phải khoanh tất cả đèn giao thông có thể tác động đến việc lái xe.
- **Bỏ qua:** Hình phản chiếu của đèn giao thông trên kính xe hoặc vũng nước.

## 6. Visibility / occlusion
- **Bị che một phần (cành cây, xe tải lớn):** Vẽ box quanh phần vỏ đèn nhìn thấy được. Nếu phần bị che >80% khiến không nhìn được màu đèn, chọn `state=unknown`.
- **Loá (Glare):** Nếu chụp ngược sáng hoặc phơi sáng lâu làm đèn lóa to hơn vỏ thực tế, cố gắng ước lượng vỏ đèn thật, chọn `state=unknown` nếu không thể chắc chắn màu.

## 7. Ambiguity / escalation
- Nếu thấy 1 cụm đèn lơ lửng không biết của ai: Gắn nhãn, để `relevance=unknown`.
- Mọi case phân vân không thể dùng luật này để giải quyết -> Dùng tag `ESCALATE` vào object đó. Nhóm chấm điểm (QA) sẽ lọc các tag `ESCALATE` này.

## 8. Temporal rule
- Không áp dụng — bài toán hiện tại ưu tiên xử lý frame tĩnh. (Nếu dùng clip `lisa`, nhãn không cần link track giữa các frame).

## 9. Examples
| sample_id | Thấy gì | Expected output | Rule áp dụng |
|---|---|---|---|
| BDD07 | Giao lộ, 1 đèn đi thẳng sáng xanh, 1 đèn rẽ trái sáng đỏ. Xe đang đi thẳng | Box1: `state=green, relevance=relevant`. Box2: `state=red, relevance=not_relevant` | Xác định relevance dựa trên làn xe |
| BDD11 | Đèn xa tít, chỉ thấy chấm sáng, không thấy vỏ | Không label (IGNORE) | Ngoài scope vì quá nhỏ |

## 10. Common mistakes
1. **Chỉ vẽ quanh mỗi cái bóng đèn:** Lỗi phổ biến. Phải vẽ bao quanh CẢ CÁI VỎ NHỰA của đèn.
2. **Quên gán Relevance:** Mặc định CVAT có thể không bắt buộc, annotator hay quên chọn `relevant` hay `not_relevant`. Cần review kỹ.
