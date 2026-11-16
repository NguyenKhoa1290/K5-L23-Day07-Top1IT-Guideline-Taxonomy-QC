# Guideline gán nhãn — v1.1 (Calibrated) — Nhóm Top1IT

> Bản hiệu chuẩn kỹ thuật (Calibrated Specification) chính thức của nhóm Top1IT sau buổi họp Calibration Pha 5. Tài liệu này cập nhật và làm rõ các hướng dẫn cụ thể dựa trên việc rà soát 596 ca bất thường từ báo cáo IAA (Pha 4), dùng làm căn cứ kiểm định chất lượng (QC) và bàn giao dữ liệu (Pha 6).

- **Phiên bản:** `v1.1`
- **Ngày ban hành:** 2026-11-16
- **Nhóm:** `Top1IT`
- **Thành viên:**
  - Phạm Văn Hùng (`161`) — Reviewer bản cuối
  - Phan Thanh Hào (`165`) — Thành viên
  - Nguyễn Đăng Khoa (`168`) — Người điều phối (Coordinator)
- **Tình trạng:** Khóa cứng sau Calibration cho đợt QC và bàn giao nhãn cuối.

---

## 1. Mục tiêu và Phạm vi

Bộ dữ liệu gồm 40 ảnh chụp quang cảnh giao thông đường phố đô thị tại khu vực Đông Nam Á (Việt Nam, Thái Lan), phục vụ bài toán huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đối tượng 2D (Object Detection) cho hệ thống giám sát giao thông thông minh và xe tự hành (ADAS / Autonomous Driving).

- **Nhiệm vụ của mô hình:** Xác định chính xác vị trí tọa độ (2D Bounding Box) và phân loại đúng các thực thể di chuyển hoặc có nguy cơ va chạm giao thông thực tế.
- **Rủi ro khi có lỗi gán nhãn:**
  - *Bỏ sót (False Negative - fn):* Bỏ sót người đi bộ, người điều khiển phương tiện hoặc xe cơ giới đang lưu thông dẫn tới nguy cơ va chạm đặc biệt nghiêm trọng.
  - *Gán nhầm vật vô tri (False Positive - fp):* Gán nhãn cho tượng đài điêu khắc, hình chân dung áp phích quảng cáo, ma-nơ-canh, mô hình đồ chơi hoặc hình phản chiếu qua gương sẽ gây hiện tượng "phanh ma" (phantom braking) cực kỳ nguy hiểm trong vận hành xe tự lái.
  - *Sai lệch biên box (IoU thấp) hoặc ôm hàng quá khổ:* Làm mô hình ước lượng sai kích thước và khoảng cách an toàn của phương tiện.

---

## 2. Lớp và Thuộc tính

### 2.1. Danh mục lớp (Classes)

Khung taxonomy duy trì 5 lớp đối tượng và 1 lớp vùng bỏ qua (khớp hoàn toàn với `submission/taxonomy.json`):

