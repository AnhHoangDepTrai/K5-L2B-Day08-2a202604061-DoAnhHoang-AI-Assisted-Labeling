# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.15; lớp còn lại mặc định 0.3 | r01/r02: trên 4 ảnh gold, person recall 93% (14/15), precision 39% (14/36). Motorbike/car không đạt recall 90% với ngưỡng <=0.5; bicycle và bus không có mẫu, truck chỉ có 1. |
| three_wheeler | YOLOE-26s, prompt tuk-tuk / cyclo / cargo tricycle | 0.15 | r03: trên 7 vật gold, recall 100% (7/7), precision 78% (7/9); img_002 có box nhầm pickup 0.37. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<chưa chạy; điền sau Pha 4>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp `three_wheeler`, nên tuk-tuk bị bỏ sót hoặc gán `car`; dùng YOLOE để gợi ý lớp này rồi người duyệt xác nhận (r01–r04).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): YOLOE gán nhầm `three_wheeler` cho pickup ở img_002 với điểm 0.37 (r03).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: threshold_sweep chọn và đo trên cùng 4 ảnh; person có 15 mẫu, three_wheeler 7 mẫu, nên kết quả chỉ là ước lượng nội bộ, cần QC trên batch độc lập.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
