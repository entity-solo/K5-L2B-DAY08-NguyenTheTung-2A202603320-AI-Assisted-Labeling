# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603320 - Nguyễn Thế Tùng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy, bẫy lọt và phân tích review. Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp đúng một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân loại là `motorbike` nếu xe chạy hoàn toàn bằng động cơ điện (không có bàn đạp chân hoặc người lái để chân trên sàn xe); phân loại là `bicycle` nếu là xe đạp trợ lực có bàn đạp chân truyền thống. | `img_005.jpg`, `img_022.jpg` | `CAL-02` |
| 3 | Hàng hoá nhô ra cồng kềnh trên xe đạp / ba gác: box xe có gồm hàng không? | Bounding box của phương tiện chỉ bao quanh khung xe và giá chở hàng gắn cố định; phần hàng hóa cồng kềnh nhô ra ngoài > 20% chiều dài xe sẽ không tính vào box phương tiện. | `img_031.jpg`, `img_008.jpg` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_004.jpg`, `img_013.jpg` | Xe tuk-tuk / ba gác chở khách trên phố | YOLO26m nhận nhầm thành `car` hoặc `truck` do COCO thiếu lớp xe 3 bánh bản địa | Luôn phân loại là `three_wheeler` cho mọi phương tiện ba bánh (tuk-tuk, xe ba gác, xích lô), bất kể máy đề xuất `car` hay `truck`. | Mục 2 & Mục 4.3: Nhận diện và bắt buộc gán `three_wheeler` cho xe 3 bánh địa phương. (`CAL-01`) |
| `img_024.jpg`, `img_014.jpg` | Bẫy sai lớp (`wrong_class`) và box lỏng (`loose`) bị bỏ sót khi review | Máy hiển thị box có điểm `conf` cao nhưng lớp sai hoặc mép box quá rộng | Tuân thủ nghiêm ngặt bước 0: ẩn toàn bộ nhãn máy và quét ảnh độc lập trước khi kiểm tra từng gợi ý của AI để tránh thiên kiến tự động hóa. | Mục 3: Bổ sung quy trình kiểm soát chất lượng và quy tắc chống thiên kiến tự động hóa. (`CAL-04`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | Quy chuẩn ban đầu cho 36 ảnh phố châu Á. |
| v1.1 | - Chốt phân loại xe điện (`CAL-02`).<br>- Quy định ranh giới hàng hoá cồng kềnh nhô ra ngoài xe (`CAL-03`).<br>- Chuẩn hoá quy tắc ép lớp `three_wheeler` cho xe ba bánh địa phương bị máy COCO nhận nhầm (`CAL-01`).<br>- Bổ sung quy trình chống thiên kiến tự động hóa khi duyệt nhãn máy (`CAL-04`). | Hoàn thiện sau vòng review 24 ảnh batch, phân tích 12 ảnh bẫy và thống kê AI Error Profile. |
