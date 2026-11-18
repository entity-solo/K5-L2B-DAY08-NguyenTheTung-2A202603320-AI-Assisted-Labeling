# Báo cáo Day 08 — 2A202603320 - Nguyễn Thế Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.5 | Quét ngưỡng ở Part 3 đạt recall 90% (9/10 vật gold), precision 41%. Ở ngưỡng 0.25 (`r01`) bắt nhiều box nhiễu, còn ở 0.5 (`r02`) loại bỏ tốt các dự đoán ảo. |
| bicycle / motorbike / car / bus / truck | YOLO26m | 0.3 (mặc định) | Các lớp ít vật trong gold (< 5 vật như bus, truck, bicycle) không đủ mẫu để chọn ngưỡng riêng; lớp motorbike và car không đạt recall 90% ở bất kỳ ngưỡng nào <= 0.5 nên giữ mặc định 0.3 theo khuyến nghị của `threshold_sweep.py`. |
| three_wheeler | YOLOE-26s, prompts: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.3 (mặc định) | YOLO26 COCO hoàn toàn thiếu nhãn xe ba bánh bản địa; YOLOE ở `r03` bắt chính xác 8 xe ba bánh trên tập pilot mà YOLO26 bỏ qua. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1** (`sam2.1_b.pt`) vì cân bằng tối ưu giữa tốc độ suy luận trên CPU (~7.8s/ảnh) và chất lượng đường biên mask sắc nét ôm sát chi tiết phương tiện; không đòi hỏi quyền gated token phức tạp như SAM 3 và chất lượng vượt trội so với MobileSAM.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi YOLO26 nhận nhầm tuk-tuk/xe ba bánh thành `car` hoặc `truck` là lỗi thuộc về giới hạn closed-set (COCO 80 lớp không có nhãn ba bánh). Dù tăng hay hạ ngưỡng thì model vẫn gán nhầm với conf cao. Cách sửa: Phải dùng model open-vocabulary (YOLOE) với prompt văn bản đặc thù, sau đó dùng script `merge_prelabels.py` ưu tiên lớp `three_wheeler` đè lên box của YOLO26.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: `img_002.jpg` có xe tuk-tuk bị YOLO26 gán nhầm là `car` với conf lên tới 0.82.
  - Điểm thấp mà đúng: `img_011.jpg` có xe ba gác chở hàng ở góc xa cự ly trung bình chỉ đạt conf 0.28 nhưng YOLOE vẫn định vị chính xác.
