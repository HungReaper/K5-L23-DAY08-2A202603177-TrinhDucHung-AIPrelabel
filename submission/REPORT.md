# Báo cáo Day 08 — 2A202603177 - Trịnh Đức Hưng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp                                             | Model + cỡ                                                                        | Ngưỡng                                                                                       | Vì sao (dẫn `run_id` trong `model_log.csv`)                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------ | ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`)                                                           | person 0.35 · motorbike 0.5 · car 0.35 · bicycle / bus / truck 0.3 (`--conf` mặc định) | r04 (conf 0.25) bắt đủ person nhưng nhiều box thừa / trùng (215 box trên 8 ảnh pilot); r05 (conf 0.5) bớt box thừa (140 box) nhưng sót người nhỏ. Trên gold: person 0.35 → 17/11/3 (F1 0.708), ít FP hơn hẳn 0.15 (18/21/2) mà chỉ mất 1 TP; motorbike 0.5 → 5/5/2 so với 5/12/2 ở 0.15 (không mất TP); car 0.35 ≈ 7/8/1. Không dùng 0.15 script gợi ý vì gần gấp đôi số box phải xoá khi duyệt |
| three_wheeler                                    | YOLOE-26s-seg (`yoloe-26s-seg.pt`), prompt: "tuk-tuk", "cyclo", "cargo tricycle" | 0.3 (`--conf` mặc định)                                                                   | r06: YOLOE bắt được xe ba bánh mà YOLO26 gọi bicycle / truck (img_002, img_032), nhưng cũng bắt nhầm xe ngoài mép (img_002, img_011). Gold ở 0.3: 5/1/2 so với 7/2/0 ở 0.15: chọn ít FP, chấp nhận vẽ thêm xe bị sót                                                                                                                                                                                               |

Số liệu tham chiếu (trích nguyên từ `qc/threshold_sweep.csv`, 4 ảnh gold; TP/FP/FN là số box):

| Lớp          | n_gold | Gợi ý của script    | thr 0.15 (TP/FP/FN) | thr 0.3 (TP/FP/FN) | thr 0.5 (TP/FP/FN) |
| ------------- | ------ | ---------------------- | ------------------- | ------------------ | ------------------ |
| person        | 20     | 0.15                   | 18/21/2             | 17/13/3            | 15/5/5             |
| bicycle       | 5      | 0.2                    | 5/1/0               | 4/1/1              | 2/0/3              |
| motorbike     | 7      | mặc định            | 5/12/2              | 5/7/2              | 5/5/2              |
| car           | 8      | mặc định            | 7/9/1               | 7/8/1              | 7/6/1              |
| bus           | 1      | mặc định (< 5 vật) | 1/0/0               | 1/0/0              | 1/0/0              |
| truck         | 0      | mặc định (< 5 vật) | 0/2/0               | 0/2/0              | 0/1/0              |
| three_wheeler | 7      | 0.15                   | 7/2/0               | 5/1/2              | 3/0/4              |

