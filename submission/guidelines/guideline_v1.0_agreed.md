# Guideline gán nhãn — v1.0 — Nhóm Top1IT

> Tài liệu đặc tả kỹ thuật gán nhãn Bounding Box (2D) chính thức đã được toàn bộ nhóm Top1IT thống nhất sau buổi họp Pha 2. Tài liệu này đóng vai trò là chuẩn quy tắc duy nhất để các thành viên thực hiện lượt gán nhãn độc lập (Pha 3), đo lường độ đồng thuận IAA (Pha 4), hiệu chuẩn Calibration (Pha 5) và đánh giá kiểm soát chất lượng QC (Pha 6).

- **Phiên bản:** `v1.0`
- **Ngày thống nhất:** 2026-11-16
- **Nhóm:** `Top1IT`
- **Thành viên tham gia:**
  - Phạm Văn Hùng (`161`) — Reviewer bản cuối
  - Phan Thanh Hào (`165`) — Thành viên
  - Nguyễn Đăng Khoa (`168`) — Người điều phối (Coordinator)
- **Tình trạng:** Khóa cứng cho lượt gán độc lập Pha 3.

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

Khung taxonomy v1.0 duy trì 5 lớp đối tượng và 1 lớp vùng bỏ qua (khớp hoàn toàn với `submission/taxonomy.json`):

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
   - **`1` (Che nhẹ):** Đối tượng bị che từ $> 0\%$ đến $30\%$ diện tích nhìn thấy (ví dụ: người đi bộ đứng che một góc cản xe ô tô).
   - **`2` (Che vừa / Che nhiều):** Đối tượng bị che từ $> 30\%$ đến $70\%$ diện tích, nhưng các đặc trưng nhận diện chính vẫn rõ ràng (ví dụ: xe máy bị ô tô khác che mất nửa thân dưới).
   - **`3` (Che nặng):** Đối tượng bị che $> 70\%$ diện tích, chỉ nhìn thấy một phần nhỏ (ví dụ: chỉ thấy đầu và vai người trong đám đông dày đặc). Mức 3 bắt buộc cân nhắc kết hợp với thuộc tính `can_review = true`.

2. **`truncated` (Cắt bởi viền ảnh):**
   - **`no`:** Toàn bộ thân thể hoặc cấu trúc đối tượng nằm trọn vẹn $100\%$ bên trong khung hình ảnh.
   - **`yes`:** Đối tượng bị mép ảnh (trái, phải, trên, dưới) cắt ngang làm mất một phần cấu trúc hình học.

3. **`can_review` (Đánh dấu cần xem lại):**
   - **`false` (không tick - mặc định):** Người gán tự tin áp dụng quy tắc rõ ràng, không có tranh chấp hay mơ hồ.
   - **`true` (tick chọn):** Trường hợp biên khó phân định, đối tượng có độ nghi ngờ cao (ví dụ: xe bán tải nắp cao `img_037`, xe bị nhòe chuyển động nặng `img_014`, góc khuất phức tạp) cần mang ra hiệu chuẩn trong buổi họp Calibration.

---

## 3. Hệ thống Quy tắc vẽ Bounding Box (Rules)

Các quy tắc được định danh bằng mã chuẩn `R-01` đến `R-08` để dẫn chiếu thống nhất trong quá trình QC và Calibration:

### `R-01` — Nguyên tắc ôm sát phần nhìn thấy (Tight Visible Box)
- Bounding box phải ôm sát các điểm biên ngoài cùng của **phần thực tế nhìn thấy được** của đối tượng, sai số cho phép không quá $2\text{px}$.
- Tuyệt đối **không vẽ suy đoán** (mental completion) phần cơ thể hoặc thân xe bị che khuất bởi vật khác. Phần bị che được phản ánh qua thuộc tính `occluded_level`.
- Box bao gồm: thân xe, gương chiếu hậu, tay lái, mũ bảo hiểm, tay chân người.
- Box **loại trừ**: bóng đổ dưới lòng đường, vệt sáng đèn pha trên mặt đường ướt (`img_015`, `img_031`), và các vật thể nền phía sau.

