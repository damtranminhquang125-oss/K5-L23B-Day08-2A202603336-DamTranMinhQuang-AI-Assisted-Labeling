# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 155 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 2/155 = 1.3% | 155 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 1/8 = 12.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 383/510 = 75.1% / 64/510 = 12.5% / 57/510 = 11.2% | 510 box máy |
| Máy: added | 44/493 = 8.9% | 493 đối tượng sau duyệt |
| Bắt lỗi bẫy | 12/12 = 100.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_lop | 1 | img_013.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_013.jpg |

## Giới hạn thống kê
- 1 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **47%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 12.5% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