| Lớp | Định nghĩa chuẩn | Ví dụ trong bộ ảnh | Phản ví dụ / Không gán |
|---|---|---|---|
| `car` | Ô tô con chở người từ 4 đến 9 chỗ (sedan, hatchback, SUV, MPV, xe mui trần, taxi). | Xe AMC Pacer (`img_024`), xe taxi đỗ (`img_040`), xe diễu hành hợp chuẩn (`img_019`). | Xe bán tải (`truck`), ô tô đồ chơi tủ kính (`img_038`), bia đá tạc ô tô (`img_027`), xe con nằm trên sàn xe chở (`img_002`) $\rightarrow$ **Không gán**. |
| `truck` | Xe cơ giới chở hàng có thùng/cabin riêng (xe tải, xe đầu kéo, xe chuyên dụng), xe bán tải (pickup) và xe buýt. | Xe chở xe Renault (`img_002`), xe tải đêm (`img_007`), xe pickup (`img_001`, `img_012`, `img_037`), xe buýt lớn (`img_009`), xe tải nhẹ CMC (`img_036`). | Xe ba bánh tuk-tuk chở hàng (`img_035`) $\rightarrow$ là `motorcycle`; tượng bia đá tạc bán tải (`img_027`) $\rightarrow$ **Không gán**. |
| `van` | Xe ô tô thân kín chở người hoặc hàng hóa, thân vỏ liền khối từ cabin tới đuôi (kiểu Hiace, Transit). | Minibus chở khách nhỏ, xe van bưu chính kín (`img_005`, `img_016`). | Xe buýt lớn (`img_009`) $\rightarrow$ là `truck`; xe tải thùng kín có cabin độc lập (`img_007`) $\rightarrow$ là `truck`. |
| `motorcycle` | Xe mô tô, xe gắn máy 2 bánh có động cơ, xe máy có gắn sidecar và xe cơ giới 3 bánh chở khách/hàng (tuk-tuk, auto-rickshaw). | Xe máy chạy trên đường (`img_005`, `img_008`, `img_026`), mô tô sidecar (`img_022`), xe tuk-tuk chở khách (`img_004`), tuk-tuk chở hàng (`img_016`, `img_035`). | Xe đạp, xích lô không có động cơ (`img_006`, `img_023`, `img_039`) $\rightarrow$ **Không gán** box xe; tượng xe máy trang trí (`img_011`). |
| `person` | Người thật còn sống đang đi bộ, đứng, hoặc đang điều khiển / ngồi trên phương tiện giao thông. | Người đi bộ (`img_001`), người gánh hàng (`img_003`), người điều khiển / ngồi sau xe máy (`img_005`, `img_026`), tài xế tuk-tuk mở (`img_004`). | Tượng đồng Wellington (`img_017`), ma-nơ-canh tủ kính (`img_033`), hình người in trên áp phích (`img_018`), tượng Predator (`img_011`) $\rightarrow$ **Không gán**. |
| `ignore_region` | Vùng đánh dấu không yêu cầu gán chi tiết từng đối tượng đơn lẻ; bỏ sót trong vùng này không tính là lỗi. | Cụm đám đông dày đặc ở xa không thể tách rời từng người (`img_019`, `img_032`), dãy xe máy ở hậu cảnh quá xa và mờ. | Không dùng để che lấp các đối tượng nhìn thấy rõ ranh giới nhằm trốn tránh việc gán box. |

**Xử lý đối tượng ngoài taxonomy (Out-of-taxonomy):**
Phương tiện thô sơ không gắn động cơ như xe đạp, xe lôi, xích lô (`img_006`, `img_023`, `img_039`) **không thuộc phạm vi** 5 lớp phương tiện trên. **Không gán box phương tiện** cho các loại xe này. Tuy nhiên, người đang đạp hoặc ngồi trên các phương tiện này vẫn được gán box lớp `person` theo quy tắc thông thường nếu nhận diện rõ.

---

### 2.2. Thuộc tính (Attributes) & Ngưỡng định lượng đo được

Mọi thuộc tính đều được định nghĩa với ngưỡng định lượng rõ ràng, triệt tiêu cách diễn giải cảm tính:

1. **`occluded_level` (Mức độ che khuất do vật khác chắn phía trước):**
   - **`0` (Không bị che):** $0\%$ diện tích đối tượng bị che khuất, nhìn thấy hoàn toàn trọn vẹn.
   - **`1` (Che nhẹ):** Đối tượng bị che từ $> 0\%$ đến $30\%$ diện tích nhìn thấy.
   - **`2` (Che vừa / Che nhiều):** Đối tượng bị che từ $> 30\%$ đến $70\%$ diện tích, nhưng các đặc trưng nhận diện chính vẫn rõ ràng.
   - **`3` (Che nặng):** Đối tượng bị che $> 70\%$ diện tích, chỉ nhìn thấy một phần nhỏ.
2. **`truncated` (Cắt bởi viền ảnh):**
   - **`no`:** Toàn bộ thân thể hoặc cấu trúc đối tượng nằm trọn vẹn $100\%$ bên trong khung hình ảnh.
   - **`yes`:** Đối tượng bị mép ảnh (trái, phải, trên, dưới) cắt ngang làm mất một phần cấu trúc hình học.
3. **`can_review` (Đánh dấu cần xem lại):**
   - Ở bản nhãn cuối cùng `v1.1`, toàn bộ các ca gắn cờ `can_review` từ vòng gán độc lập đã được nhóm rà soát và đóng lại (`false`) sau khi hiệu chuẩn thống nhất.