### `R-02` — Tách box Người và Phương tiện (Rider & Vehicle Separation)
- Đối với xe máy, mô tô và xe tuk-tuk: **Gán box riêng biệt** cho người và phương tiện.
  - Mỗi người (người điều khiển, người ngồi sau) được gán 1 box `person`.
  - Phương tiện được gán 1 box `motorcycle`.
  - Box `person` và `motorcycle` được phép chồng lấn không gian lớn lên nhau.
  - Ví dụ: 3 người ngồi trên 1 xe máy (`img_005`) $\rightarrow$ Vẽ 1 box `motorcycle` và 3 box `person`.
- Đối với xe ô tô/xe van kín: Người ngồi bên trong xe chỉ nhìn thấy mờ qua kính chắn gió thì **không gán** box `person`.
- Đối với người trên xe đạp/xích lô: Chỉ gán 1 box `person`, không gán box cho xe đạp/xích lô (`R-08`).

### `R-03` — Xe chở xe (Car Carrier) & Hàng hóa nhô quá khổ
- **Xe tải chở ô tô chuyên dụng (`img_002`):** Toàn bộ tổ hợp đầu kéo và sàn rơ-moóc được gán bằng **1 box `truck` duy nhất**. **Không gán từng ô tô con** nằm trên sàn xe chở hàng, vì chúng đóng vai trò là kiện hàng hóa được chuyên chở trong cấu trúc xe.
- **Hàng hóa nhô quá khổ:** Hàng hóa, vật dụng cồng kềnh chất nhô ra ngoài kích thước xe (gánh hàng `img_003`, cây dài chở trên xe máy `img_029`, bao tải chất cao trên tuk-tuk `img_035`) **không tính vào ranh giới box** xe/người. Box chỉ ôm sát kết cấu chuẩn của xe và cơ thể người. Nếu hàng nằm gọn trong lòng thùng xe tải/pickup (`img_012`) thì tính vào box `truck`.

### `R-04` — Phân loại các phương tiện đặc thù
- **Xe bán tải (Pickup truck):** Tất cả các dòng xe pickup chở hàng thùng hở hoặc có gắn nắp thùng cao (như Ford Ranger, Isuzu D-Max, Master 75 tại `img_001`, `img_012`, `img_037`) đều được xếp vào lớp **`truck`**.
- **Xe buýt / Xe khách lớn:** Do không có lớp `bus` riêng biệt trong taxonomy v0, nhóm thống nhất xếp xe buýt lớn và xe khách đường dài (`img_009`) vào lớp **`truck`** (dựa trên kích thước khung gầm xe tải nặng).
- **Xe ba bánh cơ giới (Tuk-tuk / Auto-rickshaw):** Toàn bộ các dòng xe cơ giới 3 bánh chở khách hoặc chở hàng (`img_004`, `img_016`, `img_035`, `img_040`) được xếp vào lớp **`motorcycle`**.
- **Mô tô có sidecar (`img_022`):** Gán 1 box `motorcycle` duy nhất ôm trọn cả phần xe mô tô và thuyền gắn phụ (sidecar).

### `R-05` — Loại trừ vật vô tri không có động năng (Negative Cases)
- **Tượng đài, bia điêu khắc:** Tượng đồng Công tước Wellington (`img_017`), bia đá tưởng niệm tạc hình xe bán tải (`img_027`), tượng quái vật kim loại (`img_011`) $\rightarrow$ **Tuyệt đối không gán**. Tượng là vật tĩnh vô tri, không tham gia giao thông; gán sẽ gây lỗi false positive nguy hiểm cho xe tự lái.
- **Mô hình đồ chơi, ma-nơ-canh:** Ô tô đồ chơi trong tủ kính (`img_038`), ma-nơ-canh trưng bày quần áo (`img_033`) $\rightarrow$ **Không gán**.
- **Hình in quảng cáo và ảnh phản chiếu:** Hình chân dung người trên biển quảng cáo/áp phích (`img_018`), bóng xe phản chiếu qua gương chiếu hậu (`img_020`) $\rightarrow$ **Không gán**.