- Giới hạn: Chọn ngưỡng trên chính 4 ảnh gold (in-sample) nên các chỉ số precision/recall trên gold là con số **lạc quan** (do chọn và chấm trên cùng dữ liệu); mẫu 4 ảnh chỉ mang tính ước lượng tương đối, chưa phản ánh hết sự đa dạng của 24 ảnh batch.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 386 | 90.6% | 386/426 box máy |
| edited | 12 | 2.8% | 12/426 box máy |
| deleted | 28 | 6.6% | 28/426 box máy |
| added | 63 | 13.7% | 63/461 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp **`three_wheeler`** máy tệ nhất: chỉ accepted 8 box trong khi có 6 box phải sửa, 11 box phải xoá và 7 box phải vẽ thêm (tỉ lệ can thiệp chỉnh sửa cao nhất). Nguyên nhân do góc chụp che khuất và mui bạt đa dạng của xe ba gác trong chợ đông (ví dụ ở `img_010.jpg`, `img_017.jpg`).
- Trong số deleted + added có 7 ca đổi lớp (máy gán sai lớp nhưng box đúng chỗ); số vật máy **sót thật** ≈ 63 - 7 = 56 đối tượng (chủ yếu là xe máy và xe đạp ở rìa xa).
- So với gold: precision của **máy** là **52.7%** (29/55 TP+FP, KTC 95%: 40–65%), recall của **máy** là **72.5%** (29/40 đối tượng gold, KTC 95%: 57–84%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 = 75.0%** (KTC 95%: 47–91%); trong đó: bắt trọn 3/3 bẫy missing (100%), 3/3 bẫy extra (100%), 2/3 bẫy loose (66.7%), và 1/3 bẫy wrong_class (33.3%).
- Các bẫy còn lọt:
  - `img_024.jpg` (`wrong_class`): xe ô tô con ở góc phải xa bị máy gán thành `truck`.
  - `img_014.jpg` (`loose`): box người ở góc trái bị lệch sang phải ~40px và hụt mép trên.
  - `img_022.jpg` (`wrong_class`): người đứng ở lề đường bên trái bị máy gán thành `bicycle`.
- Phân tích hiện tượng: Ở lượt duyệt 1, người duyệt bắt được 8/12 bẫy (66.7%). Sang lượt duyệt 2 sau khi rà soát độc lập, người duyệt bổ sung thêm nhiều xe máy/xe đạp và bắt thêm 1 bẫy missing, nâng kết quả lên **9/12 (75.0%)**. Tuy nhiên, 3 bẫy tinh vi trên vẫn bị lọt. Đây là minh chứng thực tế rõ ràng nhất của **Thiên kiến tự động hóa (Automation Bias)**: khi các gợi ý của AI có tỷ lệ accepted cao (> 90%), mắt người có phản xạ tin tưởng các box điểm `conf` cao và dễ lướt qua các lỗi sai lớp ở xa hoặc box bị lệch viền vài chục pixel nếu không chủ động zoom to 200%.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay thủ công: **4.0 phút/ảnh** (16 phút / 4 ảnh) vs duyệt và rework có pre-label: **1.46 phút/ảnh** (35 phút / 24 ảnh). Tốc độ vẫn nhanh hơn gấp **2.7 lần** so với vẽ tay hoàn toàn từ đầu.
- Thiên lệch: Có thiên lệch nhỏ do người duyệt đã làm quen với bối cảnh giao thông từ bước gán gold trước đó.
- Kết luận: **RẤT ĐÁNG DÙNG** cho các lớp phổ thông (`person`, `motorbike`, `car`) vì tiết kiệm hơn 60-70% thời gian; tuy nhiên **CẦN GIÁM SÁT CHẶT CHẼ** với lớp đặc thù (`three_wheeler`) và các vật thể nhỏ ở rìa ảnh.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Bắt buộc thực hiện Bước 0:* Ẩn toàn bộ nhãn máy (phím H), quét mắt độc lập tối thiểu 15 giây đếm nhẩm số lượng người và xe trước khi mở gợi ý của AI.
  2. *Quy tắc nghỉ giải lao định kỳ:* Bắt buộc dừng nghỉ 3-5 phút sau mỗi block 8-10 ảnh để tránh mỏi mắt dẫn đến mất tập trung.
  3. *Quy tắc kiểm tra chéo độc lập với điểm số:* Không cho phép bỏ qua kiểm tra đường biên với các box có `conf > 0.8`, luôn phóng to 200% để nắn biên ở các ca phương tiện chở hàng cồng kềnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click / prompt | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box + 2 pos + 3 neg clicks | **CÓ gồm người lái** (bị nuốt tài xế mặc áo tím vào mask xe) | SAM gặp khó khi bóc tách người lái ngồi trong cabin mở của xe tuk-tuk. Dù đã dùng box gợi ý và đặt 3 click âm ngay trên đầu/vai người lái, mask vẫn bị dính thân người tài xế vào thân xe và lem nhẹ sang bánh xe máy bên cạnh (vi phạm quy tắc 4.1.4 nếu không sửa tay). | `submission/masks/tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box + 1 pos click | Không gồm người lái (xe đỗ không người) | Bám sát viền xe tuk-tuk màu vàng xanh, biên dạng thân xe và bánh xe được tách rất sắc nét, không bị lem ra lòng đường hay xe lân cận. | `submission/masks/tuktuk_img004.jpg` |
| `img_031.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box + 4 pos + 1 neg clicks | **KHÔNG gồm hàng hoá** (bỏ sót mẹt hoa quả) | Căm xe / nan hoa bị thủng lỗ chỗ, viền răng cưa nham nhở do cấu trúc mảnh; SAM xem mẹt nhãn/cà chua cồng kềnh là vật thể rời nên không gom vào mask xe đạp (muốn gom hàng hoá theo quy tắc 4.1.4 phải tốn thêm nhiều click dương vào từng mẹt quả). | `submission/masks/bicycle_img031.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.1.4 (tách riêng người lái và phương tiện) rất hữu ích; tuy nhiên guideline còn thiếu quy định về việc mask polygon có nên đục lỗ qua các nan hoa xe đạp / khung hở của xe ba gác hay phủ kín khối (solid polygon).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Hoạt động rất mượt mà và trực quan, thao tác click chuột trực tiếp trên giao diện web nhanh và tiện lợi hơn chạy script offline do có phản hồi thị giác ngay lập tức.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** trong `submission/qc/calibration_decisions.csv`; quan trọng nhất là `CAL-01` (chuẩn hoá gán `three_wheeler` cho xe 3 bánh bị máy COCO gọi nhầm) và `CAL-02` (chốt ranh giới xe đạp điện vs xe máy điện).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Định nghĩa phân loại xe điện; chuẩn hoá quy tắc loại bỏ hàng cồng kềnh nhô quá 20% chiều dài xe; bắt buộc gán `three_wheeler` cho xe 3 bánh; siết chặt quy trình review 4 bước.
- Nguyên nhân gốc: Kết hợp cả ba yếu tố: guideline v1.0 cố ý để mở (chưa định nghĩa xe điện/hàng cồng kềnh); máy thiếu nhãn cục bộ; và người duyệt bị ảnh hưởng bởi thiên kiến tự động hóa.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh ngẫu nhiên từ 24 ảnh batch (seed 42); người kiểm: Nguyễn Thế Tùng (**hạn chế**: người kiểm định đồng thời là người đã duyệt lô ảnh).
- Lỗi theo **đối tượng**: **0/92 = 0.0%** (đạt tiêu chuẩn ≤ 2%).
- Lỗi theo **ảnh**: **0/5 = 0.0%** (đạt tiêu chuẩn ≤ 2%).
- Tỷ lệ bắt bẫy: **9/12 = 75.0%** (chưa đạt ngưỡng ≥ 80%).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên 95% khoảng tin cậy Wilson là **43%**. Với cỡ mẫu 5 ảnh, ngưỡng 2% về mặt lý thuyết xác suất không thể được chứng minh tuyệt đối.
- Script kiểm được / không kiểm được: Script đếm chính xác số liệu phiếu kiểm nhưng không tự phát hiện được các lỗi vi mô về ôm sát biên pixel hay các lỗi bị cả máy và người cùng bỏ sót.

**Quyết định cuối cùng: REWORK**, ngày 2026-11-17.
- Lý do (đi theo cây quyết định Hold → Rework → Release): Mặc dù trên mẫu kiểm tra QC 5 ảnh sạch lỗi (0.0% ảnh lỗi, 0.0% đối tượng lỗi), nhưng tỷ lệ bắt bẫy đạt **9/12 = 75.0%** vẫn nằm dưới ngưỡng tin cậy **80%** (cần tối thiểu 10/12 bẫy). Đi đúng theo cây quyết định kỹ thuật của môn học: khi bắt bẫy chưa đạt ngưỡng, quyết định trung thực và chuẩn xác nhất là **REWORK** để tiếp tục rà soát siết chặt chất lượng, kiên quyết không Release non khi chưa thỏa mãn toàn bộ tiêu chí.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Google Antigravity làm trợ lý phân tích tài liệu lab, chạy các script tự động hóa, tính toán thống kê và kiểm tra chéo các yêu cầu rubric; mọi kết quả đều được kiểm chứng và rà soát thủ công.
- Gold có bị nhiễm không: Hoàn toàn **không**; 4 ảnh gold được gán thủ công bằng tay 100% trước khi chạy bất kỳ model pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, chạy CPU (không có GPU CUDA), CVAT Docker v2.82.0 chạy local cổng 8080 rất mượt mà và ổn định.
- Dùng cache `data/cache/` thay vì tự chạy? Chạy trực tiếp các lệnh thử nghiệm trên tập pilot; sử dụng file cache sẵn có của repo cho tập 36 ảnh của YOLO26 và YOLOE để tối ưu hóa thời gian chạy CPU.
- Lỗi / bug script đã gặp: Model YOLOE yêu cầu git để tự động cài đặt repo CLIP trong lần chạy đầu tiên; lệnh SAM cần cung cấp bounding box gợi ý để định vị chuẩn xác điểm trung tâm vật thể.
- Một điều bạn làm khác lần sau: Sẽ dành thời gian phóng to quan sát kỹ hơn các góc khuất và thực hiện nghiêm ngặt quy tắc ẩn nhãn máy trước khi duyệt để không bị lọt bẫy tự động hóa.
