# Báo cáo Day 08 — 2a202604061 - Đỗ Anh Hoàng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.15; lớp còn lại mặc định 0.3 | r01/r02: trên 4 ảnh gold, person recall 93% (14/15), precision 39% (14/36). Motorbike/car không đạt recall 90% với ngưỡng <=0.5; bicycle và bus không có mẫu, truck chỉ có 1. |
| three_wheeler | YOLOE-26s, prompt tuk-tuk / cyclo / cargo tricycle | 0.15 | r03: trên 7 vật gold, recall 100% (7/7), precision 78% (7/9); img_002 có box nhầm pickup 0.37. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 Base là mask chính; MobileSAM được dùng làm đối chiếu vì nhẹ hơn và chạy nhanh trên CPU.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp `three_wheeler`, nên tuk-tuk bị bỏ sót hoặc gán `car`; dùng YOLOE để gợi ý lớp này rồi người duyệt xác nhận (r01–r04).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): YOLOE gán nhầm `three_wheeler` cho pickup ở img_002 với điểm 0.37 (r03).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: threshold_sweep chọn và đo trên cùng 4 ảnh; person có 15 mẫu, three_wheeler 7 mẫu, nên kết quả chỉ là ước lượng nội bộ, cần QC trên batch độc lập.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 522 | 522/627 = 83.3% | 627 box máy |
| edited | 35 | 35/627 = 5.6% | 627 box máy |
| deleted | 61 | 61/627 = 9.7% | 627 box máy |
| added | 83 | 83/640 = 13.0% | 640 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler` có 17 box bị xóa và 6 box thêm trong lượt review; YOLO26 không có lớp này nên dễ bỏ sót hoặc gán sai thành `car/truck`.
- So với gold: precision / recall của **máy** 45.1% / 68.1% (mẫu số: 47 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91.7%; loại còn lọt: `img_026.jpg` loại loose.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Việc rework cần kiểm lại từng cạnh viền và quét ảnh trước khi nhìn nhãn máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 35 phút cho 4 ảnh (8.75 phút/ảnh); duyệt 120 phút cho 24 ảnh (5 phút/ảnh).
- Kết luận: Có giá trị hỗ trợ người duyệt cho các lớp COCO phổ biến và cho SAM; không đáng tin làm nhãn hoàn toàn tự động cho `three_wheeler`, dựa trên recall gold 68.1%.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét toàn ảnh và đếm vật trước khi nhìn box AI.
  2. Kiểm từng lớp và 4 cạnh viền trước khi Accept.
  3. Tick `can_review` cho ca che khuất/box trùng và rà lại bằng Filters.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 Base | 3 | Có; mask bao quanh người lái/hành khách bên trong thân xe | Mask bao rộng vùng người lái, cần thêm điểm âm hoặc box; 317255 px, 1 polygon | submission/masks/tuktuk_img013_sam2.jpg |
| img_013.jpg | MobileSAM | 3 | Có; mask theo phần xe nhưng vẫn có vùng người/hành khách | 245541 px, 2 polygon; liền kém hơn SAM 2.1 | submission/masks/tuktuk_img013_sam1.jpg |
| img_004.jpg | SAM 2.1 Base | 3 | Mask ôm xe ba bánh; không tách riêng người lái ở một số tiếp giáp | 93190 px, 1 polygon; vùng mái/thân/bánh được bám tương đối rõ | submission/masks/tuktuk_img004_sam2.jpg |
| img_017.jpg | SAM 2.1 Base | 3 | Mask ôm xe ba bánh; người lái/hành khách chưa tách hoàn toàn | 84532 px, 1 polygon; điểm âm chưa loại hết phần người | submission/masks/tuktuk_img017_sam2.jpg |

- Đối chiếu guideline mục 4.2: quy tắc 2 điểm dương trên thân/mái và 1 điểm âm trên người lái hoặc nền giúp định hướng mask; quy tắc vẫn thiếu một hướng dẫn cụ thể khi mask dính người lái vì người nằm trong phần khung nhìn thấy của phương tiện.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa chạy Interactor trong CVAT ở bài này; offline bằng sam2_points.py đã cho phép thử nhanh SAM 2.1 và MobileSAM trên cùng tọa độ.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): xe ba bánh địa phương phải là `three_wheeler`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: phân loại `three_wheeler`, cụm vật quá nhỏ và box trùng/che khuất.
- Nguyên nhân gốc: sai dự đoán của model kết hợp ca chưa rõ trong guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người duyệt là Đỗ Anh Hoàng, tự kiểm trực tiếp overlay và ghi `khong` nếu không phát hiện lỗi.
- Lỗi theo **đối tượng**: 0/252 = 0.0% · theo **ảnh**: 0/8 = 0.0% (ngưỡng Release ≤ 2%)
- Top lỗi: chưa ghi nhận lỗi rõ ràng trong 8 ảnh mẫu.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): cận trên Wilson là 32%, nên 0 lỗi không chứng minh lô sạch dưới 2%.
- Script kiểm được / không kiểm được: script kiểm thống kê và định dạng; không thay thế người soát ôm sát từng object_id.

**Quyết định: Release**, ngày 2026-11-18. Lý do: bắt bẫy đạt 11/12, mẫu QC không ghi lỗi, không có lỗi hệ thống.
Phiếu QC đã được người duyệt điền đủ cho 8 ảnh.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): dùng YOLO26, YOLOE, SAM 2.1/MobileSAM và Copilot hỗ trợ phân tích; kết quả được kiểm bằng script và ảnh.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không ghi nhận nhiễm; gold được gán trước Pha 2–3.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows, Python 3.12.4, chạy CPU, CVAT Docker local.
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng cache làm nguồn chính; Pha 2–3 chạy model trong local.
- Lỗi / bug script đã gặp (nếu có): sai tên gold-manual/gold_manual và batch-review/batch_reviewed; đã đổi/copy theo chuẩn.
- Một điều bạn làm khác lần sau: ghi thời gian từng bước ngay khi chạy và kiểm từng object_id trong QC overlay.
