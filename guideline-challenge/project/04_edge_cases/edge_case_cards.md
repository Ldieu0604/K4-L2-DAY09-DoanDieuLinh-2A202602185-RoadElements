# Edge-case library

Thư viện tổng hợp 10 trường hợp biên (Edge Cases) thực tế cho bộ dữ liệu đèn tín hiệu giao thông (Traffic Light Dataset TL01 - TL30).

---

CASE ID: EC-01
Sample: TL01
Scene: Giao lộ đường đô thị ban ngày rõ nét
Observation: Đầu đèn tín hiệu chính đang bật sáng màu xanh (Green) rõ ràng trên giá treo
Decision: LABEL
Expected: Bounding box chữ nhật ôm sát vỏ ngoài (housing) của đầu đèn. Gán state=green, relevance=relevant, direction=all
Rationale: Xe ego chuẩn bị tiến vào giao lộ, đèn xanh điều khiển luồng thẳng của xe
Common mistake: Chỉ vẽ bao quanh vòng tròn bóng đèn phát sáng mà không vẽ bao trùm cả vỏ đèn
Diversity: normal

---

CASE ID: EC-02
Sample: TL02
Scene: Giao lộ ngã tư đông đúc ban ngày
Observation: Cụm đèn tín hiệu hiển thị màu ĐỎ (Red) kiểm soát luồng xe chạy thẳng
Decision: LABEL
Expected: Bounding box tight ôm khít vỏ đèn. Gán state=red, relevance=relevant, direction=all
Rationale: Đây là tình huống an toàn tối thượng (Critical Safety): Đèn đỏ của làn xe ego bắt buộc phải nhận diện chính xác 100% để kích hoạt phanh
Common mistake: Nhầm lẫn trạng thái màu đỏ thành màu vàng hoặc bỏ sót đèn đỏ
Diversity: critical

---

CASE ID: EC-03
Sample: TL03
Scene: Giao lộ giờ thấp điểm hoặc đèn nhánh phụ
Observation: Cụm vỏ đèn gắn trên cột nhưng không có bóng đèn nào phát sáng (đèn tắt / Off)
Decision: LABEL
Expected: Bounding box ôm sát vỏ đèn kim loại. Gán state=off, relevance=not_relevant, direction=all
Rationale: Mô hình downstream cần nhận biết sự hiện diện của đèn nhưng hiểu rằng đèn đang không hoạt động (off)
Common mistake: Bỏ qua không gán nhãn vì thấy đèn không sáng bóng nào
Diversity: ambiguity

---

CASE ID: EC-04
Sample: TL06
Scene: Giao lộ lớn có nhiều làn xe và nhiều đầu đèn cùng xuất hiện
Observation: Có 2 hoặc 3 đầu đèn treo cạnh nhau điều khiển các hướng rẽ và đi thẳng khác nhau
Decision: LABEL
Expected: Vẽ từng bounding box riêng biệt cho từng đầu đèn độc lập. Đèn làn xe ego gán relevance=relevant, đèn làn rẽ gán relevance=not_relevant
Rationale: Tránh xe tự hành phanh gấp giữa giao lộ khi đèn rẽ đỏ nhưng đèn làn xe ego đang xanh
Common mistake: Vẽ một bounding box lớn gộp chung cả 2-3 đầu đèn vào một khối
Diversity: conflict

---

CASE ID: EC-05
Sample: TL07
Scene: Đường đô thị có cây xanh hoặc biển báo che khuất
Observation: Đầu đèn tín hiệu bị nhánh cây hoặc biển chỉ dẫn che mất khoảng 30-40% phần thân vỏ
Decision: LABEL
Expected: Bounding box chỉ vẽ bao quanh phần vỏ đèn thực tế nhìn thấy được (Visible box). Gán state theo màu nhìn thấy, tích needs_review=true
Rationale: Downstream perception cần học phát hiện vật thể bị che khuất một phần (partial occlusion) dựa trên visual cues thực tế
Common mistake: Vẽ phỏng đoán (amodal box) to bao trùm xuyên qua cả cành cây hoặc bỏ qua không vẽ
Diversity: occlusion