Chuỗi script gợi ý: `--thr "person=0.15,bicycle=0.2,three_wheeler=0.15"`

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (nguồn: r08 YOLO26m conf 0.05 và r09 YOLOE conf 0.05 trên 36 ảnh)
- **Phiên bản SAM bạn chọn**: **SAM 2.1 base** (`sam2.1_b.pt`). Chạy được trên RTX 3050 Ti 4 GB (r07: 1.7 s/ảnh), tải tự do không cần xin quyền như SAM 3; trên img_004 và img_017 mask ôm sát cả mui lẫn bánh chỉ với 3 click. Đã so với SAM 1 (MobileSAM) trên img_013: MobileSAM cho mask liền hơn nhưng lấn sang người lái (mục 5).
- Lỗi **không sửa được bằng đổi ngưỡng**: sai lớp. YOLO26 gọi xe ba bánh là bicycle / truck (r04, r05: img_032, img_002) và r07 gọi tuk-tuk ở img_004, img_013 là `car`; tăng ngưỡng chỉ làm mất box chứ không đổi lớp. Sửa: lấy lớp `three_wheeler` từ YOLOE (prompt riêng), gộp bằng `merge_prelabels.py --iou 0.6`; phần còn lại đổi lớp tay khi duyệt (`review_stats.md`: 8 trường hợp đổi lớp).
- Box **điểm cao mà sai** (so pre-label với `batch_reviewed.json`): img_024 `bus` 0.87 → thật là `car`; img_025 `motorbike` 0.84 → `bicycle`; img_003 `three_wheeler` 0.80 bị xoá; img_010 `three_wheeler` 0.71 thực ra trùng box một người. **Điểm thấp mà đúng**: img_017 `three_wheeler` 0.36 và nhiều `person` 0.35–0.36 (img_005, img_006, img_022, img_025, img_035) được giữ nguyên. → `conf` không đủ để tự chấp nhận (CAL-05).
- Giới hạn: ngưỡng chọn trên chính 4 ảnh gold (48 đối tượng; bus 1, truck 0, bicycle 5) nên precision / recall máy trên gold là in-sample, lạc quan; bus / truck không có căn cứ chọn ngưỡng, chỉ dùng mặc định 0.3. Cần một tập gold độc lập để đo lại.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ         | Mẫu số                     |
| ------------ | --- | --------------- | ---------------------------- |
| accepted     | 377 | 377/434 = 86.9% | 434 box máy                 |
| edited       | 20  | 20/434 = 4.6%   | 434 box máy                 |
| deleted      | 37  | 37/434 = 8.5%   | 434 box máy                 |
| added        | 52  | 52/449 = 11.6%  | 449 đối tượng sau duyệt |

(Đã loại 21 đối tượng quanh bẫy. 8 trường hợp trong deleted + added là đổi lớp, nên vật máy sót thật ≈ 44.)