---

## 3. Hệ thống Quy tắc vẽ Bounding Box (Rules)

- **`R-01` (Tight Visible Box):** Box ôm sát phần nhìn thấy thực tế, không vẽ suy đoán, loại trừ bóng đổ / vệt nước đèn đường (`img_015`, `img_031`).
- **`R-02` (Tách box Người & Phương tiện):** Người đi xe máy hoặc tuk-tuk hở có box `person` riêng; phương tiện có box `motorcycle` riêng.
- **`R-03` (Xe chở ô tô & Hàng hóa nhô):** Trên xe tải chở xe (`img_002`), chỉ gán duy nhất 1 box `truck` cho xe chở, **tuyệt đối không gán các ô tô con** nằm trên sàn xe chở hàng. Hàng hóa nhô ra ngoài thân xe không tính vào box.
- **`R-04` (Phân loại phương tiện đặc thù):**
  - Xe bán tải (pickup truck) thùng kín hay hở (`img_001`, `img_012`, `img_037`) $\rightarrow$ gán **`truck`**.
  - Xe buýt lớn, xe khách (`img_009`) $\rightarrow$ gán **`truck`**.
  - Xe 3 bánh (tuk-tuk / auto-rickshaw) (`img_004`, `img_016`, `img_035`, `img_040`) $\rightarrow$ gán **`motorcycle`**.
  - Xe sidecar (`img_022`) $\rightarrow$ 1 box **`motorcycle`** ôm cả xe và thuyền sidecar.
- **`R-05` (Loại trừ vật vô tri):** Tuyệt đối không gán tượng điêu khắc (`img_017`), bia đá (`img_027`), đồ chơi tủ kính (`img_038`), ma-nơ-canh (`img_033`), ảnh áp phích quảng cáo (`img_018`), bóng trong gương (`img_020`).
- **`R-06` (Vật nhỏ & Vùng ignore):** Chiều cao cạnh ngắn $< 20\text{px}$ không gán box đơn lẻ. Cụm đám đông dày đặc ở xa không thể tách rời thì dùng 1 box `ignore_region`.
- **`R-07` (Nhòe chuyển động):** Ảnh bị nhòe chuyển động (`img_014`) vẽ box bao trọn cả vùng nhòe nhận diện được.
- **`R-08` (Ngoài taxonomy):** Phương tiện thô sơ không động cơ (xe đạp, xích lô tại `img_006`, `img_023`, `img_039`) không gán box phương tiện, chỉ gán box `person` cho người điều khiển / người ngồi.

---

## 4. Lịch sử Thay đổi Phiên bản

| Phiên bản | Thay đổi chính | Lý do & Căn cứ thực tế |
|---|---|---|
| `v0` | Bản khởi tạo sơ bộ của bài lab | Chưa xử lý ca khó và chưa có định lượng che khuất. |
| `v1.0` | Thống nhất đặc tả ban đầu của nhóm Top1IT | Phân lớp bán tải/buýt $\rightarrow$ `truck`, tuk-tuk $\rightarrow$ `motorcycle`, định lượng che khuất 30/70%, ban hành mã quy tắc `R-01` đến `R-08`. |
| `v1.1` | **Hiệu chuẩn sau họp Calibration (Pha 5):**<br>1. Rà soát và giải quyết dứt điểm 596 ca bất thường từ IAA.<br>2. Xác nhận dứt điểm loại bỏ các box ô tô con trên sàn xe chở xe `img_002` theo `R-03`.<br>3. Thống nhất áp dụng chuẩn hóa `occluded_level` cho các đối tượng che khuất sâu ở hậu cảnh.<br>4. Đóng toàn bộ các cờ `can_review` sau khi đạt đồng thuận nhóm. | Dựa trên kết quả đo IAA tại [iaa_summary.md](file:///c:/Users/phant/K5-L23-Day07-Top1IT-Guideline-Taxonomy-QC/submission/qc/iaa_summary.md) và các ca trong [calibration_candidates.csv](file:///c:/Users/phant/K5-L23-Day07-Top1IT-Guideline-Taxonomy-QC/submission/qc/calibration_candidates.csv). |