---

CASE ID: EC-06
Sample: TL10
Scene: Khung cảnh góc nhìn xa trên đường thẳng
Observation: Đầu đèn tín hiệu nằm ở khoảng cách xa (> 80m), kích thước hiển thị nhỏ (khoảng 10x15 pixels)
Decision: LABEL
Expected: Phóng to (Zoom) giao diện CVAT để vẽ box chính xác bao quanh vỏ đèn; gán state theo màu đèn nhìn thấy
Rationale: Phát hiện đèn từ xa giúp hệ thống xe tự hành lập kế hoạch giảm tốc êm ái (Early Planning)
Common mistake: Bỏ sót không vẽ vì cho rằng đèn quá nhỏ
Diversity: small_far

---

CASE ID: EC-07
Sample: TL12
Scene: Chụp ngược sáng hoặc phơi sáng lâu làm ánh đèn phát quang mạnh
Observation: Ánh đèn sáng chói tạo quầng lóa (glare/halo) tỏa rộng ra xung quanh đầu đèn
Decision: LABEL
Expected: Bounding box ôm sát kích thước vật lý của vỏ đèn kim loại; tuyệt đối không kéo rộng bao phủ quầng sáng lóa
Rationale: Nếu vẽ bao trùm quầng lóa, mô hình perception sẽ ước lượng sai kích thước và khoảng cách chiều sâu 3D của đèn
Common mistake: Kéo box to gấp 2-3 lần kích thước thật của đầu đèn do quầng sáng
Diversity: low_visibility

---

CASE ID: EC-08
Sample: TL15
Scene: Giao lộ có cột đèn dành cho người đi bộ hoặc đèn quay lưng lại xe
Observation: Cột đèn thấp dành cho người đi bộ hoặc đầu đèn quay lưng lại hướng xe di chuyển
Decision: IGNORE
Expected: Bỏ qua hoàn toàn, không vẽ bất kỳ bounding box nào
Rationale: Scope bài toán chỉ bao gồm đèn điều khiển luồng ô tô của xe tự hành; vẽ đèn đi bộ hoặc đèn ngược chiều gây nhiễu
Common mistake: Nhìn thấy có đèn là vẽ box bất kể hướng quay và đối tượng điều khiển
Diversity: negative

---

CASE ID: EC-09
Sample: TL21
Scene: Giao lộ phức tạp, đèn ở khoảng cách xa và không phát sáng rõ (nghi ngờ tắt hoặc hỏng)
Observation: Đèn nằm ở vị trí khó phân biệt, không đủ bằng chứng kết luận đèn đang Off hay màu đèn bị chói nắng
Decision: ESCALATE
Expected: Vẽ box ôm đầu đèn, gán state=unknown, relevance=unknown và tích chọn needs_review=true
Rationale: Không đủ căn cứ quang học để kết luận chắc chắn; cần đánh dấu cờ để chuyên gia QA đối chiếu HD Map
Common mistake: Tự suy đoán theo cảm tính dẫn đến bất đồng ý kiến
Diversity: escalation

---

CASE ID: EC-10
Sample: TL23
Scene: Đèn tín hiệu giao lộ ban đêm / thiếu sáng
Observation: Đèn tín hiệu ĐỎ hiển thị rõ ràng ngay trên làn đường xe ego chuẩn bị tiến tới
Decision: LABEL
Expected: Bounding box tight ôm khít vỏ đèn. Gán state=red, relevance=relevant, direction=all
Rationale: Trường hợp Critical Escape rủi ro cao: Đèn đỏ bắt buộc nhận diện đúng 100%, sai sót sẽ dẫn đến vượt đèn đỏ
Common mistake: Gán nhầm trạng thái thành off hoặc green do tương phản ban đêm yếu
Diversity: critical
