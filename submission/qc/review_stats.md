# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 434; đối tượng sau duyệt: 449; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 377 | 377/434 = 86.9% | 434 box máy |
| edited (sửa) | 20 | 20/434 = 4.6% | 434 box máy |
| deleted (xoá) | 37 | 37/434 = 8.5% | 434 box máy |
| added (vẽ thêm) | 52 | 52/449 = 11.6% | 449 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 44.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 237 | 7 | 8 | 4 |
| bicycle | 5 | 1 | 3 | 2 |
| motorbike | 79 | 8 | 3 | 19 |
| car | 42 | 1 | 6 | 19 |
| bus | 2 | 0 | 1 | 0 |
| truck | 2 | 0 | 5 | 2 |
| three_wheeler | 10 | 3 | 11 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 23 | 9 | 62.9% (KTC 95%: 50–74%) | 81.2% (KTC 95%: 68–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 1/3
  - loose: 3/3
  - extra: 2/3
  - missing: 2/3
  - lọt: img_023.jpg (wrong_class), img_034.jpg (extra), img_013.jpg (missing), img_024.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
