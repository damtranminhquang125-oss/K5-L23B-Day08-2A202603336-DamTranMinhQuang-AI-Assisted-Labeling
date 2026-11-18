# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603336 - Đàm Trần Minh Quang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người ngồi trong tuk-tuk có mui, chỉ thấy một phần | Gán `person` khi ít nhất 20% người nhìn thấy; không suy đoán phần bị che. Nếu không chắc đạt ngưỡng, đánh `can_review`. | img_013 | CAL-02 |
| 5 | Kính trong suốt trong mask xe | Kính chắn gió/cửa sổ là phần xe và thuộc mask xe. Người nhìn thấy qua kính là vật riêng; mask phần người nhìn thấy, không chồng với mask xe. | img_013 | CAL-03 |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc bổ sung |
|---|---|---|---|---|
| img_013 | Tuk-tuk bị gán `car` | YOLO26m dự đoán `car`. SAM tạo mask từ box, không tự sửa nhãn. | Sửa nhãn thành `three_wheeler`; đối chiếu YOLOE hoặc sửa tay, không dùng đổi ngưỡng để xử lý nhầm lớp. | Mục 2: rà mọi box `car`/`truck` có hình xe ba bánh; xác nhận thì sửa sang `three_wheeler`. Nếu không đủ dấu hiệu để xác định, đánh `can_review`. SAM chỉ tạo mask, không xác nhận lớp. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt ngưỡng người nhìn thấy trong xe có mui; chốt cách mask kính và người; bổ sung bước rà box car/truck có thể là xe ba bánh. | img_013; CAL-01, CAL-02, CAL-03 |
