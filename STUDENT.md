# Thông tin học viên

- Họ và tên: Trịnh Đức Hưng
- MSHV: 2A202603177
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: https://github.com/HungReaper
- Repo bài nộp (URL): https://github.com/HungReaper/K5-L23-DAY08-2A202603177-TrinhDucHung-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows 11 Home, GPU NVIDIA GeForce RTX 3050 Ti Laptop 4 GB (CUDA, torch 2.15.0+cu132)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 377/434 = 86.9% / 20/434 = 4.6% / 37/434 = 8.5% (mẫu số: 434 box máy); added: 52/449 = 11.6% (mẫu số: 449 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 39/62 = 62.9% / 39/48 = 81.2% (4 ảnh gold, 48 đối tượng)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 21/4 = 5.25 phút/ảnh; duyệt có pre-label 44.4/24 = 1.85 phút/ảnh
- Lỗi QC: 80.0% theo ảnh (4/5), 6.6% theo đối tượng (6/91)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 4/5 > 2%, lỗi hệ thống bỏ sót (5/6 lỗi), bắt bẫy 8/12 < 80%
