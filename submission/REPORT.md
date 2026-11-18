# Báo cáo Day 08 — 2A202603088 - Ngọ Đức Trinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mỗi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.35, bicycle=0.30, motorbike=0.35, car=0.30, bus=0.85, truck=0.40 | Cân bằng Precision và Recall trên tập gold sau khi quét `threshold_sweep.py` (`run_y26_all`) |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | three_wheeler=0.35 | Model COCO không có lớp `three_wheeler` nên dùng open-vocabulary YOLOE-26s để bắt tuk-tuk và xe ba gác (`run_yoloe_all`) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,bicycle=0.3,motorbike=0.35,car=0.3,bus=0.85,truck=0.4,three_wheeler=0.35" --conf 0.3 --iou 0.5 --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 (`sam2.1_b.pt`)** chạy trên Google Colab GPU T4 vì chất lượng biên mask sắc nét, tốc độ xử lý nhanh và không bị giới hạn quyền truy cập gated như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhầm ngữ nghĩa lớp (semantic misclassification) như YOLO26m gán tuk-tuk/ba gác thành `car` hoặc `truck` với độ tin cậy cao không thể sửa bằng chỉnh ngưỡng. Phải dùng mô hình open-vocabulary (YOLOE-26s) cho lớp `three_wheeler` kết hợp duyệt thủ công trên CVAT để đổi lớp.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Ở `img_013.jpg`, YOLO26m dự đoán chiếc tuk-tuk là `car` với điểm tin cậy cao (>0.75) nhưng sai lớp; ở `img_034.jpg`, người đi bộ ở xa có điểm tin cậy thấp (~0.32) nhưng là đối tượng đúng.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Tập gold chỉ gồm 4 ảnh (61 đối tượng) nên bộ ngưỡng tối ưu trên tập này có xu hướng khớp quá mức (overfit/in-sample), dẫn đến Precision/Recall thực tế trên 24 ảnh batch thấp hơn một chút so với ước tính ban đầu.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 446/703 = 63.4% | 703 box máy |
| edited | 42 | 42/703 = 6.0% | 703 box máy |
| deleted | 195 | 195/703 = 27.7% | 703 box máy |
| added | 44 | 44/532 = 8.3% | 532 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **`three_wheeler`** và **`bicycle`** là hai lớp máy làm tệ nhất (`three_wheeler`: 19 deleted, 3 edited so với 10 accepted; `bicycle`: 11 deleted, 8 added so với 4 accepted). Nguyên nhân do đặc thù giao thông châu Á có xe tuk-tuk, ba gác máy (`img_013.jpg`) và xe xích lô (`img_006.jpg`) dễ bị nhầm sang `car` hoặc `motorbike`.
- So với gold: precision / recall của **máy** **65.3% (KTC 95%: 54–75%) / 80.3% (KTC 95%: 69–88%)** (mẫu số: **61 đối tượng gold**, TP=49, FP=26, FN=12).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 (75.0%, KTC 95%: 47–91%)**; loại nào lọt: **1 `missing` (`img_034.jpg`) và 2 `loose` (`img_006.jpg`, `img_025.jpg`)**; vì sao (bạn đã bỏ bước nào của checklist)? Do bị ảnh hưởng bởi thiên kiến tự động hóa ở các khung hình đông phương tiện, đôi khi bỏ qua bước quét toàn ảnh trước khi bật lớp phủ nhãn máy nên lọt 1 vật thể nhỏ bị xóa và 2 khung hơi rộng viền.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ `accepted` đạt 63.4% và bắt được 100% bẫy `wrong_class` (3/3) và `extra` (3/3) cho thấy việc kiểm tra sai lớp và box ma rất kỹ, nhưng tỉ lệ bắt bẫy `loose` (1/3) thấp cho thấy mắt dễ chấp nhận các khung có vị trí tương đối đúng mà chưa soi kỹ độ ôm sát đường biên.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **5.0 phút/ảnh (20 phút / 4 ảnh)** vs duyệt **1.88 phút/ảnh (45 phút / 24 ảnh)**. Thiên lệch (bạn gán gold trước nên đã quen cảnh): Việc gán tay 4 ảnh gold trước giúp làm quen bối cảnh và taxonomy, nhưng tập `batch` có mật độ vật thể dày hơn nên tốc độ duyệt thực tế vẫn nhanh hơn gán tay khoảng 2.6 lần.
- Kết luận: **Có, đáng dùng làm bước khởi tạo (đặc biệt hiệu quả với lớp `person`, `motorbike`, `car`)**, dựa trên **tỉ lệ `accepted` đạt 63.4% (446/703 box) và thời gian hoàn thành giảm từ 5.0 phút/ảnh xuống 1.88 phút/ảnh**.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Tắt lớp phủ pre-label (`Hide annotations`) để quét toàn bộ ảnh từ trái sang phải đếm vật thể trước khi bật nhãn máy kiểm tra.
  2. Phóng to (zoom >= 150%) vào tất cả các cụm phương tiện 3 bánh và xe đạp để kiểm tra độ ôm sát của 4 cạnh bounding box.
  3. Đối chiếu mọi ca phân vân với bảng quy tắc `guideline_v1.1_calibrated.md` trước khi bấm Accept.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không bao gồm người ngồi sau; tách riêng thân xe tuk-tuk | Hơi lem nhẹ ở phần bánh sau bên phải | `tuktuk_img013.jpg` |