### `R-06` — Ngưỡng kích thước tối thiểu & Vùng bỏ qua (`ignore_region`)
- **Ngưỡng kích thước tối thiểu (`min_box_height_px = 20`):** Các đối tượng riêng lẻ có chiều cao cạnh dưới $20\text{px}$ (vật quá xa hoặc quá nhỏ) thì **không gán box đơn lẻ**.
- **Áp dụng `ignore_region`:** Khi có cụm nhiều đối tượng tụ lại ở xa hoặc đám đông dày đặc mà phần lớn các đối tượng bị che khuất nặng (`occluded_level = 3`) và chồng lấn không thể phân biệt ranh giới rõ ràng (`img_019`, `img_032`), vẽ 1 box đa giác hoặc chữ nhật với lớp `ignore_region` bao trọn cụm này.

### `R-07` — Xử lý ảnh nhòe chuyển động (Motion Blur) & Thiếu sáng
- **Nhòe chuyển động (`img_014`):** Nếu đối tượng bị nhòe do tốc độ màn trập chậm nhưng vẫn nhận biết chắc chắn được loại đối tượng (người đi xe máy), vẽ bounding box **ôm trọn toàn bộ vùng nhận diện được bao gồm cả vệt nhòe chuyển động** của đối tượng, đồng thời tick `can_review = true` nếu biên quá mơ hồ.
- **Ảnh thiếu sáng / ban đêm / mưa (`img_007`, `img_013`, `img_031`):** Vẫn gán box cho các đối tượng nhận diện rõ bằng mắt thường. Đối tượng chỉ là đốm sáng hoặc vệt mờ không xác định được hình thái thì bỏ qua.

### `R-08` — Xử lý phương tiện ngoài taxonomy
- Các phương tiện thô sơ không động cơ (xe đạp, xích lô, xe kéo tay tại `img_006`, `img_023`, `img_039`): **Không gán box cho phương tiện**. Chỉ gán box `person` cho người đạp xe / người ngồi nếu nhìn rõ.

---

## 4. Bảng Tra cứu Ca khó Tiêu biểu (11 Ca Đối chiếu Nhóm)

Bảng đối chiếu tổng hợp quyết định của nhóm từ tài liệu họp nhóm và 40 ảnh thực tế:

| Ảnh | Tình huống đặc thù | Rule ID | Quyết định nhóm Top1IT | Lý do kỹ thuật |
|---|---|:---:|---|---|
| `img_001.jpg`, `img_037.jpg` | Xe bán tải (pickup truck), xe bán tải gắn nắp thùng cao | `R-04` | Xếp vào lớp **`truck`** | Khung gầm tải và công năng chuyên chở hàng hóa riêng biệt; tránh gây nhầm lẫn với xe con 4 chỗ. |
| `img_002.jpg` | Xe tải chuyên dụng chở 8 ô tô con trên sàn hai tầng | `R-03` | Chỉ gán **1 box `truck`** cho xe chở; **không gán** 8 xe con trên sàn | Ô tô con đóng vai trò hàng hóa chuyên chở trên phương tiện; gán xe con sẽ làm mô hình hiểu sai vị trí mặt đường lưu thông. |
| `img_004.jpg`, `img_035.jpg` | Xe 3 bánh cơ giới tuk-tuk / Auto Rickshaw | `R-04` | Xếp vào lớp **`motorcycle`** | Phương tiện cơ giới nhỏ, cơ động, cabin mở, cùng nhóm tải trọng và hành vi giao thông với xe máy. |
| `img_005.jpg`, `img_026.jpg` | Người điều khiển và nhiều người ngồi sau xe máy | `R-02` | Tách riêng: **1 box `motorcycle`** + **mỗi người 1 box `person`** | Mô hình cần phát hiện độc lập sự hiện diện của người đi bộ/người đi xe để cảnh báo va chạm sinh mạng. |
| `img_006.jpg`, `img_023.jpg`, `img_039.jpg` | Xe đạp, xích lô đạp (phương tiện thô sơ ngoài taxonomy) | `R-08` | **Không gán xe**; chỉ gán **`person`** cho người | Taxonomy v1.0 không có lớp `bicycle`/`cyclo`; không tự ý ép sang lớp cơ giới khác. |
| `img_007.jpg`, `img_031.jpg` | Xe tải chở hàng trong đêm tối, đường ướt mưa phản chiếu | `R-01`, `R-07` | Gán **`truck`** ôm sát thân xe; **loại trừ vệt nước phản chiếu** | Box chỉ ôm thân vỏ cơ học; vệt nước sáng trên đường không phải bộ phận phương tiện. |
| `img_009.jpg` | Xe buýt lớn, xe khách chở khách liên tỉnh | `R-04` | Xếp vào lớp **`truck`** | Kích thước lớn, khung gầm hạng nặng; tạm xếp vào nhóm phương tiện tải trọng lớn do thiếu lớp `bus`. |
| `img_014.jpg` | Người đi xe máy bị nhòe chuyển động (motion blur) | `R-07` | Gán **ôm hết vùng nhòe**, tick `can_review = true` | Đảm bảo bao quát toàn bộ vùng ảnh hưởng của vật thể chuyển động tốc độ cao. |
| `img_017.jpg`, `img_027.jpg` | Tượng đồng Công tước Wellington đội nón, Bia đá tạc hình bán tải | `R-05` | **Tuyệt đối không gán** | Vật tĩnh vô tri, không có động năng tham gia giao thông; gán sẽ gây lỗi báo động giả (false positive). |
| `img_018.jpg`, `img_020.jpg` | Hình chân dung áp phích quảng cáo, Xe trong gương chiếu hậu | `R-05` | **Tuyệt đối không gán** | Không phải thực thể thực tế trên lòng đường. |
| `img_038.jpg` | Mô hình xe ô tô đồ chơi trưng bày qua tủ kính | `R-05` | **Tuyệt đối không gán** | Đồ chơi trong nhà, không có giá trị giao thông đường phố. |