- Lớp máy tệ nhất: **three_wheeler**, accepted chỉ 10/24 = 41.7% box máy (edited 3, deleted 11) và added 6. YOLOE bắt nhầm box quanh người / xe khác (img_003, img_010, img_014 bị xoá) và YOLO26 gọi tuk-tuk là bus / car (img_013). Kế đến là **truck** (accepted 2/7, deleted 5) và bỏ sót **motorbike / car** (added 19 mỗi lớp: xe đỗ bị che, img_006, img_035).
- So với gold: precision / recall của **máy** **39/62 = 62.9%** (KTC 95%: 50–74%) / **39/48 = 81.2%** (KTC 95%: 68–90%) (mẫu số: 48 đối tượng gold trên 4 ảnh; 62 box máy)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 = 66.7%** (KTC 95%: 39–86%, ngưỡng 80%). Lọt 4: `wrong_class` img_023 (car → three_wheeler, vật nhỏ sát mép trên) và img_024 (car → truck); `extra` img_034 (box car giả); `missing` img_013 (three_wheeler bị xoá). Theo loại: loose 3/3, extra 2/3, missing 2/3, wrong_class 1/3. Vì sao: tôi bỏ bước 0 của checklist (quét ảnh, đếm vật **trước** khi nhìn box máy) và không zoom kiểm lớp của box nhỏ ở mép; box nằm đúng chỗ thì tôi mặc định lớp đúng.
- Accepted 377/434 = 86.9% nhưng bắt bẫy chỉ 66.7%: tôi chủ yếu **xác nhận** box máy thay vì **kiểm** chúng (thiên kiến tự động hoá). Bẫy loose (lệch biên, thấy ngay) bắt đủ 3/3; bẫy phải đối chiếu nội dung ảnh (sai lớp, thiếu vật) thì lọt. QC Part 6 cho cùng kiểu: 5/6 lỗi là bỏ sót vật mà máy cũng sót.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **21/4 = 5.25 phút/ảnh** vs duyệt **44.4/24 = 1.85 phút/ảnh** (nhanh ≈ 2.8 lần). Thiên lệch: gold gán trước nên lúc duyệt tôi đã quen CVAT, taxonomy và kiểu cảnh phố, có lợi cho thời gian duyệt; gold chỉ 4 ảnh nên so sánh / ảnh chỉ là xấp xỉ. Duyệt nhanh cũng đi kèm bỏ sót (QC: 4/5 ảnh có lỗi).
- Kết luận: **có, cho person / motorbike / car** (accepted person 237/252, motorbike 79/90, car 42/49 box máy; duyệt nhanh ≈ 2.8 lần), **không tin cho three_wheeler / truck** (accepted 10/24 và 2/7: sửa / xoá gần như vẽ lại). Điều kiện: phải thêm bước quét vật sót, vì máy và người duyệt cùng bỏ sót xe bị che (QC: 5 bỏ sót / 91 đối tượng).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quét trước, nhìn box sau**: mỗi ảnh đếm số vật theo lớp trước khi bật nhãn máy, rồi so với số box. Kiểm bằng: vòng bẫy sau bắt ≥ 10/12.
  2. **Đếm hàng xe đỗ** (CAL-02): mỗi hàng / cụm xe đỗ đếm từ trước ra sau, thiếu box thì vẽ thêm. Kiểm bằng: QC vòng 2 có 0 lỗi `bo_sot` motorbike trên mẫu.
  3. **Zoom mọi box `truck` / `three_wheeler` và box nhỏ sát mép**, không chấp nhận theo `conf` (CAL-04, CAL-05). Kiểm bằng: 0 bẫy `wrong_class` lọt; thời gian duyệt không dưới ~1.5 phút/ảnh (nhanh hơn là dấu hiệu chỉ bấm chấp nhận).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh    | SAM (phiên bản)                   | Số click                              | Mask gồm người lái / hàng hoá?                                                        | Lỗi quan sát                                                                                                                              | File trong `masks/`      |
| ------- | ----------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| img_004 | SAM 2.1 base (`sam2.1_b.pt`)      | 3 (2 dương, 1 âm)                   | Không: không thấy người lái / hàng; người đứng cạnh (click âm) không bị lấn | Ôm sát mui, thân, bánh; lấn nhẹ một mảng bóng tối dưới gầm (~x650–730, y545–565)                                             | `tuktuk_img004_sam2.jpg` |
| img_013 | SAM 2.1 base (`sam2.1_b.pt`)      | 5 (2 dương, 3 âm vào người lái) | Không gồm người lái (đúng guideline), không có hàng                               | Mask **lốm đốm, rỗ** ở mui và thân, sót bánh xe và khung inox; phải thêm click âm nhiều lần mới đẩy người lái ra | `tuktuk_img013_sam2.jpg` |
| img_013 | SAM 1 MobileSAM (`mobile_sam.pt`) | 3 (2 dương, 1 âm)                   | **Lấn người lái**: mặt, chân, dép (sai: người là vật riêng)               | Liền hơn SAM 2.1 (gồm bánh, ghế, khung) nhưng lấn người; ghế / nội thất sau khung hở cũng bị tô                             | `tuktuk_img013_sam1.jpg` |
| img_017 | SAM 2.1 base (`sam2.1_b.pt`)      | 3 (2 dương, 1 âm)                   | Không có người / hàng; tuk-tuk xanh phía sau không bị lấn                          | Tốt nhất: ôm cả mui, kính chắn gió, hai bánh; mép mui hơi răng cưa                                                              | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: **giúp**: 4.2.3 (bóng đổ không thuộc vật → bỏ mảng tối ở img_004; kính xe thuộc vật → giữ kính ở img_017) và 4.1.4 + 4.2.4 (người là vật riêng → biết mask MobileSAM ở img_013 sai). **Thiếu**: 4.2.2 chỉ nói lỗ < 5% không vẽ, chưa nói **ghế / nội thất nhìn thấy qua khung hở** của tuk-tuk có thuộc mask không (img_013); hàng nhô ra đã chốt ở v1.1 (CAL-03).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử; chỉ dùng `scripts/sam2_points.py` offline.

