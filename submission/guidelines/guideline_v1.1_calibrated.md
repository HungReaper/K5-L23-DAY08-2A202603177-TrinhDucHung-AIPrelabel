# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603177 - Trịnh Đức Hưng

> Chỉ ghi phần thay đổi so với v1.0. Mỗi quyết định khớp một dòng trong `qc/calibration_decisions.csv` và có ảnh ví dụ.
> Phân loại nguyên nhân: **guideline mơ hồ** (CAL-01, CAL-03, CAL-05 → sửa chữ trong guideline) và **máy / tôi làm sai** (CAL-02, CAL-04 → không đổi quy tắc nhãn, chỉ thêm bước vào checklist duyệt ở mục 3).

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Mỗi xe còn thấy **≥ 20%** và box cao **≥ 20 px** → **một box `motorbike` riêng**. Chỉ dùng `ignore_region` khi không tách được từng xe (không thấy bánh / yên / biển số riêng của từng chiếc). Sửa mục **4.1.6**: "cụm ≥ 5 vật → `ignore_region`" chỉ áp dụng cho vật **cao < 20 px**. | img_006 (hàng xe sau #70/#75), img_035 (xe trắng sau #434) | CAL-01 |
| 3 | Hàng nhô ra: box / mask xe có gồm hàng không? | Đo phần nhô theo **chiều dài khung xe** (từ mép bánh trước đến mép bánh sau). Nhô **≤ 20%** → box **và** mask xe **gồm** hàng; nhô **> 20%** → box / mask chỉ ôm xe, hàng không gán. Giỏ / thùng **gắn cố định** luôn thuộc xe (giữ như mục 5 v1.0). Sửa mục **4.1.5** và thêm vào **4.2**. | img_031 (khay quả trên baga nhô ~107 / 578 px ≈ 18% → gồm khay) | CAL-03 |
| 7 | Mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | **Không đặt.** `conf` chỉ dùng để lọc pre-label (ngưỡng theo lớp ở Part 3); mọi box máy vẫn phải được nhìn. Lý do: accepted 377/434 = 86.9% nhưng bắt bẫy chỉ 8/12 = 66.7%. | img_023, img_024, img_034, img_013 (4 bẫy lọt) | CAL-05 |

**Chưa chốt** (không gặp ca rõ ràng trong 5 ảnh mẫu QC nên chưa có ảnh ví dụ để làm căn cứ; giữ nguyên v1.0, gặp thì tick `can_review`):
câu 1 (xe đạp điện), câu 4 (người trong xe ba bánh có mui, ngưỡng 20%), câu 5 (kính xe trong mask), câu 6 (box máy chồng nhau ảnh đêm).

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

Batch không có box nào tick `can_review` (0/461 đối tượng trong `release/final_det.coco.json`), nên mẫu QC chỉ gồm 5 ảnh ngẫu nhiên.

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_006, img_035, img_003 | Xe máy / xe đạp đỗ hoặc bị che sau chiếc đầu hàng, sau người | YOLO26m chỉ ra box cho xe đầu hàng, bỏ sót xe phía sau (motorbike added = 19, cao nhất cùng car); tôi chấp nhận mà không quét → QC thấy 5 bỏ sót trên 3/5 ảnh | Gán từng xe theo CAL-01; sửa các ảnh lỗi vào `labels/qc_fixed.json` | Không đổi quy tắc nhãn; thêm bước 1 checklist (mục 3) — CAL-02 |
| img_023, img_024 | Bẫy `wrong_class`: car → three_wheeler (vật cắt mép trên ảnh), car → truck | (bẫy nhân tạo) — tôi để lọt 2/3 bẫy đổi lớp | Trả về `car` | Mục 4.3 v1.0 đã đủ (đếm bánh). Thêm bước 2 checklist (mục 3) — CAL-04 |
| img_031 | Xe đạp chở khay quả buộc trên baga, nhô sau xe ~18% | YOLO26m box xe đạp (conf 0.45) không gồm khay; tôi chấp nhận | Mở rộng box #395 sang trái đến hết khay | Mục 4.1.5 + 4.2 — CAL-03 |

## 3. Checklist duyệt bổ sung (không đổi nhãn, đổi cách làm)

1. **Quét hàng xe đỗ** (CAL-02): với mỗi hàng / cụm xe, đếm từ trước ra sau số xe thấy ≥ 20%, so với số box. Thiếu → vẽ thêm trước khi chuyển ảnh.
2. **Zoom mọi box `truck` / `three_wheeler`** (CAL-04): kiểm số bánh + hình dáng trước khi chấp nhận; vật cắt mép không đếm được bánh → `can_review`.
3. **Không chấp nhận theo `conf`** (CAL-05): box conf cao vẫn phải nhìn biên và lớp.

## 4. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2, 3, 7; sửa 4.1.5 (đo hàng nhô theo khung xe, ≤ 20% gồm hàng), 4.1.6 (`ignore_region` chỉ cho vật < 20 px), 4.2 (mask theo cùng quy tắc hàng nhô); thêm checklist duyệt 3 bước | CAL-01 (img_006, img_035), CAL-03 (img_031), CAL-05 (bẫy lọt); CAL-02 (img_003, img_006, img_035) và CAL-04 (img_023, img_024) là lỗi máy / người duyệt → chỉ vào checklist |
