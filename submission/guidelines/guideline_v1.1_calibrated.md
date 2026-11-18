# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603078 - Trương Tuấn Vũ

Ba quy tắc dưới đây được rút ra từ ảnh đã rà và dùng cho lượt rà tiếp theo.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Tính giỏ và hàng nhô ra thế nào? | Tính giỏ gắn cố định vào xe. Box dừng ở phần xe và bỏ phần hàng nhô quá khoảng 20% chiều dài xe theo mục 4.1.5. Trong mask xe, tô khung và phần giỏ cố định nhìn thấy, loại vùng trái cây rời. | `img_031.jpg` | CAL-01 |
| 4 | Gán người ngồi trong xe mui che ra sao? | Giữ ngưỡng khoảng 20% của v1.0. Dùng phần đầu và thân nhìn thấy để ước lượng, vẽ box riêng quanh phần người thấy được nếu đạt ngưỡng. Ca sát ngưỡng đánh dấu `can_review`. Loại phần người lái ra khỏi mask xe. | `img_013.jpg`, `img_035.jpg` | CAL-02 |

Câu 1 và 5 cần ví dụ rõ về động cơ xe điện hoặc kính xe. Câu 2 và 6 cần rà riêng các cụm xe sát nhau và ảnh đêm. Câu 7 cần thêm dữ liệu độc lập để đề xuất ngưỡng tin cậy dùng khi duyệt.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_032.jpg` · CAL-03 | Xích lô đen sát vỉa hè dễ nhầm với xe đạp khi bánh bị khuất. | YOLO26n gọi là `bicycle`, confidence 0,38 (r05). | Gán `three_wheeler` khi nhận ra cấu tạo xích lô. | Mục 2 và 4.3. Tìm cặp bánh cùng trục và ghế chở khách gắn khung. Nếu bánh bị che và dấu hiệu chưa đủ, chọn lớp có bằng chứng tốt nhất và đánh dấu `can_review`. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ hàng rời và giỏ gắn cố định, người lái bị che, cách phân biệt xích lô với xe đạp. | CAL-01, CAL-02, CAL-03 |
