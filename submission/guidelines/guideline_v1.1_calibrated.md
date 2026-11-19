# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2a202604061 - Đỗ Anh Hoàng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe ba bánh địa phương được xếp như thế nào? | Nếu có ba bánh hoặc xe ba gác/tuk-tuk/xích lô, dùng `three_wheeler`; không dùng `car`/`truck`/`motorbike` của YOLO26. | img_002.jpg | CAL-01 |
| 2 | Cụm vật quá nhỏ trong ảnh đỗ xe máy dày đặc thì sao? | Vật rõ, cao >=20 px được gán từng cái; cụm cao dưới 20 px hoặc không thể tách thì dùng một `ignore_region`. | img_005.jpg, img_018.jpg | CAL-02 |
| 3 | Hai box xe ba bánh chồng nhau hoặc che người/hàng hóa thì sao? | Giữ hai đối tượng riêng khi rõ ràng; xóa duplicate chỉ khi cùng vật. Tick `can_review` cho ca che khuất hoặc hai vật khó tách. | img_018.jpg | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002.jpg | Tuk-tuk bị YOLO26 bỏ sót hoặc gọi sai lớp. | YOLO26m detector COCO. | YOLOE thêm ứng viên `three_wheeler`, người duyệt giữ box đúng và sửa lớp. | Lớp mục 2: ưu tiên `three_wheeler` cho xe ba bánh địa phương. |
| img_005.jpg | Cụm motorbike/person dày đặc và vật nhỏ. | YOLO26 tạo nhiều box chồng lấn ở vùng đông. | Người duyệt dùng `ignore_region` cho cụm không thể tách; box rõ vẫn gán từng vật. | Quy tắc mục 4.1.6; thêm ngưỡng dày đặc gần camera. |
| img_018.jpg | Box chồng trên tuk-tuk/xe máy/person. | YOLO26 và YOLOE có thể sinh box trùng hoặc sai lớp. | Xóa duplicate, giữ người và xe riêng, tick `can_review` nếu che >20%. | Mục 4.1.4 và 4.3. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt xe ba bánh, cụm vật quá nhỏ và quy tắc xử lý box trùng/che khuất. | CAL-01, CAL-02, CAL-03 |