| `img_001.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 (1 pos) | Không có hàng hóa ngoài xe; ôm trọn thân ô tô | Đường biên mượt, tách tốt khỏi nền đường | `car_img001.jpg` |
| `img_036.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 (1 pos) | Bao gồm áo mưa người đi xe máy | Tách chính xác người ngồi trên xe khỏi thân xe | `person_img036.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc tách biệt người điều khiển khỏi phương tiện (mỗi thực thể một instance mask riêng) giúp đặt điểm click dương/âm chính xác; cần bổ sung quy tắc xử lý phần chi tiết mảnh như gương chiếu hậu hoặc nan hoa bánh xe khi tạo mask.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Sử dụng công cụ offline `sam2_points.py` trên Google Colab GPU T4 để tránh quá tải RAM trên máy cá nhân cấu hình yếu; ưu điểm của chạy offline là lưu lại được tọa độ click tái lập được kết quả, trong khi Interactor trong CVAT phản hồi trực quan tức thì hơn.

## 6. Calibration (Part 6)

- Số quyết định: **4** (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): **`CAL-01`** (chuẩn hóa việc tách lớp `three_wheeler` cho xe tuk-tuk/ba gác máy bằng YOLOE-26s và duyệt lại toàn bộ xe 3 bánh trên batch).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Chốt phân loại xe xích lô đạp vào `bicycle`, xe ba gác/tuk-tuk có động cơ vào `three_wheeler`; quy tắc cắt bỏ bóng đổ khỏi bounding box (`CAL-03`) và ngưỡng che khuất >= 30% (`CAL-04`).
- Nguyên nhân gốc: **Do khoảng trống giữa bộ nhãn COCO mặc định của máy (thiếu lớp đặc thù giao thông châu Á) và các trường hợp biên trong guideline v1.0**.

## 7. QC và quyết định (Part 6)

- Mẫu: **5** ảnh (**0** ảnh `can_review` + **5** ngẫu nhiên, seed 42: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: **0/104 = 0.0%** · theo **ảnh**: **0/5 = 0.0%** (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Trên 5 ảnh mẫu QC ngẫu nhiên không ghi nhận lỗi (`ok` toàn bộ 104 đối tượng sau khi đã rà soát kỹ); trong quá trình review trước đó, 3 lỗi phổ biến nhất là sai lớp (`three_wheeler` bị gọi `car` ở `img_013.jpg`), box lỏng (`img_006.jpg`) và bỏ sót người ở xa (`img_034.jpg`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trên mẫu nhỏ 5 ảnh, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới **43%**, do đó mẫu nhỏ này chưa đủ lực thống kê để chứng minh tuyệt đối ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script kiểm tự động được cấu trúc COCO, ID hợp lệ, tỉ lệ bắt bẫy từ `traps_key.json` và tổng hợp số liệu từ phiếu `qc_review.csv`, nhưng không tự kiểm tra được độ ôm sát thực tế của box hay các vật thể mà cả máy và người cùng bỏ sót.

**Quyết định: Rework**, ngày **2026-11-18**. Lý do (đi theo Hold → Rework → Release): Dù mẫu QC 5 ảnh đạt 0.0% lỗi, tỉ lệ bắt bẫy trên tập batch đạt **75.0% (9/12)**, thấp hơn ngưỡng tin cậy **80%** của quy trình (lọt 2 bẫy `loose` và 1 bẫy `missing`).
Nếu Rework / Hold, việc cần làm tiếp: Rà soát lại toàn bộ 24 ảnh tập `batch` tập trung vào độ ôm sát của bounding box (`loose`) và các đối tượng nhỏ ở hậu cảnh (`img_006.jpg`, `img_025.jpg`, `img_034.jpg`), sau đó cập nhật lại `batch_reviewed.json` và chạy lại `review_stats.py` để đạt tỉ lệ bắt bẫy ≥ 80% trước khi chuyển sang Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng YOLO26m, YOLOE-26s và SAM 2.1 để chạy pre-label và tạo mask theo đúng yêu cầu bài Lab; dùng trợ lý AI hỗ trợ gỡ lỗi môi trường Python/Colab và đối chiếu cú pháp các script kiểm tra, mọi file JSON/CSV đầu ra đều được xác thực lại bằng `validate_labels.py` và `check_submission.py`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Không** — 4 ảnh gold được gán tay hoàn toàn thủ công trên CVAT ở Part 1 trước khi chạy bất kỳ mô hình pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Máy cá nhân Windows 11 (CPU, RAM hạn chế) chạy CVAT qua Docker Desktop và các script phân tích nhẹ; toàn bộ bước chạy mô hình nặng (YOLO26, YOLOE, SAM 2.1) được thực hiện trên Google Colab GPU T4 để tránh treo máy.
- Dùng cache `data/cache/` thay vì tự chạy? **Không** — tự chạy đầy đủ các mô hình trên Google Colab và tải kết quả về máy để quét ngưỡng và gộp nhãn.
- Lỗi / bug script đã gặp (nếu có): Gặp lỗi thiếu thư viện `ultralytics` khi chạy `sam2_points.py` trực tiếp trên máy local và lỗi `FileNotFoundError` khi Colab tự ngắt kết nối làm reset thư mục làm việc; đã khắc phục bằng cách chạy lại bước setup/tải trọng số trên Colab.
- Một điều bạn làm khác lần sau: Luôn tắt hẳn lớp phủ nhãn máy (`Hide annotations`) để quét ảnh và đếm vật thể độc lập trước khi bật pre-label lên duyệt, tránh thiên kiến tự động hóa làm lọt các box hơi lỏng viền.