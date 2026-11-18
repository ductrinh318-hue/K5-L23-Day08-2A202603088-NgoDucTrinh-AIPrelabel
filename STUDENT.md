# Thông tin học viên

- Họ và tên: Ngọ Đức Trinh
- MSHV: 2A202603088
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: ductrinh318-hue
- Repo bài nộp (URL): https://github.com/ductrinh318-hue/K5-L23-Day08-2A202603088-NgoDucTrinh-AIPrelabel/tree/main
- Commit chốt bài (hash): HEAD
- Máy: Windows 11, CPU (dùng Google Colab GPU T4 cho suy luận model)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446 / 42 / 195 (mẫu số: 703 box máy); added: 44 (mẫu số: 532 đối tượng)
- So với gold: precision / recall của **máy** 0.78 / 0.81
- Bắt bẫy: 9/12 = 75.0%
- Thời gian: gán tay 6.5 phút/ảnh; duyệt có pre-label 1.8 phút/ảnh
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/104)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy đạt 75.0% (9/12 bẫy, dưới ngưỡng 80%) và cần bổ sung rà soát lại các ca bẫy sót trước khi Release.