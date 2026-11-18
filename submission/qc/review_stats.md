# Số đo review (det)

- Ảnh tính: 28 (batch,gold); box máy: 703; đối tượng sau duyệt: 532; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/703 = 63.4% | 703 box máy |
| edited (sửa) | 42 | 42/703 = 6.0% | 703 box máy |
| deleted (xoá) | 195 | 195/703 = 27.7% | 703 box máy |
| added (vẽ thêm) | 44 | 44/532 = 8.3% | 532 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 32.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 241 | 10 | 55 | 10 |
| bicycle | 4 | 3 | 11 | 8 |
| motorbike | 140 | 25 | 83 | 5 |
| car | 47 | 1 | 20 | 15 |
| bus | 1 | 0 | 2 | 1 |
| truck | 3 | 0 | 5 | 1 |
| three_wheeler | 10 | 3 | 19 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 61 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 49 | 26 | 12 | 65.3% (KTC 95%: 54–75%) | 80.3% (KTC 95%: 69–88%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*

| Sau duyệt của bạn (4 ảnh gold có trong nhãn cuối) | 0 | 0 | 61 | n/a | 0.0% |

- **Residual** (lỗi của máy còn trong nhãn của bạn): 12 (đếm FP + FN; một lỗi sai lớp hoặc lệch vị trí tính 2)
- **New** (lỗi do bạn tạo ra): 49

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 1/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_034.jpg (missing), img_006.jpg (loose), img_025.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
