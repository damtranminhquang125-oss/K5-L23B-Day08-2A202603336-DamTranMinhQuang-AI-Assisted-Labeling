# Báo cáo Day 08 — 2A202603336 - Đàm Trần Minh Quang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.25 | r01,2,yolo26-m,--model yolo26 --size m --split pilot --conf 0.25 --out work/prelabel/t_y26,pilot,215,0.73,"Phát hiện được người, xe máy và ô tô trong nhiều ảnh; bắt được phương tiện ở cảnh đông.","Tuk-tuk thường bị gán car/truck thay vì three_wheeler; cảnh đông có nhiều box person/motorbike chồng lặp hoặc dư.",,work\prelabel\t_y26,cpu,8,0,umbrellax10;handbagx2;traffic lightx2;backpackx2, |
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.5 | r02,2,yolo26-m,--model yolo26 --size m --split pilot --conf 0.5 --out work/prelabel/t_y26_c05,pilot,139,0.72,"Ngưỡng 0.5 giảm nhiều box điểm thấp và box nhiễu so với conf 0.25; các vật nổi bật vẫn được phát hiện.","Vẫn bỏ sót người và xe nhỏ ở xa; tuk-tuk vẫn bị gán car/truck, và một số box còn chồng lặp.",,work\prelabel\t_y26_c05,cpu,8,0,umbrellax5;handbagx2;traffic lightx1;backpackx1, |
| three_wheeler | YOLOE-26s | 0.25 | r03,2,yoloe-s,--model yoloe --classes three_wheeler --split pilot --conf 0.25 --out work/prelabel/t_yoloe,pilot,8,0.28,"Phát hiện được tuk-tuk xanh ở img_002 và các xích lô ở img_032.","Chỉ có 8 box trong 8 ảnh; bỏ sót nhiều xe ba bánh, có box nhầm ở img_002 và img_011; vài box khá rộng, dính người hoặc nền.",,work\prelabel\t_yoloe,cpu,8,0,, |
| person / motorbike / car | YOLO26m + SAM 2.1 B | 0.25 | r04,2,yolo26-m+sam(sam2.1_b.pt),"--model yolo26 --size m --classes person,motorbike,car --images img_004.jpg img_013.jpg --conf 0.3 --segmenter sam --sam sam2.1_b.pt --out work/prelabel/t_sam",custom,16,5.61,"Tạo được mask cho xe và người ở img_004, img_013; mask bám gần đúng hình bao ngoài của một số vật chính.","Ở img_013, tuk-tuk bị gán car và mask lẫn người lái, hành khách cùng vùng nền; ở img_004, box/mask car chồng lên xe ba bánh.",,work\prelabel\t_sam,cpu,2,16,umbrellax3;handbagx1;bottlex8;chairx1, |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 B: Chất lượng mask ở mức ổn, tốc độ nhanh, RAM 32gb nhưng không có CPU, Không phải đăng ký tài khoản Hugging Face để dùng như SAM 3
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 gán xe tuk-tuk thành car (ví dụ ở img_013.jpg). Đổi ngưỡng chỉ lọc box, không sửa được nhãn; dùng YOLOE với prompt tuk-tuk=three_wheeler, ưu tiên kết quả đó khi gộp, rồi rà sửa thủ công trong CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Ở img_032.jpg, YOLOE nhận đúng xích lô bên phải là three_wheeler dù confidence chỉ 0.27.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Ngưỡng được chọn trên chính 4 ảnh gold nên kết quả đo trên gold có thể lạc quan, không còn là đánh giá độc lập. Chọn ngưỡng trên pilot/dev; giữ gold để đánh giá cuối. Nếu đã dùng gold để chọn ngưỡng, cần một bộ hold-out khác để đánh giá.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 383 | 383/510 = 75.1% | 510 box máy |
| edited | 64 | 64/510 = 12.5% | 510 box máy |
| deleted | 57 | 57/510 = 11.2% | 510 box máy |
| added | 44 | 44/493 = 8.9% | 493 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler — chỉ 8 box được giữ nguyên, 10 cần sửa và 25 bị xoá (cao nhất). Tuk-tuk dễ bị nhầm thành car: thấy ví dụ ở img_004.jpg và img_013.jpg, nơi box car chồng lên xe tuk-tuk. YOLO26 không có lớp tuk-tuk riêng; YOLOE có thể nhận lớp này nhưng box vẫn cần kiểm tra và chỉnh.
- So với gold: precision của **máy** 67,7%, recall 89,8%; mẫu số recall là 49 đối tượng gold (TP=44, FP=21, FN=5).
## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 12/12 (100%); loại nào lọt: không có; vì sao: không có bẫy nào bị bỏ sót nên chưa thấy bước checklist nào bị bỏ. Bắt đủ 3/3 ở cả wrong_class, missing, extra, loose. Tuy vậy, chỉ có 12 bẫy nên chưa thể khẳng định cách duyệt luôn hiệu quả.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 75,1% nhưng bắt bẫy 100% cho thấy cách em duyệt giữ được nhiều box máy đúng, đồng thời vẫn phát hiện các lỗi được cài vào. Nếu accepted cao nhưng bắt bẫy thấp thì có thể do quá tin nhãn máy hoặc lướt checklist; ngược lại, accepted thấp nhưng bắt bẫy cao có thể là duyệt kỹ nhưng sửa/xoá nhiều—cần phân biệt sửa có căn cứ với sửa quá tay.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 6 phút/ảnh vs duyệt 3 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): 50%
- Kết luận: Có, dùng máy như trợ lý pre-label chứ không thay người duyệt. Trên 4 ảnh gold, precision 67,7% (44 TP, 21 FP) và recall 89,8% (44/49); trong 24 ảnh, 75,1% box được giữ nguyên. Tuy nhiên, three_wheeler yếu nhất (25 deleted, 10 edited), nên cần kiểm tra kỹ lớp này. Số gold ít và ngưỡng được chọn trên chính gold, nên kết quả có thể lạc quan
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi bật nhãn máy, quét ảnh gốc và ghi nhận đối tượng; mục tiêu 24/24 ảnh được xem trước.
  2. Dùng checklist cố định để kiểm tra lớp, box và vật nhỏ/bị che; kiểm riêng mọi box three_wheeler.
  3. Kiểm tra ngẫu nhiên .../24 ảnh sau duyệt bằng người thứ hai; ghi lỗi và giữ lô để sửa nếu tỷ lệ lỗi vượt ngưỡng QC 2%.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004 | SAM 2.1 B | 3 | Không thấy người lái/hàng hoá rõ trong mask này | Mask chỉ phủ một phần xe; có rò ra cây/nền, điểm dương đặt ngoài xe | submission\masks\tuktuk_img004.jpg |
