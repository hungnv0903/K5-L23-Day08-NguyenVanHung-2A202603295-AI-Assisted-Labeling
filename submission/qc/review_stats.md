# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 520; đối tượng sau duyệt: 522; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 492 | 492/520 = 94.6% | 520 box máy |
| edited (sửa) | 4 | 4/520 = 0.8% | 520 box máy |
| deleted (xoá) | 24 | 24/520 = 4.6% | 520 box máy |
| added (vẽ thêm) | 26 | 26/522 = 5.0% | 522 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 18.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 268 | 2 | 1 | 1 |
| bicycle | 7 | 0 | 1 | 2 |
| motorbike | 133 | 0 | 2 | 4 |
| car | 51 | 0 | 3 | 11 |
| bus | 2 | 0 | 1 | 2 |
| truck | 4 | 0 | 2 | 4 |
| three_wheeler | 27 | 2 | 14 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 49 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 37 | 12 | 50.0% (KTC 95%: 39–61%) | 75.5% (KTC 95%: 62–85%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 4/12 = 33.3% (KTC 95%: 14–61%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - extra: 2/3
  - wrong_class: 1/3
  - missing: 1/3
  - lọt: img_006.jpg (loose), img_003.jpg (extra), img_024.jpg (wrong_class), img_031.jpg (loose), img_033.jpg (missing), img_010.jpg (wrong_class), img_029.jpg (missing), img_025.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
