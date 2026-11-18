# Báo cáo Day 08 — 2A202603295 - Nguyễn Văn Hùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.30 (mặc định) | run_01 & run_02: YOLO26 nhận diện rất tốt các lớp phổ thông chuẩn COCO; ngưỡng 0.30 giữ cân bằng giữa precision và recall |
| three_wheeler | YOLOE-26s | 0.15 (từ sweep) | run_03: YOLOE dùng open-vocabulary tìm đúng tuk-tuk/ba gác; quét threshold_sweep đạt recall 100% ở ngưỡng 0.15 |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **MobileSAM (SAM 1, `mobile_sam.pt`, 38.8 MB)** vì máy tính RAM 8GB, chạy thuần CPU. MobileSAM cho tốc độ inference nhanh (< 2 giây/ảnh trên CPU), tốn rất ít RAM và chất lượng mask đủ ôm sát viền vật thể.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi bản chất từ từ vựng đóng (closed-set vocabulary) của COCO trong YOLO26: COCO không có nhãn xe ba bánh nên mô hình buộc phải gán vào car hoặc truck. Không thể sửa bằng cách tăng/giảm threshold mà bắt buộc phải dùng mô hình open-vocabulary (YOLOE) với text prompt riêng rồi gộp nhãn.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Trong `img_013.jpg`, xe tuk-tuk có conf rất cao (0.82) từ YOLO26 nhưng bị gán nhầm thành `truck`. Ngược lại, ở `img_004.jpg`, xe tuk-tuk từ YOLOE có conf thấp (0.18) nhưng lại bao trọn chính xác chiếc xe ba bánh.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Vì tập gold chỉ có 4 ảnh (49 đối tượng), kích thước mẫu nhỏ khiến phân bố chưa đại diện hết các góc chụp; việc tối ưu ngưỡng trên cùng tập dữ liệu này gây hiện tượng overfitting cục bộ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 492 | 492/520 = 94.6% | 520 box máy |
| edited | 4 | 4/520 = 0.8% | 520 box máy |
| deleted | 24 | 24/520 = 4.6% | 520 box máy |
| added | 26 | 26/522 = 5.0% | 522 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` có tỷ lệ deleted cao nhất (14/31 box máy bị xóa hoặc đổi lớp). Điển hình ở `img_013.jpg` và `img_014.jpg`, máy nhầm lẫn giữa thân xe có mui và xe tải nhỏ, đồng thời một số box bao trùm cả người lái xe.
- So với gold: precision / recall của **máy**: Precision đạt **50.0%** (37/74, KTC 95%: 39–61%), Recall đạt **75.5%** (37/49, KTC 95%: 62–85%) (mẫu số: 49 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **4/12 = 33.3%**; loại nào lọt: lọt 8 bẫy gồm 3 loose (`img_006.jpg`, `img_025.jpg`, `img_031.jpg`), 2 missing (`img_029.jpg`, `img_033.jpg`), 2 wrong_class (`img_010.jpg`, `img_024.jpg`), 1 extra (`img_003.jpg`); vì sao (bạn đã bỏ bước nào của checklist)? Người duyệt có tâm lý ỷ lại vào nhãn máy có sẵn (automation bias), đã bỏ qua bước 1 (quét mắt toàn bộ ảnh trước khi hiện nhãn máy) và không phóng to ảnh để soi kỹ độ khít của 4 cạnh viền (dẫn đến lọt toàn bộ 3 bẫy loose).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỷ lệ accepted rất cao (94.6%) đi kèm tỷ lệ bắt bẫy thấp (33.3%) chứng minh rõ ràng hiện tượng **thiên kiến tự động hóa**: người duyệt lướt qua quá nhanh, dễ dàng chấp nhận đề xuất của AI mà chưa thực sự kiểm tra chéo từng chi tiết.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **6.25 phút/ảnh** (25 phút / 4 ảnh) vs duyệt **0.83 phút/ảnh** (20 phút / 24 ảnh). Thiên lệch: khi gán gold từ đầu chưa quen ảnh nên thao tác chậm; khi duyệt batch đã quen khung cảnh và có sẵn box nháp nên tốc độ tăng gấp ~7.5 lần.
- Kết luận: **CÓ**, máy cực kỳ đáng dùng cho các lớp phổ thông (`person`, `motorbike`, `car`) vì tiết kiệm được hơn 85% thời gian gán nhãn, nhưng **phải có người giám sát chặt chẽ** đối với các lớp đặc thù (`three_wheeler`).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Luôn bấm ẩn nhãn (icon con mắt trên CVAT) trong 5 giây đầu để tự đếm số lượng người và phương tiện trước khi bật nhãn AI lên đối chiếu.
  2. Bắt buộc phóng to (zoom in) ít nhất 200% vào mọi đối tượng có gắn cờ `can_review` hoặc xe ba bánh để kiểm tra 4 mép viền.
  3. Duyệt ngược từ cuối danh sách (từ ảnh 24 về ảnh 1) để tránh mỏi mắt và mất tập trung ở các ảnh cuối lô.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | MobileSAM (SAM 1) | 2 (1 pos, 1 neg) | Tách riêng thân xe, không gồm người lái | Click âm ở góc cabin đã loại trừ được phần thân người lái | mask_01_tuktuk_img013.jpg |
| img_004.jpg | MobileSAM (SAM 1) | 2 (1 pos, 1 neg) | Ôm khít thân xe tuk-tuk nhỏ | Mask bị lẹm nhẹ ở phần bóng râm dưới gầm xe | mask_02_tuktuk_img004.jpg |
| img_009.jpg | MobileSAM (SAM 1) | 2 (1 pos, 1 neg) | Bao trọn khung xe ba gác và thùng hàng | Phần tay lái kim loại mỏng bị đứt đoạn mask | mask_03_threewheeler_img009.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc tách riêng người điều khiển phương tiện (person) và thân xe (three_wheeler) rất hữu ích; tuy nhiên SAM có xu hướng tự động gộp cả người lái nếu chỉ click 1 điểm dương ở giữa xe. Cần click âm vào người lái để tách biệt.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Thực hiện offline qua script `sam2_points.py` nhẹ và ổn định hơn rất nhiều so với chạy trực tiếp trong CVAT vì CVAT yêu cầu serverless Nuclio tiêu tốn nhiều RAM.

## 6. Calibration (Part 6)

- Số quyết định: **3** (`submission/qc/calibration_decisions.csv`); quan trọng nhất (dẫn `case_01`): Chốt dứt khoát phân loại mọi xe ba bánh (tuk-tuk, xích lô, xe ba gác) vào nhãn `three_wheeler`, không gán vào car/truck dù có mui hay thùng hàng lớn.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung hướng dẫn gom cụm đông đúc xa vào `ignore_region`, tách box `person` cho người điều khiển phương tiện và quy chuẩn nhận diện `three_wheeler`.
- Nguyên nhân gốc: Kết hợp cả 3 yếu tố: mô hình AI nhầm lẫn do thiếu từ vựng COCO, người duyệt bị thiên kiến tự động hóa, và guideline v1.0 chưa có quy định cụ thể cho vùng đám đông ở hậu cảnh.

## 7. QC và quyết định (Part 6)

- Mẫu: **5** ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: người duyệt tự kiểm tra lại bài của chính mình, dễ bỏ sót lại lỗi cũ).
- Lỗi theo **đối tượng**: **1/107 = 0.9%** · theo **ảnh**: **1/5 = 20.0%** (ngưỡng Release ≤ 2% theo ảnh).
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 lỗi ở `img_006.jpg` do box xe máy bị vẽ hơi lỏng phía đuôi).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên khoảng tin cậy Wilson lên tới **62%**. Do kích thước mẫu chỉ có 5 ảnh nên sai số thống kê rất lớn, chưa đủ độ tin cậy toán học để đảm bảo tỷ lệ lỗi toàn bộ lô thực sự ≤ 2%.
- Script kiểm được / không kiểm được: Script chỉ kiểm đếm số lượng lỗi dựa trên phiếu đánh giá của con người; script không tự động đánh giá được độ ôm khít pixel hay phát hiện lỗi người và máy cùng đồng thời bỏ sót.

**Quyết định: REWORK**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Guideline đã rõ ràng (không cần Hold). Tuy nhiên tỷ lệ ảnh có lỗi ở mẫu QC là 20.0% (> ngưỡng 2%) và tỷ lệ bắt bẫy chỉ đạt 33.3% (< ngưỡng 80%).
Nếu Rework / Hold, việc cần làm tiếp: Cần rà soát lại toàn bộ lô 24 ảnh, đặc biệt tập trung phóng to kiểm tra các box lỏng (loose) và quét kỹ các phương tiện bị bỏ sót ở ven đường.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng mô hình YOLO26 và YOLOE để sinh pre-label nháp; dùng MobileSAM để tạo mask. Mọi nhãn đều được kiểm tra đối chiếu qua script `validate_labels.py` và `review_stats.py`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Hoàn toàn **không**. 4 ảnh gold được tự tay vẽ 100% trên CVAT từ trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Máy Windows 11, RAM 8GB, CPU (không GPU rời). Đã tối ưu giới hạn 4GB RAM cho WSL2 qua `.wslconfig` để Docker chạy mượt mà.
- Dùng cache `data/cache/` thay vì tự chạy? Có sử dụng file cache đã chạy sẵn của YOLO26m và YOLOE trong `data/cache/` để tránh quá tải CPU và tiết kiệm thời gian, sau đó tự chạy các script quét ngưỡng và merge nhãn trên máy.
- Lỗi / bug script đã gặp (nếu có): Đã khắc phục lỗi `cvat-cli` do phiên bản server và client lệch nhau bằng cách gọi REST API trực tiếp qua Python script.
- Một điều bạn làm khác lần sau: Sẽ dành nhiều thời gian hơn để soi kỹ từng box lỏng và ẩn nhãn trước khi duyệt để nâng cao tỷ lệ bắt bẫy lên trên 80%.