| img_013 | SAM 2.1 B | 3 | Có — mask còn dính người lái và hành khách; không thấy hàng hoá rõ | Chưa tách người khỏi xe sạch; có lỗ/rìa chưa gọn quanh khoang xe | submission\masks\tuktuk_img013.jpg |
| img_017 | SAM 2.1 B | 3 | Không thấy người đi bộ bên ngoài bị gộp; không thấy hàng hoá rõ | Mask bám phần lớn xe nhưng rò lên mái che/nền tại điểm click | submission\masks\tuktuk_img017.jpg |

- Đối chiếu guideline mục 4.2: Quy tắc hữu ích là mask bám phần nhìn thấy, loại bóng đổ, kính vẫn thuộc xe, và hai vật chạm nhau phải có mask riêng, không chồng lấn. Ở img_013, người lái/hành khách cần được tách khỏi mask xe. Guideline còn thiếu hướng dẫn cụ thể cho người ngồi trong xe ba bánh và hàng hoá nhô ra; cần chốt thêm trong guideline đã hiệu chỉnh.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): CAL-01 — YOLO26 gán tuk-tuk thành car; phải kiểm tra và sửa thành three_wheeler, không đổi ngưỡng để sửa nhầm lớp.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: rà box car/truck có hình xe ba bánh và đối chiếu YOLOE hoặc sửa tay; với tuk-tuk có mui, gán person nếu thấy ít nhất 20%; mask xe gồm kính nhưng tách người thành mask riêng, không chồng lấn.
- Nguyên nhân gốc: kết hợp — giới hạn của máy ở CAL-01 và các điểm guideline chưa chốt về người trong xe có mui/kính ở CAL-02, CAL-03. Chưa có bằng chứng cho thấy nguyên nhân là chưa nắm quy tắc.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt: sai_lop, sai_vi_tri_kich_thuoc).
- Lỗi theo **đối tượng**: 2/155 = 1.3%>> · theo **ảnh**: 1/8 = 12.5%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: sai lớp — 1 lỗi, img_013.jpg, ID 130; sai vị trí/kích thước — 1 lỗi, cùng ảnh, ID 136.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): cận trên Wilson 95% của tỷ lệ ảnh lỗi là 47%; mẫu 8 ảnh quá nhỏ để chứng minh đạt ngưỡng 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): cấu trúc file, lớp hợp lệ, box trong ảnh và số liệu QC. Không kiểm được: box có ôm sát vật thật, nhãn có đúng thực tế, vật bị cả người lẫn máy bỏ sót, hay bạn có thực sự theo checklist.

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): REWORK, ngày 2026-11-18. Lý do: tỷ lệ ảnh lỗi 12,5% vượt ngưỡng 2%.
Nếu Rework / Hold, việc cần làm tiếp: Xác nhận ID 130 có phải minivan (để sửa lớp nếu đúng) và chỉnh box ID 136; cập nhật nhãn, chạy lại build_release.py, rồi tạo mẫu QC mới bằng qc_sample.py --force và kiểm lại.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Github Copilot, kiểm tra số liệu, đọc file
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: 32GB/CPU/CVAT 2.81
- Dùng cache `data/cache/` thay vì tự chạy? Không
- Lỗi / bug script đã gặp (nếu có): Không
- Một điều bạn làm khác lần sau: <<>>