## 6. Calibration (Part 6)

- Số quyết định: **5** (`calibration_decisions.csv`: 3 `cau_hoi_mo`, 1 `loi_may`, 1 `bay_lot`). Quan trọng nhất: **CAL-01** (xe máy đỗ dày gần camera → gán từng xe) cùng **CAL-02** (YOLO26m bỏ sót xe đỗ phía sau chiếc đầu hàng), vì đây chính là lỗi lặp chiếm 5/6 lỗi QC (img_003, img_006, img_035).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - CAL-01 (img_006, img_035): mục 4.1.6, xe gần camera còn thấy ≥ 20% và cao ≥ 20 px → một box `motorbike` riêng; `ignore_region` chỉ cho vật cao < 20 px hoặc không tách được từng xe.
  - CAL-03 (img_031): mục 4.1.5 + 4.2, hàng nhô đo theo chiều dài khung xe; ≤ 20% thì box / mask gồm hàng, > 20% thì không.
  - CAL-05 (img_023, img_024, img_034, img_013): không đặt mức `conf` tự chấp nhận; mọi box máy đều phải nhìn.
  - Checklist duyệt 3 bước (mục 3 v1.1): quét + đếm hàng xe đỗ (CAL-02), zoom mọi box `truck` / `three_wheeler` (CAL-04), không chấp nhận theo `conf` (CAL-05).
  - Chưa chốt câu mở 1, 4, 5, 6: không có ảnh ví dụ trong mẫu QC, giữ v1.0 + `can_review`.
- Nguyên nhân gốc:
  - **Guideline mơ hồ** (sửa chữ): CAL-01, CAL-03, CAL-05.
  - **Máy + tôi duyệt sai** (guideline đã đủ, chỉ thêm checklist): CAL-02 (máy sót xe bị che, tôi chấp nhận mà không quét), CAL-04 (lọt 2/3 bẫy đổi lớp dù mục 4.3 đã bảo đếm bánh).

## 7. QC và quyết định (Part 6)

