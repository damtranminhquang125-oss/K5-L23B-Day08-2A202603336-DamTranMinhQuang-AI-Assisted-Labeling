# Thông tin học viên

- Họ và tên: Đàm Trần Minh Quang
- MSHV: 2A202603336
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: damtranminhquang125-oss
- Repo bài nộp (URL): https://github.com/damtranminhquang125-oss/K5-L23B-Day08-2A202603336-DamTranMinhQuang-AI-Assisted-Labeling
- Commit chốt bài (hash):
- Máy: Windows, CPU (không có GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 383 / 64 / 57 (mẫu số: 510 box máy); added: 44 (mẫu số: 493 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 67.7% / 89.8% (49 đối tượng gold)
- Bắt bẫy: 12/12 = 100%
- Thời gian: gán tay 6 phút/ảnh (25 phút / 4 ảnh); duyệt có pre-label chưa ghi trong `time_log.csv`
- Lỗi QC: 12.5% theo ảnh (1/8), 1.3% theo đối tượng (2/155)
- **Quyết định:** Rework — lỗi xuất hiện ở 1/8 ảnh QC, vượt ngưỡng Release 2%
