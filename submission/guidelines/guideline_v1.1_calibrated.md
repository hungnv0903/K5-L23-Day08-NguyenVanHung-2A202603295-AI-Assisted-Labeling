# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603295 - Nguyễn Văn Hùng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Cụm đám đông người và xe máy ở cự ly xa quá dày đặc có tách từng box không? | Không tách từng box nhỏ < 20px; dùng 1 box ignore_region bao trọn toàn bộ cụm đông ở hậu cảnh | img_005.jpg | case_02 |
| 2 | Xe ba bánh chở hàng / tuk-tuk có mui che: phân loại vào car hay three_wheeler? | Toàn bộ phương tiện 3 bánh (tuk-tuk, cyclo, xe ba gác) quy về three_wheeler, không gán car/truck | img_013.jpg | case_01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk chở khách có cabin và mui | YOLO26 gán nhầm thành car/truck | Sửa thành three_wheeler, vẽ thêm box person cho người lái | Mục 2.7: Mọi loại xe 3 bánh đều là three_wheeler; tách riêng người lái |
| img_005.jpg | Đám đông dày đặc ở hậu cảnh xa | YOLOE và YOLO26 sinh nhiều box nhỏ chồng chéo | Xóa các box nhỏ < 20px, vẽ 1 box ignore_region bao quanh | Mục 2.8: Vùng đám đông dày đặc ở xa không thể tách từng vật quy về ignore_region |
| img_023.jpg | Xe máy và người lái xe trên đường | YOLO26 vẽ 1 box motorbike gộp cả người lái lẫn xe | Tách thành 2 box riêng: 1 box person và 1 box motorbike | Mục 2.1 & 2.3: Người ngồi trên xe máy/xe đạp bắt buộc vẽ box person riêng ôm toàn thân |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Quy định rõ chuẩn hóa nhãn three_wheeler; hướng dẫn dùng ignore_region cho đám đông xa; bắt buộc tách box person cho người điều khiển phương tiện | Khắc phục lỗi nhầm lẫn của mô hình YOLO26 và làm rõ các ca mập mờ trong thực tế (case_01, case_02, case_03) |