- Mẫu: **5** ảnh (**0** ảnh `can_review` + **5** ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 91 đối tượng. Batch không có box nào tick `can_review` (0/461), nên không có ca khó trong mẫu. Người kiểm: tôi (**hạn chế**: tôi cũng là người duyệt lô này, dễ bỏ sót đúng chỗ đã bỏ sót).
- Lỗi theo **đối tượng**: **6/91 = 6.6%** · theo **ảnh**: **4/5 = 80.0%** (ngưỡng Release ≤ 2%)
- Top lỗi (`qc_review.csv`):
  - `bo_sot`: 5/6 lỗi, ví dụ img_006 (2 xe máy đỗ sau #75), img_003 (xe máy sau #16, xe đạp của #3), img_035 (xe tay ga sau #434).
  - `sai_vi_tri_kich_thuoc`: 1/6, img_031 (#395 bicycle thiếu khay quả nhô ~18%).
  - Không ghi nhận `thua`, `sai_lop`, `sai_thuoc_tinh`, `sai_mask` trong mẫu.
- **Giới hạn mẫu nhỏ**: 4 ảnh lỗi / 5 ảnh, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **96%**. Với 5 ảnh thì ngay cả 0/5 cũng không chứng minh được ngưỡng 2% (quy tắc 3: tới ~60%). Ở đây kết luận ngược lại thì chắc: tỉ lệ lỗi rõ ràng vượt ngưỡng. Bắt bẫy 8/12 = 66.7% (KTC 95%: 39–86%) cũng chỉ từ 12 bẫy.
- Script kiểm được / không kiểm được:
  - Kiểm được: định dạng COCO, lớp trong taxonomy, box trong ảnh, đếm accepted / edited / deleted / added, bắt bẫy, tỉ lệ lỗi từ phiếu.
  - Không kiểm được: box có ôm sát vật thật không (như #395 thiếu khay), vật mà **cả máy lẫn tôi cùng bỏ sót** (5 bỏ sót trong QC đều là loại này, không nằm trong `added`), và tôi có thật sự làm theo checklist không. QC bằng mắt cũng chỉ do một người làm.

**Quyết định: REWORK**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release):

- **Hold?** Không: các câu mở còn lại (1, 4, 5, 6) không xuất hiện trong mẫu và không làm đổi nhãn đã gán; dữ liệu không có ảnh hỏng (`qc_report.py` chạy với `--guideline-gap 0 --data-problem 0`).
- **Rework?** Có, cả ba điều kiện: ảnh lỗi 4/5 = 80% > 2%; lỗi hệ thống `bo_sot` 5/6 lỗi; bắt bẫy 8/12 = 66.7% < 80%.

Việc cần làm tiếp:

1. Sửa 4 ảnh lỗi trong CVAT (thêm 5 box bị sót, mở rộng #395), rồi duyệt lại **toàn bộ 24 ảnh batch** theo checklist v1.1 (quét + đếm hàng xe đỗ, zoom `truck` / `three_wheeler`), vì lỗi là hệ thống chứ không riêng mẫu.
2. Export các ảnh đã sửa → `submission/labels/qc_fixed.json`; chạy lại `build_release.py`, `qc_sample.py --force --seed <seed mới>`, `qc_report.py`; ghi vòng QC 2 vào báo cáo này.
3. Nếu vòng 2 còn nhiều `bo_sot` motorbike / car: hạ ngưỡng YOLO26 cho motorbike / car (Part 3) hoặc thêm lượt tìm vật sót trước khi duyệt.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào): Claude Code. (1) Trích bảng số liệu từ `threshold_sweep.csv` vào mục 1, đã đối chiếu lại với file CSV. Nháp REPORT mục 1–8 từ `model_log.csv`, `time_log.csv`, `review_stats.md`, lịch sử lệnh và ảnh mask. Tôi kiểm lại từng lỗi QC trên ảnh gốc và từng con số với file nguồn. AI không được dùng khi gán gold (Part 1) và khi duyệt 24 ảnh batch trong CVAT (Part 4).
- Gold có bị nhiễm không? **Không**: gán tay 4 ảnh gold trong CVAT (16:46–17:07, commit `0d2e19d` lúc 17:13) **trước** khi chạy bất kỳ model nào (Part 2, commit `89757be` lúc 18:10), không hỏi AI.
- Máy / GPU / CVAT: Windows 11 Home, NVIDIA RTX 3050 Ti Laptop 4 GB (CUDA, torch 2.15.0+cu132), CVAT v2.81.0 trên Docker. Vướng mắc cài đặt: không đáng kể; GPU 4 GB đủ cho YOLO26m, YOLOE-26s và SAM 2.1 base (r07: 1.7 s/ảnh).
- Dùng cache `data/cache/` thay vì tự chạy? **Không**: tự chạy YOLO26m (r08) và YOLOE (r09) trên 36 ảnh bằng CUDA.
- Lỗi / bug script đã gặp: không gặp bug script. Chỉ lưu ý: YOLO26 trả nhiều lớp COCO ngoài taxonomy (umbrella, handbag, bottle, banana…) bị script loại (cột `dropped` trong `work/runs.csv`), không ảnh hưởng nhãn.
- Một điều làm khác lần sau: **quét và đếm vật trước khi bật pre-label**, và tick `can_review` cho ca không chắc ngay khi duyệt (lô này 0/461 box `can_review`, nên mẫu QC không có ca khó nào và calibration phải dựa vào lỗi tìm thấy sau).
