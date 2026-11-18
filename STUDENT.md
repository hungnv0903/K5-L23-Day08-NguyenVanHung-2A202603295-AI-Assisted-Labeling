# Thông tin học viên

- Họ và tên: Nguyễn Văn Hùng
- MSHV: 2A202603295
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: hungnv0903
- Repo bài nộp (URL): https://github.com/hungnv0903/K5-L23-Day08-NguyenVanHung-2A202603295-AI-Assisted-Labeling
- Commit chốt bài (hash): [Chốt sau khi push]
- Máy: Windows 11, RAM 8GB, CPU (không GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 492/520 = 94.6% / 4/520 = 0.8% / 24/520 = 4.6% (mẫu số: 520 box máy); added: 26/522 = 5.0% (mẫu số: 522 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 50.0% (37/74) / 75.5% (37/49) (mẫu số: 49 đối tượng gold)
- Bắt bẫy: 4/12 = 33.3%
- Thời gian: gán tay 6.25 phút/ảnh (25 phút / 4 ảnh); duyệt có pre-label 0.83 phút/ảnh (20 phút / 24 ảnh)
- Lỗi QC: 20.0% theo ảnh (1/5), 0.9% theo đối tượng (1/107)
- **Quyết định:** REWORK — lý do một dòng: Tỷ lệ ảnh có lỗi 20% > 2% và tỷ lệ bắt bẫy 33.3% < 80% do ảnh hưởng của thiên kiến tự động hóa
