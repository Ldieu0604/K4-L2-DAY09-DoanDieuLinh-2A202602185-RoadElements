# QA plan + quality gates

Các ngưỡng dưới đây là tiêu chí nội bộ đề xuất cho bài toán traffic-light state và ego relevance, không phải chuẩn
ngành. QA owner: Hoàng Công Chứ. Blind gold do Hoài Thanh + Tân chuẩn bị; chỉ một trong hai người giữ gold chạy
`make freeze`. QA owner không tự sửa gold sau freeze.

## Flow

Guideline v1 → calibration độc lập → chẩn đoán bất đồng → guideline v2 → freeze gold → blind handoff → chấm export
peer → guideline v3 nếu có gap → quality gate.

- **Reviewer và phạm vi:** Chứ điều phối QA và review export peer. Calibration cần ít nhất hai thành viên label độc
  lập; Chứ đối chiếu toàn bộ các dòng có `agree=0` trong `06_calibration_measure.csv`, chọn tối thiểu 3 bất đồng lớn
  nhất để ghi vào `06_calibration_report.csv`. Với blind, Chứ chấm từng gold decision trong `transfer_score.csv`;
  mở hình/export CVAT để kiểm geometry hoặc khi `peer_evidence` chưa đủ. Nếu có thể, người chấm không tự chấm lại
  annotation của chính mình.
- **Sampling:** Calibration review 100% ảnh thuộc split `calibration` và mọi bất đồng. Blind review 100% decision,
  đặc biệt mọi decision `critical`, `geometry`, `needs_review` hoặc ảnh `image_escalate`. Khi áp dụng cho production,
  review 100% ảnh có tag rủi ro (`critical`, `edge`, `ambiguity`, `occlusion`, `small_far`, `low_visibility`) và ảnh
  của annotator mới; lấy thêm mẫu ngẫu nhiên 20% từ phần còn lại, tối thiểu 5 ảnh hoặc toàn bộ nếu phần còn lại dưới
  5 ảnh. Ghi các sample ID đã review cùng kết quả trong log tương ứng.
- **Ghi và đóng issue:** Bất đồng calibration vào `06_calibration_report.csv`; câu hỏi của peer trong blind window
  vào `07_blind_handoff/clarification_log.csv` nguyên văn, không giải thích miệng; từng gold decision chấm trong
  `transfer_score.csv` (cột `correct` và `note`); feedback về tính dễ dùng phân loại trong `peer_feedback.md`. Lỗi
  chỉ đóng khi có diagnosis, action/owner, bằng chứng và kết quả kiểm tra lại. CVAT export ZIP calibration của từng
  người được giữ nguyên, đặt tên theo annotator (ví dụ `chu.zip`) tại `project/06_calibration_exports/`; không ghi đè
  export gốc. Xác nhận ZIP mở được và đúng định dạng trước khi chạy `make calib`.
- **Guideline gap / version:** Chứ ghi bằng chứng và đề xuất trong report/log; Linh (spec owner) quyết định wording
  quy tắc, cập nhật `02_guideline.md` và `08_revision_log.md`. Calibration dẫn tới v2; vấn đề phát hiện qua blind
  dẫn tới v3. Sau thay đổi, chạy lại review trên các ảnh/decision bị ảnh hưởng. Không sửa `gold_decisions.csv`,
  `sample_pack.csv` hoặc expected/severity trong `transfer_score.csv` sau freeze; nếu nghi gold sai, ghi `gold sai:`
  trong `note` cùng bằng chứng để debrief.

## Defect severity

| Severity | Định nghĩa cho project này                                                                                              | Ví dụ                                                                                                                                   | Action mặc định                                                                                                                               |
| -------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Critical | Lỗi có thể khiến hệ thống lập kế hoạch đi sai ở giao lộ hoặc dừng nhầm nghiêm trọng.                                    | Bỏ sót đèn đỏ relevant; gán đèn đỏ của làn khác thành relevant; nhầm state đỏ/xanh hoặc relevance làm đảo quyết định dừng/đi.           | Dừng gate, sửa/review 100% các ảnh cùng failure mode; đối chiếu với gold owner. Không được có critical escape khi PASS.                       |
| Major    | Làm thiếu hoặc sai thông tin cần cho quyết định nhưng chưa đảo trực tiếp quyết định dừng/đi trong trường hợp đã review. | Bỏ sót một đèn xe cơ giới trong scope nhưng không phải đèn chi phối ego; direction sai; gán `unknown`/`image_escalate` không theo rule. | Rework toàn bộ slice bị ảnh hưởng, coach nếu execution error; mở lại gate trước khi bàn giao.                                                 |
| Minor    | Sai geometry/metadata cục bộ, ý nghĩa chính của object vẫn giữ nguyên.                                                  | Box lệch quá 2 px so với mép housing nhìn thấy; box chưa tight nhưng vẫn bao đúng cụm đèn.                                              | Sửa object và kiểm tra mẫu cùng annotator/rule; ghi lại để theo dõi lặp lỗi.                                                                  |
| Question | Chưa đủ bằng chứng để phân loại lỗi hoặc quyết định theo guideline hiện tại.                                            | Không suy ra được làn ego đang đi; glare khiến màu không xác định; peer phải hỏi cách xử lý.                                            | Dùng `unknown`/`needs_review` hoặc `image_escalate` đúng scope; ghi clarification nếu phát sinh trong blind. Linh quyết định có cần đổi rule. |

