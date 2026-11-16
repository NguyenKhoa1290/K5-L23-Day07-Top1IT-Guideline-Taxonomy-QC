# Tổng hợp guideline cá nhân — tài liệu họp nhóm

> Tài liệu tổng hợp các bản cá nhân 161, 165 và 168 để chuẩn bị họp. Đây chưa phải guideline v1.0 đã thống nhất; mọi mục cần quyết định phải do nhóm chốt. Đối chiếu lại ảnh gốc khi thảo luận.

## 1. Mục tiêu và phạm vi

Ba bản đều hướng tới gán bounding box cho người và phương tiện trong ảnh để tạo dữ liệu phát hiện đối tượng. Taxonomy hiện tại có các lớp `person`, `car`, `truck`, `van`, `motorcycle`, `ignore_region` và các thuộc tính `occluded_level`, `truncated`, `can_review`.

Hướng dẫn nhóm cần xác định rõ vật nào thuộc phạm vi, vật nào bị loại và các trường hợp cần review; taxonomy v1.0 phải phản ánh quyết định của nhóm.

## 2. Các điểm tương đồng để kiểm tra và chốt

- Mỗi đối tượng được nhận dạng rõ có box riêng, ôm sát phần nhìn thấy.
- `person` gồm người thật, kể cả người đang điều khiển hoặc ngồi trên phương tiện; cần chốt cụ thể số box và phạm vi từng box khi người đi xe.
- `truncated` mô tả việc đối tượng bị cắt bởi mép ảnh; `can_review` đánh dấu box cần kiểm tra thêm.
- `ignore_region` là vùng không gán chi tiết từng vật, có thể dùng cho vùng quá đông hoặc quá nhỏ; điều kiện áp dụng và ranh giới vùng cần chốt cụ thể.
- Người và phương tiện không xác định đủ chắc cần được xem lại; cần phân biệt box có thể đánh dấu `can_review` với trường hợp chưa đủ cơ sở vẽ box và phải ghi vấn đề cấp ảnh.

Các điểm trên là phần tổng hợp để xác nhận, không thay thế quyết định của nhóm trên ảnh thực tế.

## 3. Các ca và quy tắc cần họp

| Chủ đề | Khác biệt / câu hỏi từ các bản cá nhân | Ảnh gợi ý đối chiếu | Quyết định nhóm |
|---|---|---|---|
| Tuk-tuk | 161 tạm xếp vào `motorcycle`; 165 đề xuất lớp riêng `tuk_tuk`; 168 loại khỏi định nghĩa `motorcycle`. Nhóm chọn cách nào? | `img_004.jpg`, `img_035.jpg` | Tạm xếp là nhóm xe máy |
| Xe bán tải | 161 xếp pickup vào `truck`; 165 xếp bán tải thông thường vào `car`. Phân biệt thế nào với xe có nắp thùng cao? | `img_001.jpg`, `img_037.jpg` | xe tải |
| Xe buýt | 161 đề xuất không gán trong taxonomy hiện có; 165 đề xuất thêm `bus` và nêu phương án tạm khác nếu giữ v0. Có đưa xe buýt vào phạm vi không? | `img_009.jpg`, `img_037.jpg` | Xếp vào truck |
| Che khuất | 165 và 168 dùng ranh giới 30% / 70% theo tỷ lệ bị che; 161 đưa ra các khoảng khác. Chốt cách ước lượng, ngưỡng không chồng lấn và trường hợp không đủ bằng chứng. | Tự chọn ảnh có vật bị che | Dùng 30-70 |
| Vật không thật và vật cản | 161 và 165 loại tượng, đồ chơi và hình phản chiếu; 168 đề xuất ngoại lệ vật cản và gán mô hình xe ở `img_027`. Một vật không thật có được gán không, nếu có thì theo tiêu chí và lớp nào? | `img_027.jpg`, `img_038.jpg` | Không gán (áp dụng R-05) |
| Phản chiếu và hình in | Xác định ảnh trong gương, hình trên biển quảng cáo và vật thật trong cùng ảnh được xử lý ra sao. | `img_018.jpg`, `img_020.jpg` | Không gán |
| Xe chở xe | Gán xe chở và từng xe được chở thế nào? Box xe chở có tính hàng hóa hay không? | `img_002.jpg` | Chỉ gán xe chở |
| Người trên xe | Xác định số box cho người lái, người ngồi sau và xe; quy định phạm vi box khi chúng che nhau. | `img_005.jpg`, `img_026.jpg` | Giống quy tắc 30-70 |
| Ảnh nhòe và bóng ma | 168 đề cập gán cả bóng ma; nhóm cần chốt một đối tượng có bao nhiêu box và ranh giới box khi ảnh bị nhòe. | `img_014.jpg` | Gán hết vùng nhòe |
| Vật nhỏ và đám đông | Khi nào gán riêng, khi nào dùng `ignore_region`? Taxonomy hiện có `min_box_height_px = 20`; cần chốt cách xử lý từng tình huống và không thay đổi mục `rules`. | Tự chọn ảnh có vật nhỏ | Không gán |
| Ngoài taxonomy | Xe đạp, xích lô và các loại không có lớp: bỏ qua, đánh dấu vấn đề hay bổ sung lớp trước khi khóa? Không tự gán sang lớp không phù hợp. | `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | không đánh |

## 4. Kết quả cần có sau cuộc họp

1. Ghi quyết định cụ thể, lý do và mã quy tắc `R-01`, `R-02`… vào `submission/guidelines/guideline_v1.0_agreed.md`. Với mỗi ca, thử quy tắc trên ảnh gốc để cả nhóm dự đoán nhất quán số box và lớp.
2. Cập nhật `submission/taxonomy.json` theo guideline, đặt `version` là `v1.0` và giữ nguyên mục `rules`.
3. Chạy `python scripts/make_cvat_labels.py` để kiểm schema và tạo nhãn CVAT; chỉ khóa v1.0 sau khi nhóm xác nhận.
4. Sau khi khóa v1.0, chia ảnh cho ba mã bằng `python scripts/split_assign.py --members 161,165,168 --copy-images`; mỗi ảnh được hai người gán độc lập. Mỗi người gửi nhãn riêng trước khi tập hợp.

Giữ nguyên các bản cá nhân sau khi họp. Trước khi nộp, kiểm tra tên theo mẫu `guideline_<mã>.md` và các yêu cầu về thông tin cá nhân trong `SUBMISSION.md`.
