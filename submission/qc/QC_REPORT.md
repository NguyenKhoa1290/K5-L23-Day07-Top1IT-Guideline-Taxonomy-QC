# Báo cáo QC — Top1IT

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và kiểm định chất lượng (QC) dữ liệu bounding box 2D cho bài toán phát hiện đối tượng giao thông đường phố; dữ liệu gồm 40 ảnh tuyển chọn cho các ca biên, chưa đại diện cho mọi điều kiện thời tiết thực tế.
- Nguồn và phép sử dụng dữ liệu: Toàn bộ ảnh được khai thác từ nguồn mở Wikimedia Commons với giấy phép hợp lệ CC BY / CC BY-SA / CC0 ghi nhận đầy đủ tại data/ATTRIBUTION.csv.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256 nhãn cuối labels_calibrated.json: e56ce288ca0a9bdf2e1cf8c4002567796332dad7f16c370fbccfac03206f010e).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng guideline_v1.0_agreed.md và taxonomy.json (v1.0); khi bàn giao dùng guideline_v1.1_calibrated.md sau khi hiệu chuẩn giải quyết 596 ca bất thường, giữ nguyên cấu trúc taxonomy v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (100% hoàn thành phân công gán mù đôi 40 ảnh cho 3 thành viên 161, 165, 168).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 205 cặp box ghép trên 40 ảnh; IoU trung bình đạt 0.815; đồng thuận phân loại lớp đạt 93.2%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 43 box chưa ghép bên A và 251 box chưa ghép bên B (chủ yếu do các vật nhỏ ở hậu cảnh xa và xe trên sàn xe chở); 0 box bị loại do ignore; 6 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Tại img_017.jpg (tượng Wellington) và img_027.jpg (bia đá bán tải), người gán có thể cùng vẽ box nếu không đọc kỹ quy tắc R-05; nhóm đã kiểm tra và xác nhận loại bỏ hoàn toàn các ca này.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ 596 ca bất thường trong submission/qc/calibration_decisions.csv; dẫn chiếu các mã quy tắc R-01 đến R-08; người chốt là 168 và người kiểm lại là 161.
- Tác động v1.1 lên nhãn và ảnh: Bản v1.1 làm rõ quy tắc loại bỏ xe con trên xe chở img_002 (R-03), loại bỏ ảnh phản chiếu gương img_020 (R-05), chuẩn hóa occluded_level 30-70% và đóng toàn bộ cờ can_review.
- Tham chiếu bằng chứng khi Rework: Giữ nguyên lịch sử từ vòng gán độc lập đến vòng hiệu chuẩn calibrated và snapshot vòng QC round-01.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 161, 2026-11-16
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/490 = 0.0%
- Vật bỏ sót, ảnh và vùng: Không có vật bỏ sót sau vòng hiệu chuẩn calibration.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 39/39 ca khó và 1/1 ảnh còn lại, không còn lỗi lặp; phiếu chi tiết tại submission/qc/reviews/round-01/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được là sai lệch định dạng schema thuộc tính hoặc box vượt quá viền ảnh; lỗi cần người đọc ảnh là phân biệt xe thật với hình phản chiếu qua gương chiếu hậu (img_020) hoặc tượng đài tĩnh (img_027).

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Đạt tỷ lệ lỗi 0.0% trên cả 40 ảnh đã duyệt toàn lô, IoU và phân loại lớp nhất quán với guideline v1.1; bàn giao cho bên thụ hưởng để huấn luyện thử nghiệm mô hình phát hiện đối tượng 2D.
- Nếu còn việc: Không còn tồn đọng, toàn bộ dữ liệu đã sẵn sàng nghiệm thu bàn giao.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca img_002.jpg áp dụng quy tắc R-03 chỉ giữ 1 box truck xe chở tại tọa độ [140.1, 140.3, 752.1, 399.6], kết quả kiểm tra khớp chính xác giữa nhãn calibrated, CSV và manifest.
