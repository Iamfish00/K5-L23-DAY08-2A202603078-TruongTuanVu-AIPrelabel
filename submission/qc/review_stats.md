# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 234; đối tượng sau duyệt: 351; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 199 | 199/234 = 85.0% | 234 box máy |
| edited (sửa) | 15 | 15/234 = 6.4% | 234 box máy |
| deleted (xoá) | 20 | 20/234 = 8.5% | 234 box máy |
| added (vẽ thêm) | 137 | 137/351 = 39.0% | 351 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 131.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 98 | 8 | 6 | 61 |
| bicycle | 3 | 0 | 1 | 5 |
| motorbike | 55 | 6 | 5 | 37 |
| car | 25 | 0 | 1 | 29 |
| bus | 1 | 0 | 1 | 2 |
| truck | 0 | 0 | 2 | 1 |
| three_wheeler | 17 | 1 | 4 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 75 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 28 | 7 | 47 | 80.0% (KTC 95%: 64–90%) | 37.3% (KTC 95%: 27–49%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - extra: 3/3
  - wrong_class: 3/3
  - missing: 3/3
  - lọt: img_029.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
