# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603088 - Ngọ Đức Trinh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (img_0xx) và khớp một dòng trong qc/calibration_decisions.csv.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | case_id |
|---|---|---|---|---|
| 1 | Xe đạp điện: gán Bicycle hay motorbike? | Gán Bicycle nếu có bàn đạp lộ rõ và kiểu dáng khung xe đạp truyền thống; gán motorbike nếu không có bàn đạp, thân vỏ kín như xe tay ga. | img_004.jpg | CAL-01 |
| 2 | Xe xích lô và xe ba gác máy phân tách thế nào? | Xe xích lô (chạy bằng sức người đạp) thuộc lớp Bicycle; xe 3 bánh có động cơ (tuk-tuk, xe ba gác máy) thuộc lớp 	hree_wheeler. | img_013.jpg | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box can_review)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk / xe ba gác chở hàng bị model nhận diện nhầm sang xe ô tô hoặc xe tải. | YOLO26m gán nhầm nhãn thành car hoặc 	ruck với độ tin cậy cao. | Xóa nhãn sai và gán lại đúng lớp 	hree_wheeler. | Mục 2.3 (Phân loại phương tiện 3 bánh có động cơ): Mọi xe 3 bánh gắn động cơ đều thuộc 	hree_wheeler, không gộp vào car/	ruck. |
| img_006.jpg | Khung bao quanh phương tiện bị rộng/lỏng do dính phần bóng đổ hoặc vật thể nền phía sau. | YOLOE-26s và bẫy loose tạo box dư lề > 15% kích thước vật thể. | Kéo gọn 4 cạnh box ôm sát mép ngoài cùng của phương tiện (kể cả phần hàng hóa cố định trên xe). | Mục 3.1 (Quy tắc vẽ Bounding Box): Box phải ôm sát biên thực tế của đối tượng, loại bỏ hoàn toàn bóng đổ mặt đường. |
| img_034.jpg | Người đi bộ / người điều khiển xe máy ở xa hoặc bị che khuất một phần trong đám đông. | YOLO26m bỏ sót đối tượng hoặc bẫy missing xóa mất khung nhãn. | Bổ sung box đầy đủ cho mọi đối tượng có chiều cao >= 20px và nhận diện rõ hình dáng. | Mục 2.1 (Ngưỡng kích thước tối thiểu & che khuất): Gán nhãn toàn bộ đối tượng bị che khuất <= 50% nếu xác định được loại phương tiện/người. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, case_id) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân biệt xe đạp điện (Bicycle vs motorbike), tách bạch xe xích lô (Bicycle) với xe 3 bánh động cơ (	hree_wheeler), bổ sung quy tắc ôm sát biên box và ngưỡng che khuất. | CAL-01, CAL-02, CAL-03 |
