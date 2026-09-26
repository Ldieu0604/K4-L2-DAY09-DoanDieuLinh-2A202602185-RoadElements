# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` khớp từng dòng ở đây.

## Ontology table

| Name             | Geometry  | Type (class / attribute) | Allowed values                                                 | Default         | Mutable? | Rationale                                                                          |
| ---------------- | --------- | ------------------------ | -------------------------------------------------------------- | --------------- | -------- | ---------------------------------------------------------------------------------- |
| `traffic_light`  | rectangle | class                    | n/a                                                            | n/a             | false    | Thực thể vật lý độc lập cần bounding box 2D phát hiện vị trí vỏ đèn                |
| `state`          | n/a       | attribute                | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown`    | `__undefined__` | false    | Trạng thái phát sáng của đèn; ảnh hưởng trực tiếp hành vi Stop/Go của xe           |
| `relevance`      | n/a       | attribute                | `__undefined__`, `relevant`, `not_relevant`, `unknown`         | `__undefined__` | false    | Đèn có chi phối làn đường xe ego đang đi hay không (tránh dừng nhầm vì đèn làn rẽ) |
| `direction`      | n/a       | attribute                | `__undefined__`, `straight`, `left`, `right`, `all`, `unknown` | `__undefined__` | false    | Mũi tên chỉ hướng hoặc hình tròn (`all`) mà đèn áp dụng                            |
| `needs_review`   | n/a       | attribute                | `false` (unchecked / checked)                                  | `false`         | false    | Checkbox đánh dấu vật thể nghi ngờ/tranh chấp để Lead Reviewer kiểm tra            |
| `image_escalate` | tag       | tag (image-level)        | n/a                                                            | n/a             | false    | Gán cho cả ảnh khi khung cảnh thiếu ngữ cảnh nghiêm trọng (mất làn, mất dấu vết)   |

## Class hay attribute

- **Vì sao `traffic_light` là class duy nhất?**
  Mỗi đầu đèn là một thực thể vật thể vật lý xác định có geometry hình chữ nhật bao quanh. Nếu tách thành nhiều class như `traffic_light_red_relevant_left`, taxonomy sẽ bùng nổ hàng chục class phức tạp gây nhầm lẫn khi annotator chọn trên thanh công cụ.
- **Vì sao `state`, `relevance`, `direction` là attribute?**
  Đây là các đặc tính độc lập của cùng một thực thể đèn giao thông. Phân tách thành attributes dạng dropdown giúp annotator thao tác tuần tự từng bước trên sidebar.
- **Default value và rủi ro bias:**
  Tất cả các attribute dạng select đều có giá trị mặc định là `__undefined__`. Nếu đặt mặc định là `green` hoặc `relevant`, khi annotator lơ đễnh bỏ qua không chọn, hệ thống sẽ tự động gán giá trị sai mà không phát hiện được. Việc để `__undefined__` đứng đầu danh sách buộc annotator phải chủ động suy nghĩ và chọn giá trị; bất kỳ export nào còn sót `__undefined__` sẽ bị QA bắt lỗi ngay lập tức.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): v2.74.1 (hoặc v2.76.0 chạy trên Docker Desktop local)
- **Tên task calibration**: `team01-calib-v1`
- **Guide của task đã dán `02_guideline.md`?**: Có (đã copy toàn bộ Markdown vào Task Description / Guide)
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape**, vì toàn bộ dataset BDD100K sử dụng là các ảnh tĩnh độc lập (single frames). Không có chuỗi frame liên tiếp nên không dùng interpolation track.

## Setup test

- **Người thực hiện test:** Thành viên 3 (chưa tham gia tạo schema CVAT).
- **Quy trình kiểm tra:** Mở giao diện CVAT job trên trình duyệt Chrome, quan sát thanh công cụ và sidebar.
- **Kết quả trả lời của người test:**
  - Nhận biết ngay cần dùng công cụ **Draw new rectangle** (phím tắt `N` hoặc click biểu tượng hình chữ nhật bên trái).
  - Chọn label `traffic_light` và vẽ bao quanh vỏ đèn.
  - Sau khi vẽ, chuyển sang sidebar bên phải hoặc chế độ **Attribute annotation** để điền lần lượt 3 dropdown: `state`, `relevance`, `direction`.
  - Nếu gặp đèn bị mờ hoặc che khuất nghi vấn: tích checkbox `needs_review`.
  - Nếu toàn bộ ảnh bị tuyết phủ không thấy giao lộ: dùng công cụ **Setup tag** chọn `image_escalate`.
- **Ghi nhận vướng mắc:** Lần đầu click dropdown bằng chuột đôi khi bị lag không ăn; giải pháp khắc phục là click vào ô và dùng phím mũi tên `↓` + `Enter`.
