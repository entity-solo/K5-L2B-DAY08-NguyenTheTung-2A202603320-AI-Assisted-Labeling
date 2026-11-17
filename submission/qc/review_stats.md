# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 426; đối tượng sau duyệt: 449; ngưỡng accepted: IoU >= 0.9
- Đã **loại 19 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 400 | 400/426 = 93.9% | 426 box máy |
| edited (sửa) | 9 | 9/426 = 2.1% | 426 box máy |
| deleted (xoá) | 17 | 17/426 = 4.0% | 426 box máy |
| added (vẽ thêm) | 40 | 40/449 = 8.9% | 449 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 35.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 188 | 1 | 1 | 18 |
| bicycle | 6 | 0 | 3 | 2 |
| motorbike | 132 | 3 | 3 | 7 |
| car | 52 | 0 | 2 | 3 |
| bus | 3 | 0 | 0 | 2 |
| truck | 5 | 0 | 2 | 2 |
| three_wheeler | 14 | 5 | 6 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 40 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 26 | 11 | 52.7% (KTC 95%: 40–65%) | 72.5% (KTC 95%: 57–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - loose: 2/3
  - extra: 3/3
  - missing: 2/3
  - lọt: img_024.jpg (wrong_class), img_014.jpg (loose), img_035.jpg (missing), img_022.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
