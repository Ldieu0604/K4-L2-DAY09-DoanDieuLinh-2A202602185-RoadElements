# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v2 | Sửa lỗi chính tả (elevance -> relevance, v.v.). Bổ sung quy định: Hướng (direction) được xác định theo góc nhìn của xe (ego vehicle) thay vì đoán hướng làn đường. | 2 bạn annotator trong nhóm bị bối rối không biết nên xác định hướng trái/phải theo người chụp hay theo chiều đường. | Phản hồi trực tiếp từ bước Calibration nội bộ. |
