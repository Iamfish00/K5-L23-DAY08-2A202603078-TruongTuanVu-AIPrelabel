# Báo cáo Day 08 — 2A202603078 - Trương Tuấn Vũ

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n, CPU, imgsz=640 | 0,30 (mặc định) | Dùng cho 6 lớp COCO của bài. Trên 8 ảnh pilot, ngưỡng 0,25 giữ 120 box, còn ngưỡng 0,50 giữ 54 box. Ở ngưỡng 0,50 mất box người đạp xe trong img_019.jpg (r05, r06). |
| three_wheeler | YOLOE-26s với tuk-tuk, cyclo, cargo tricycle | 0,30 (mặc định) | Tìm được tuk-tuk xanh trong img_002.jpg (0,97) và hai xích lô trong img_032.jpg. Các xích lô đen bên phải vẫn cần kiểm (r07). |

- **Kết quả CP3.** Quét trên 4 ảnh gold với mục tiêu recall 90%. Recall cao nhất trong bảng quét lần lượt là person 16/31 (52%), bicycle 0/7 (0%), motorbike 8/20 (40%), car 7/9 (78%) và three_wheeler 6/7 (86%). Bus có 1 mẫu và truck có 0 mẫu, đều dưới mức tối thiểu 5 mẫu để đề xuất ngưỡng riêng. Vì vậy dùng `--conf 0.3` cho CP4 và bỏ tùy chọn `--thr`. [Bảng quét ngưỡng](qc/threshold_sweep.md)
- **Lệnh gộp ở CP4**

```powershell
python -B scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split 'batch,gold' --out work/prelabel/merged_review/pre.coco.json
```

- Kết quả gộp có 280 box trên 28 ảnh gồm 24 ảnh batch và 4 ảnh gold. Task batch-review đã nhận 245 box trên đủ 24 ảnh batch từ `pre_traps_import.coco.json`.

- **Phiên bản SAM:** MobileSAM (SAM 1) chạy CPU, dùng box YOLO26n làm đầu vào. Lượt r08 sinh mask cho 10/10 box trên hai ảnh, trung bình 5,89 giây/ảnh cho cả phát hiện và phân đoạn. Mask người mặc áo tím trong img_004.jpg theo phần thân và chân nhìn thấy.
- Trong img_032.jpg, YOLO26n gọi xích lô đen bên phải là bicycle (0,38, r05). Đổi ngưỡng chỉ giữ hoặc bỏ box. Để sửa lớp cần dùng YOLOE tìm three_wheeler và đối chiếu ảnh khi duyệt. Trong img_013.jpg, MobileSAM thiếu mask tuk-tuk tiền cảnh vì đầu vào thiếu box xe (r08).
- **Điểm thấp mà đúng:** box person 0,28 cho người đạp xe giữa img_019.jpg có ở r05 và biến mất khi tăng ngưỡng lên 0,50 ở r06.
- **Giới hạn.** Cùng 4 ảnh gold được dùng để thử ngưỡng và đo recall. Người học xác nhận đã tham khảo gợi ý trước khi hoàn tất nhãn gold, nên tập này có nguy cơ bị ảnh hưởng bởi các gợi ý. Chỉ số trên gold chỉ mô tả mức khớp với các nhãn hiện có. Các lớp ít mẫu chưa đủ căn cứ để chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

- Bản xuất CVAT vòng 1 có 347 box trên 24 ảnh. Vòng 2 có 363 box trên 24 ảnh sau khi thêm 20 box, sửa 6 box (trong đó có 4 ca đổi lớp) và xóa 4 box trên 9 ảnh. `validate_labels.py` trả về OK. Batch còn 9 cảnh báo box dưới 20 px ở `img_018.jpg`, `img_024.jpg` và `img_027.jpg`. Gold còn 2 cảnh báo box người dưới 20 px ở `img_036.jpg`.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 199 | 85,0% | 234 box máy trong phạm vi tính |
| edited | 15 | 6,4% | 234 box máy trong phạm vi tính |
| deleted | 20 | 8,5% | 234 box máy trong phạm vi tính |
| added | 137 | 39,0% | 351 đối tượng sau duyệt trong phạm vi tính |

- Bảng so sánh loại 23 box trước và sau duyệt nằm quanh bẫy. Script đếm 6 trường hợp đổi lớp trong nhóm xóa/thêm.
- Dữ kiện để nhận xét theo lớp. `person` có số box thêm cao nhất, 61 trong 167 box người sau duyệt trong phạm vi tính. `truck` có 0/2 box máy được giữ, với mẫu rất nhỏ. Ví dụ đã sửa ở vòng 2 gồm xe đạp tựa tường bị gán `motorbike` trong `img_009.jpg`, SUV bị gán `three_wheeler` trong `img_033.jpg` và hai box `person` bao quần áo treo trong `img_031.jpg`. Trong mẫu QC, cả 4 ca bỏ sót đều là người, ví dụ hai trẻ ngồi cạnh bàn ở `img_006.jpg` và người lái xích lô ở `img_035.jpg`. Đây là nhóm cần ưu tiên rà ở lượt sau.
- So với gold trên 4 ảnh, máy có 28 TP, 7 FP và 47 FN. Precision là 28/35 = 80,0%, recall là 28/75 = 37,3%. Phép ghép dùng IoU ≥ 0,50 và cùng lớp. Bốn ảnh này cũng đã dùng để thử ngưỡng ở CP3. [Kết quả script](qc/review_stats.md)

