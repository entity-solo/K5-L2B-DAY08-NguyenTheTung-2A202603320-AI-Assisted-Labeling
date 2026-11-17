# Thông tin học viên

- Họ và tên: Nguyễn Thế Tùng
- MSHV: 2A202603320
- Lớp (L2 / L3 / khác): K5-L2B
- Tài khoản GitHub: entity-solo
- Repo bài nộp (URL): https://github.com/entity-solo/K5-L2B-DAY08-NguyenTheTung-2A202603320-AI-Assisted-Labeling
- Commit chốt bài (hash): b38f385
- Máy: Windows, không có GPU (dùng CPU)
- CVAT (Docker): v2.82.0 (local http://localhost:8080)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 400 / 9 / 17 (mẫu số: 426 box máy); added: 40 (mẫu số: 449 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 52.7% (29/55) / 72.5% (29/40)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 4.0 phút/ảnh; duyệt có pre-label 0.96 phút/ảnh
- Lỗi QC: 40.0% theo ảnh (2/5), 2.2% theo đối tượng (2/92)
- **Quyết định:** REWORK — lý do một dòng: Tỷ lệ ảnh có lỗi QC (40.0% > 2%) và tỷ lệ bắt bẫy (66.7% < 80%) chưa đạt ngưỡng do ảnh hưởng của thiên kiến tự động hóa.