## Metrics

| Metric                      | Cách tính                                                                                                                                              | Vì sao phù hợp với bài toán                                                           |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| Calibration exact agreement | Số dòng có `agree=1` chia tổng dòng so sánh trong `06_calibration_measure.csv`; báo thêm riêng theo `state`, `relevance`, `direction` và object count. | Chỉ ra attribute nào đang được hiểu khác nhau trước khi tạo blind test.               |
| Attribute completeness      | Số object có giá trị xác định cho cả `state`, `relevance`, `direction` chia tổng object được label; `__undefined__` tính là thiếu.                     | Thiếu thuộc tính làm output CVAT không dùng được dù box có mặt.                       |
| Blind decision accuracy (D) | Gold decision không phải geometry có `correct=1` chia tổng non-geometry decision trong `transfer_score.csv`.                                           | Đo khả năng peer áp dụng được các quyết định trong guideline.                         |
| Critical accuracy (C)       | Critical decision có `correct=1` chia tổng critical decision; critical escape rate = `1 - C`.                                                          | Ưu tiên lỗi ảnh hưởng trực tiếp tới dừng/đi và an toàn.                               |
| Geometry compliance (G)     | Geometry decision có `correct=1` chia tổng geometry decision; đối chiếu ảnh gốc và export, box phải theo visible housing và sai số mép không quá 2 px. | Box là đầu ra định vị cho downstream, không thể đánh giá chỉ bằng attribute.          |
| Transferability score (GTS) | Dùng `make gts`; GTS = 0.60D + 0.20C + 0.10G + 0.10I, trong đó I theo số câu hỏi clarification của peer.                                               | Tổng hợp độ chuyển giao guideline mà vẫn giữ trọng số cao cho quyết định và critical. |

## Quality gate

PASS/REWORK/REJECT dưới đây là ngưỡng đề xuất của nhóm; GTS do tool tính nhưng quyết định gate vẫn do nhóm xem cả
evidence và issue chưa đóng.

```text
PASS if:
  Calibration: exact agreement >= 90% overall và >= 95% riêng state/relevance sau khi chốt các bất đồng lớn;
  mọi object có đủ state/relevance/direction; không còn critical issue chưa xử lý trước freeze.
  Blind: GTS >= 85, C = 100%, G >= 90%, mọi clarification/decision đã được ghi và không còn major/critical issue mở.
REWORK if:
  Calibration hoặc blind không đạt một ngưỡng PASS nhưng nguyên nhân có thể sửa bằng coaching, thêm ví dụ/rule,
  hoặc sửa một slice xác định; review lại 100% slice bị ảnh hưởng và tính lại metric.
REJECT / ESCALATE if:
  Có critical escape (ngay lập tức), GTS < 70, cùng failure mode critical tái diễn sau một vòng rework,
  hoặc ảnh/rule thiếu bằng chứng để đưa ra quyết định an toàn. Escalate cho Linh và gold owner; không tự đoán.
```

**Trade-off:** Ngưỡng C = 100% nghiêm vì một nhãn sai state/relevance có thể đảo quyết định dừng/đi. Với bộ blind chỉ
4–5 ảnh, mỗi lỗi làm điểm biến động lớn; do đó không dùng GTS một mình để kết luận mà xem riêng critical, geometry,
clarification và ghi chú gold sai. Ngưỡng agreement 90% tổng thể tránh đòi đồng thuận tuyệt đối ở ca mơ hồ, còn
95% cho state/relevance tập trung vào thuộc tính trực tiếp ảnh hưởng downstream. Mẫu 20% production giữ chi phí
review vừa phải; toàn bộ ảnh rủi ro và mọi slice có lỗi vẫn được kiểm tra đầy đủ.