## 3. Thiên kiến tự động hóa và ảnh bẫy

| Lượt duyệt | Bẫy bắt được | Box lỏng | Box dư | Sai lớp | Bỏ sót |
|---|---:|---:|---:|---:|---:|
| Bản xuất đầu tiên | 5/12 (41,7%) | 2/3 | 2/3 | 0/3 | 1/3 |
| Rà lại có hỗ trợ kiểm nhãn | 11/12 (91,7%) | 2/3 | 3/3 | 3/3 | 3/3 |

- Vòng 2 diễn ra sau khi đã có kết quả vòng đầu. Số 11/12 vượt mốc 10/12 của CP4, còn 1 ca box lỏng ở `img_029.jpg`. Khoảng tin cậy 95% của vòng 2 là 65–99%. [Kết quả script](qc/review_stats.md#3-ảnh-bẫy)
- Vòng đầu có 205/234 box máy được giữ (87,6%) và bắt được 5/12 bẫy (41,7%). `Accepted` đếm các box máy được giữ trong phạm vi tính, còn điểm bẫy đếm các lỗi được phát hiện. Vòng đầu giữ nhiều box và vẫn lọt cả 3 bẫy sai lớp. Các lỗi sai lớp lọt ở vòng 1 cho thấy cần kiểm lại lớp của từng box. Phiếu vòng 1 không lưu chi tiết bước nào đã bị bỏ qua.

## 4. Máy có đáng dùng không?

- Ngày 17/11/2026, Gold làm từ 17:00 đến 17:30 và Batch từ 18:30 đến 19:00 theo mốc người học cung cấp sau buổi. Từ `time_log.csv`, Gold là 30 phút / 4 ảnh = 7,50 phút/ảnh và Batch là 30 phút / 24 ảnh = 1,25 phút/ảnh. Đây là thời lượng ước tính. Hai lượt dùng các ảnh khác nhau và Gold làm trước Batch, nên số liệu chỉ mô tả hai lượt của bài.
- Căn cứ để cân nhắc dùng pre-label: giữ 199/234 box máy ở vòng 2, nhưng trên gold máy bỏ sót 47/75 vật. Mẫu QC cũng còn 4 ca bỏ sót người. Pre-label hữu ích làm điểm xuất phát cho xe dễ thấy và dùng YOLOE để tìm thêm ứng viên xe ba bánh. Với người nhỏ, bị che hoặc ở hậu cảnh, cần có lượt tìm vật sót riêng.
- **Ba bước đề xuất để giảm thiên kiến khi duyệt**
  1. Xem ảnh gốc trước khi nhìn box gợi ý, sau đó quét lại lần lượt từ trái sang phải để tìm vật bị sót.
  2. Kiểm tra riêng nhãn người và xe ở từng ca có người lái nhìn thấy.
  3. So từng cạnh box với phần vật nhìn thấy, nhất là mép ảnh và vùng bị che.

## 5. Mask SAM cho 3 vật (Part 5)

Cả ba lượt chạy MobileSAM (SAM 1, `mobile_sam.pt`) trên CPU bằng `sam2_points.py`. Số điểm dương (+) chỉ vùng cần lấy, điểm âm (−) chỉ vùng cần loại.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_013.jpg` · tuk-tuk | MobileSAM (SAM 1) | 3 (2+, 1−) | Dính một phần mặt và chân người lái. | Mái, thân và bánh xe đã được chọn. Biên quanh người lái và sàn xe còn lẫn. Kết quả 245.541 px (35,2% ảnh), 2 polygon. | [tuktuk_img013.jpg](masks/tuktuk_img013.jpg) |
| `img_031.jpg` · người bán trái cây | MobileSAM (SAM 1) | 4 (2+, 2−) | Chọn người bán. Vùng trái cây và giỏ chính nằm ngoài mask. | Mask bám nón và thân áo, hụt phần bàn tay nhìn thấy sát giỏ. Kết quả 23.880 px (3,4% ảnh), 1 polygon. | [person_img031.jpg](masks/person_img031.jpg) |
| `img_035.jpg` · xích lô | MobileSAM (SAM 1) | 4 (3+, 1−) | Dính phần thân người lái dưới mái. | Nhận được mái, ghế và bánh xe, còn lẫn người lái vào vùng ngay trên ghế. Kết quả 4.771 px (0,7% ảnh), 1 polygon. | [xichlo_img035.jpg](masks/xichlo_img035.jpg) |

- Đối chiếu mục 4.2 của guideline, ở `img_031.jpg` có thể kiểm biên theo phần người nhìn thấy cạnh mép giỏ. Ở `img_013.jpg` và `img_035.jpg`, vùng người lái nằm sát khung xe nên cần rà riêng ranh giới đó. Quy tắc bám biên theo phần nhìn thấy giúp rà mask người cạnh giỏ ở `img_031.jpg`. Ca người lái sát khung xe được ghi vào CAL-02 để làm rõ cách tách phần người khỏi mask xe.

## 6. Calibration (Part 6)

- Có 3 ca đề xuất trong [calibration_decisions.csv](qc/calibration_decisions.csv). CAL-01 dùng `img_031.jpg` để tách giỏ cố định và hàng rời. CAL-02 dùng `img_013.jpg` và `img_035.jpg` để kiểm phần người lái bị che. CAL-03 dùng lỗi YOLO26n gán xích lô thành `bicycle` trong `img_032.jpg` (r05).
- Các quy tắc đề xuất đã được đưa vào [guideline v1.1](guidelines/guideline_v1.1_calibrated.md), kèm ảnh và mã ca. CAL-02 liên quan đến cách ước lượng mức nhìn thấy và việc SAM dính người lái. CAL-03 có bằng chứng sai lớp từ lượt chạy gốc.
- Ưu tiên CAL-02 trong lượt rà tiếp theo vì QC vẫn phát hiện thiếu người lái xích lô ở `img_035.jpg`, còn mask ở ảnh này lấn vào phần người lái. Ba quy tắc v1.1 được ghi để áp dụng khi xử lý các ca tương tự.

## 7. QC và quyết định (Part 6)

- Mẫu gồm 5/24 ảnh batch, 0 ảnh được chọn theo `can_review` và 5 ảnh ngẫu nhiên với seed 42. Tổng cộng có 60 đối tượng đã gán trong mẫu. Đã đối chiếu ảnh gốc với overlay có ID. Các ca phát hiện và người rà được ghi trong [qc_review.csv](qc/qc_review.csv).
- Theo cách đếm của script, có **6/60 = 10,0% lỗi theo đối tượng** và **5/5 = 100,0% ảnh có lỗi**. Chỉ số lỗi theo đối tượng dùng số đối tượng đã gán làm mẫu số và gồm cả các vật bỏ sót trong tử số.
- **Bỏ sót 4/6 lỗi.** Ví dụ hai trẻ ngồi cạnh bàn trong `img_006.jpg` và người lái xích lô đỏ trong `img_035.jpg`. **Sai vị trí/kích thước 2/6 lỗi.** Ví dụ box người bán trái cây trong `img_031.jpg` cắt mất phần cẳng tay.
- Cận trên Wilson 95% là 100% với 5 ảnh lỗi trong mẫu 5 ảnh. Các con số mô tả mẫu đã rà. Mẫu 5 ảnh quá nhỏ để ước lượng sát tỉ lệ lỗi của cả lô hoặc xác nhận ngưỡng 2%.
- Script kiểm cấu trúc, lớp hợp lệ, hình học và tổng hợp lỗi trên phiếu QC. Phần box ôm sát vật, lớp có đúng với ảnh và vật bị bỏ sót cần so trực tiếp với ảnh gốc.

**Gợi ý của script ngày 2026-11-18: Rework.** Có lỗi ở 5/5 ảnh, cao hơn ngưỡng 2%. Bỏ sót chiếm 4/6 lỗi. [Kết quả QC](qc/qc_report_auto.md)

**Quyết định: Rework.** Lượt QC phát hiện lỗi ở 5/5 ảnh, vượt ngưỡng 2%. Ưu tiên rà nhóm người bị che hoặc lẫn vào đám đông, sửa nhãn và kiểm một mẫu mới.

## 8. Ghi chú cá nhân

- **Mô hình đã dùng.** YOLO26n tạo box cho 6 lớp COCO trong bài, YOLOE-26s bổ sung lớp `three_wheeler`, MobileSAM (SAM 1) tạo mask theo điểm hoặc box. Trợ lý AI hỗ trợ rà nhãn và tổng hợp kết quả. Các box đã lưu được đối chiếu qua API CVAT, số liệu được kiểm bằng script của bài.
- **Gold có nhiễm theo quy định bài.** Người học xác nhận đã xem pre-label hoặc hỏi AI trước khi hoàn thành 4 ảnh gold. Chưa ghi cụ thể ảnh nào và hình thức tham khảo.
- Hệ điều hành Windows, đồ họa Intel UHD, CVAT Docker 2.81.0. Các lượt mô hình ghi trong `model_log.csv` chạy CPU.
- Dữ liệu đầu vào của CP4 lấy từ các lượt chạy CPU r03 và r04 trong `work/prelabel/`. Các lượt thử r05–r08 có lệnh và kết quả trong `model_log.csv`.
- Khi kiểm lại sau buổi làm bài, CVAT trả HTTP 502 trong lúc Docker khởi động. API hoạt động sau khi CVAT lên xong và đọc lại được hai job.
- Đề xuất cho lần sau: rà riêng người đứng ở hậu cảnh và người lái bị xe che, vì bỏ sót người chiếm 4/6 lỗi trong mẫu QC.