---

## 5. Tiêu chuẩn QC và Lịch sử Phiên bản

### 5.1. Ngưỡng kiểm soát chất lượng (QC Thresholds)
Áp dụng nghiêm ngặt theo các tham số khóa trong `submission/taxonomy.json`:
- `min_box_height_px`: $20\text{px}$ (vật nhỏ hơn bỏ qua hoặc gán `ignore_region`).
- `large_px`: $50\text{px}$ (ranh giới phân định vật lớn / vật nhỏ).
- `iou_thr_large`: $0.85$ (ngưỡng IoU tối thiểu đối với vật lớn $\ge 50\text{px}$).
- `iou_thr_small`: $0.70$ (ngưỡng IoU tối thiểu đối với vật nhỏ $< 50\text{px}$).
- `iaa_iou`: $0.85$ (ngưỡng độ tương đồng IoU liên người gán).
- `iaa_class`: $0.95$ (ngưỡng đồng thuận phân loại lớp).
- `error_rate_img_max`: $0.02$ (tỷ lệ lỗi tối đa cho phép trên mỗi ảnh).
- `match_iou`: $0.50$ (ngưỡng ghép cặp đối tượng giữa các annotator).

### 5.2. Lịch sử thay đổi phiên bản

| Phiên bản | Thay đổi chính | Lý do kỹ thuật & Căn cứ ảnh |
|---|---|---|
| `v0` | Bản khởi tạo sơ bộ của bài lab | Chưa có định lượng che khuất, chưa xử lý ca khó (tuk-tuk, tượng, xe chở xe). |
| `v1.0` | **Bản chuẩn hóa chính thức của nhóm Top1IT:**<br>1. Phân loại chuẩn xác: Tuk-tuk $\rightarrow$ `motorcycle`; Bán tải & Xe buýt $\rightarrow$ `truck`.<br>2. Định lượng `occluded_level` thành các mốc rõ ràng: $0\%$ (0), $1\text{–}30\%$ (1), $31\text{–}70\%$ (2), $> 70\%$ (3).<br>3. Ban hành quy tắc loại trừ vật vô tri (`R-05`): không gán tượng đài, đồ chơi, ma-nơ-canh, hình in quảng cáo, hình phản chiếu.<br>4. Quy định xe chở xe (`R-03`): chỉ gán 1 box xe tải chở, không gán các xe con trên sàn.<br>5. Bổ sung hệ thống định danh mã quy tắc `R-01` $\rightarrow$ `R-08`. | Khảo sát thực tế trên toàn bộ 40 ảnh (`img_001` đến `img_040`) và thống nhất biên bản cuộc họp nhóm [tong_hop_hop_nhom.md](file:///c:/Users/phant/K5-L23-Day07-Top1IT-Guideline-Taxonomy-QC/submission/guidelines/tong_hop_hop_nhom.md). |
