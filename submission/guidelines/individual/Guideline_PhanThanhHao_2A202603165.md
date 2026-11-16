# Guideline gán nhãn — v0.1 (Bản cá nhân độc lập) — Thành viên 165

- Phiên bản: `v0.1_individual_165`
- Ngày: 2026-11-16
- Người viết: Mã thành viên `165`
- Tình trạng: Bản cá nhân độc lập Pha 1 (khảo sát toàn bộ 40 ảnh từ `img_001.jpg` đến `img_040.jpg`)

---

## 1. Mục tiêu

Tập dữ liệu này cung cấp nhãn Bounding Box (2D) dùng để huấn luyện và đánh giá mô hình phát hiện vật thể (Object Detection) cho hệ thống thị giác máy tính trong giao thông đô thị và hỗ trợ xe tự hành (ADAS / Autonomous Driving).

- **Nhiệm vụ của mô hình:** Xác định chính xác vị trí không gian (tọa độ bounding box) và phân loại đúng các thực thể di chuyển hoặc tham gia giao thông trên đường phố (phương tiện cơ giới, người điều khiển, người đi bộ).
- **Mức độ rủi ro khi có lỗi gán nhãn:**
  - *Bỏ sót (False Negative - fn):* Bỏ sót người đi bộ, người đi xe máy hoặc xe cộ đang lưu thông dẫn tới nguy cơ va chạm giao thông đặc biệt nghiêm trọng.
  - *Gán nhầm vật vô tri (False Positive - fp):* Gán tượng đá/đồng, hình in biển quảng cáo, ma-nơ-canh, ảnh phản chiếu gương chiếu hậu thành người/xe thật sẽ gây hiện tượng "phanh ma" (phantom braking) làm dừng xe đột ngột nguy hiểm trên đường.
  - *Lệch ranh giới box (IoU thấp) hoặc ôm cả hàng hóa quá khổ:* Khiến mô hình ước lượng sai kích thước và ranh giới an toàn của phương tiện.

---

## 2. Lớp và thuộc tính

### 2.1. Danh sách lớp (Classes)

| Lớp | Định nghĩa (một câu) | Ví dụ thực tế | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, xe du lịch từ 4 đến 9 chỗ, xe mui trần, xe bán tải chở khách/hàng thông thường (thùng hở). | Ô tô con trên sàn xe chở (`img_002.jpg`), xe AMC Pacer (`img_024.jpg`). | Mô hình ô tô đồ chơi trong tủ kính (`img_038.jpg`), xe trong gương chiếu hậu (`img_020.jpg`), tượng đài xe bán tải bằng đá (`img_027.jpg`) $\rightarrow$ **Không gán**. |
| `truck` | Xe cơ giới chở hàng có cabin độc lập và thùng xe/rơ-moóc chuyên dụng tải trọng lớn. | Xe chở xe Renault 380 (`img_002.jpg`), xe tải chở hàng đêm (`img_007.jpg`), xe tải nhỏ CMC Veryca (`img_036.jpg`). | Xe tuk-tuk 3 bánh chở hàng (`img_035.jpg`) $\rightarrow$ không gán truck; tượng xe bán tải (`img_027.jpg`) $\rightarrow$ không gán. |
| `van` | Xe ô tô chở người hoặc hàng hóa có thân vỏ liền khối kín từ đầu tới đuôi xe. | Xe cứu thương, minibus chở khách nhỏ, xe van bưu chính kín. | Xe bán tải gắn nắp thùng cao Master 75 (`img_037.jpg`) $\rightarrow$ kết cấu khung gầm bán tải, cần review nhóm. |
| `motorcycle` | Xe mô tô, xe gắn máy, xe 2 bánh hoặc 3 bánh có gắn động cơ chở người/hàng. | Xe máy lưu thông (`img_005.jpg`, `img_008.jpg`, `img_026.jpg`, `img_030.jpg`), xe máy có sidecar thuyền phụ (`img_022.jpg`). | Xe đạp, xích lô chở khách không động cơ (`img_006.jpg`, `img_023.jpg`, `img_039.jpg`) $\rightarrow$ không gán `motorcycle`; tượng xe máy trang trí (`img_011.jpg`). |
| `person` | Người thật còn sống đang đi bộ, đứng, hoặc đang điều khiển / ngồi trên phương tiện giao thông. | Phụ nữ váy đen đứng cạnh xe (`img_001.jpg`), người bán mít (`img_003.jpg`), người lái xe máy (`img_005.jpg`, `img_026.jpg`). | Tượng đồng Wellington (`img_017.jpg`), ma-nơ-canh tủ kính (`img_033.jpg`), hình chân dung trên biển quảng cáo (`img_018.jpg`), tượng quái vật Predator (`img_011.jpg`) $\rightarrow$ **Không gán**. |
| `tuk_tuk` *(Đề xuất nhóm)* | Phương tiện cơ giới 3 bánh hoặc xe lam chở khách/hàng hóa có cabin mở hoặc mui bạt (Auto Rickshaw). | Hai xe tuk-tuk xanh (`img_004.jpg`), xe tuk-tuk chở hàng ở Bangkok (`img_016.jpg`), Auto Rickshaw chở tải nặng (`img_035.jpg`). | v0 chưa có lớp này; nếu chưa bổ sung taxonomy thì đánh dấu `can_review=true` để họp nhóm ở Pha 2. |
| `ignore_region` | Vùng đánh dấu không yêu cầu gán chi tiết từng đối tượng đơn lẻ. | Đám đông dày đặc ở xa không thể tách người, cụm phương tiện đỗ dồn kích thước $< 20\text{px}$. | Không dùng `ignore_region` để che giấu các vật nhìn rõ ranh giới nhằm tránh phải gán. |

*Xử lý đối tượng ngoài taxonomy:* Các phương tiện phi cơ giới như xích lô, xe đạp thô sơ (`img_006.jpg`, `img_023.jpg`, `img_039.jpg`) không thuộc 5 lớp phương tiện của v0: chỉ gán box `person` cho người đạp/ngồi trên xe nếu thấy rõ, không gán box xe cho xích lô/xe đạp (chờ Pha 2 nhóm quyết định có bổ sung lớp `bicycle`/`cyclo` hay không).

---

### 2.2. Danh sách thuộc tính (Attributes) & Ngưỡng định lượng đo được

Mọi thuộc tính đều có giá trị cố định và tiêu chuẩn đo lường định lượng thay vì dùng từ ngữ định tính:

1. **`occluded_level` (Mức độ che khuất):**
   - **`0` (Clear - Nhìn rõ):** Đối tượng hoàn toàn không bị che khuất ($0\%$ diện tích bị che).
   - **`1` (Che nhẹ):** Đối tượng bị che khuất từ $1\%$ đến $30\%$ diện tích bề mặt bởi vật cản phía trước.
   - **`2` (Che vừa / Che nhiều):** Đối tượng bị che khuất từ $31\%$ đến $70\%$ diện tích bề mặt, nhưng vẫn đủ đặc trưng nhận diện chắc chắn.
   - **`3` (Che nặng):** Đối tượng bị che khuất $> 70\%$ diện tích bề mặt, chỉ thấy một phần nhỏ nhưng nhờ ngữ cảnh vẫn nhận biết được sự hiện diện.
2. **`truncated` (Cắt mép ảnh):**
   - **`no`:** Toàn bộ thân vật thể nằm trọn vẹn $100\%$ bên trong khung hình ảnh.
   - **`yes`:** Vật thể bị biên ảnh (trên, dưới, trái, phải) cắt ngang qua thân làm mất một phần cấu trúc.
3. **`can_review` (Cần đánh giá lại):**
   - **`false` (không tick):** Người gán tự tin áp dụng quy tắc rõ ràng, không có tranh chấp.
   - **`true` (tick):** Trường hợp biên mơ hồ, không chắc chắn về phân loại lớp (ví dụ xe bán tải nắp cao `img_037.jpg`), vật bị nhòe chuyển động nặng (`img_014.jpg`), hoặc ranh giới phân tách chưa rõ.

---

## 3. Quy tắc vẽ Bounding Box

1. **Nguyên tắc ôm sát phần nhìn thấy (Tight Visible Bounding Box):**
   - Tọa độ 4 cạnh của box phải ôm sát các pixel nhìn thấy được của vật thể. Không tự ý phỏng đoán mở rộng box qua những phần bị che khuất hoàn toàn phía sau vật cản lớn.
   - Tọa độ box tuyệt đối không được trượt ra ngoài mép ảnh ($x \ge 0, y \ge 0, x+w \le \text{width}, y+h \le \text{height}$).
2. **Quy tắc Người đi xe máy (`img_005.jpg`, `img_026.jpg`, `img_030.jpg`):**
   - **Tách riêng 2 box độc lập:**
     - 1 box `person`: Ôm trọn người điều khiển (từ đỉnh đầu/mũ bảo hiểm đến bàn chân đặt trên gác chân xe).
     - 1 box `motorcycle`: Ôm trọn xe máy (từ lốp trước đến đuôi lốp sau, gương chiếu hậu và tay lái).
   - Nếu có người ngồi sau (hành khách): Vẽ thêm 1 box `person` riêng cho người ngồi sau.
   - Gán `occluded_level = 1` hoặc `2` cho cả người và xe do phần thân dưới của người và yên xe che khuất lẫn nhau.
3. **Quy tắc Xe chở xe (`img_002.jpg`):**
   - **Gán cả 2 cấp độ:**
     - 1 box `truck` bao toàn bộ đầu kéo và khung giá chở xe của chiếc Renault 380.
     - 8 box `car` riêng lẻ cho từng chiếc ô tô con nằm trên 2 tầng giá đỡ xe (vì mỗi chiếc ô tô con đều là một thực thể ô tô hoàn chỉnh, có đặc trưng nhận dạng rõ ràng).
4. **Quy tắc Hàng hóa và Phụ kiện cồng kềnh (`img_012.jpg`, `img_029.jpg`, `img_035.jpg`):**
   - Hàng hóa nằm gọn trong lòng thùng xe tải/bán tải: Box xe ôm sát cả thùng xe lẫn hàng hóa bên trong.
   - Hàng hóa chất quá khổ vượt khung xe (ví dụ Auto Rickshaw chất bao tải cao ngất ở `img_035.jpg`, hoặc xe máy chở cồng kềnh ở `img_029.jpg`): Bbox xe chỉ ôm thân xe cơ giới; không kéo dãn box bao trùm khối hàng khổng lồ để tránh làm sai lệch kích thước phương tiện.
5. **Quy tắc Loại trừ đối tượng phi thực thể (Negative Cases):**
   - **Tượng điêu khắc, đài tưởng niệm:** Tượng đồng Duke of Wellington (`img_017.jpg`), tượng Predator trang trí (`img_011.jpg`), tượng đài xe bán tải bằng đá (`img_027.jpg`) $\rightarrow$ **Tuyệt đối không gán**.
   - **Tranh ảnh, hình in quảng cáo:** Chân dung trên pano / billboard bầu cử (`img_018.jpg`) $\rightarrow$ **Không gán**.
   - **Hình ảnh phản chiếu:** Hình ảnh qua gương chiếu hậu (`img_020.jpg`) $\rightarrow$ **Không gán**.
   - **Ma-nơ-canh & Đồ chơi trong tủ kính:** Ma-nơ-canh tiệm thời trang (`img_033.jpg`), bộ sưu tập mô hình ô tô/búp bê trong tủ kính (`img_038.jpg`) $\rightarrow$ **Không gán**.
   - **Vật thể trang trí xe diễu hành:** Xe Hamley's Toy Parade Dangermouse (`img_019.jpg`), vỏ xe AMC Pacer chồng trên nóc xe khác (`img_024.jpg`) $\rightarrow$ Đánh dấu `can_review=true` để nhóm thống nhất.
6. **Ngưỡng kích thước tối thiểu (Small Objects):**
   - Các đối tượng có chiều cao hoặc chiều rộng $< 20\text{px}$ ở hậu cảnh xa: Bỏ qua không gán lẻ. Nếu tụ thành đám đông thì gom vào `ignore_region`.
7. **Ảnh bị nhòe chuyển động / Mưa (`img_014.jpg`, `img_015.jpg`):**
   - Nếu mắt người vẫn nhận biết được loại phương tiện (xe ô tô qua vệt đèn pha, người đi bộ qua vệt chân chuyển động), vẽ box ôm sát vùng hình thể chính và tick `can_review=true`.

---

## 4. Danh sách ca khó phát hiện trong 40 ảnh & Đề xuất quyết định

| STT | Ảnh | Tình huống cụ thể | Mã quy tắc đề xuất | Quyết định cá nhân của tôi | Lý do kỹ thuật |
|:---:|:---|:---|:---:|:---|:---|
| 1 | `img_002.jpg` | Xe tải Renault 380 chở 8 ô tô con trên hai tầng giá sắt. | `R-01` | Gán 1 box `truck` cho xe tải lớn + 8 box `car` cho 8 xe con trên sàn. | 8 ô tô con có đầy đủ đặc tính trực quan của ô tô thật; nếu bỏ qua mô hình sẽ học thói quen miss xe khi đỗ san sát. |
| 2 | `img_004.jpg`, `img_035.jpg` | Xe tuk-tuk 3 bánh (Auto Rickshaw) tại Thái Lan / Ấn Độ. | `R-02` | Đề xuất nhóm thêm class `tuk_tuk` ở Pha 2; tạm thời tick `can_review=true`. | Không thể xếp vào `car` (vì 3 bánh, mui bạt) hay `motorcycle` (kích thước lớn, chở nhiều khách), cũng không phải `truck`. |
| 3 | `img_005.jpg`, `img_026.jpg` | Người điều khiển và người ngồi sau xe máy tại Việt Nam. | `R-03` | Tách riêng box `person` và box `motorcycle`. | Người và xe có thể tách rời; người đi bộ và người lái xe cần phân tách hành vi trong bài toán tự hành. |
| 4 | `img_006.jpg`, `img_023.jpg`, `img_039.jpg` | Xích lô du lịch tại Huế chở khách, xe đạp không động cơ. | `R-04` | Chỉ gán box `person` cho người đạp/người ngồi; không gán box xe cho xích lô. | Taxonomy v0 chỉ định nghĩa phương tiện cơ giới (`car, truck, van, motorcycle`). Xích lô không có động cơ. |
| 5 | `img_009.jpg`, `img_032.jpg` | Xe buýt lớn chở khách công cộng (Bus). | `R-05` | Đề xuất nhóm thêm class `bus` ở Pha 2. Nếu giữ v0 thì tạm xếp `truck` + tick `can_review`. | Kích thước xe buýt chở người quá lớn so với `van`, nhưng cấu trúc chở khách khác với chở hàng của `truck`. |
| 6 | `img_011.jpg`, `img_017.jpg` | Tượng đồng Wellington đội cọc tiêu và tượng Predator hóa trang bên xe máy. | `R-06` | **Không gán** box `person` cho cả 2 tượng. | Tượng là vật thể bất động nhân tạo, không có hành vi sinh học của người thật; gán sẽ gây false positive. |
| 7 | `img_018.jpg` | Biển quảng cáo (billboard) in ảnh chân dung lớn của chính trị gia. | `R-07` | **Không gán** box `person`. | Ảnh in 2D trên biển hiệu không phải người tham gia giao thông. |
| 8 | `img_020.jpg` | Hình ảnh cầu vồng và xe phản chiếu trong gương chiếu hậu ô tô. | `R-08` | **Không gán** vật thể trong gương. | Vật thể trong gương là ảnh ảo quang học, tọa độ vị trí không nằm ở không gian thực phía trước camera. |
| 9 | `img_027.jpg` | Bia tưởng niệm hình xe bán tải tạc từ đá hoa cương trong nghĩa trang. | `R-09` | **Không gán** box `car`/`truck`. | Là khối đá tạc tĩnh, không có động cơ hay khả năng di chuyển trên đường. |
| 10 | `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh thời trang trong tủ kính và tủ kính bộ sưu tập xe đồ chơi/búp bê. | `R-10` | **Không gán** box cho ma-nơ-canh và đồ chơi trong tủ. | Thuộc không gian nội thất cửa hàng, là vật vô tri; tránh hiện tượng phantom braking cho hệ thống lái. |
| 11 | `img_035.jpg` | Xe Auto Rickshaw chở bao tải chất cao ngất ngưởng gấp 2 lần xe. | `R-11` | Box xe chỉ bao trọn thân xe cơ giới; không bao khối bao tải nhô lên trên. | Khối hàng nhô quá cao làm méo mó tỉ lệ khung hình (aspect ratio) chuẩn của phương tiện. |
| 12 | `img_037.jpg` | Xe bán tải gắn nắp thùng cao chuyên dụng Master 75 của GO Transit. | `R-12` | Gán `car` (hoặc `truck` tùy thảo luận nhóm) + tick `can_review=true`. | Ranh giới giữa pickup có nắp thùng cao, xe `van` và xe tải nhỏ rất mỏng manh, cần nhóm thống nhất ở Pha 2. |

---

## 5. QC và Lịch sử phiên bản

- **Ngưỡng chất lượng tự áp dụng cho Pha 3:**
  - Tự kiểm soát $100\%$ ảnh trong danh sách phân công được gán đầy đủ.
  - Các box có chiều cao hoặc chiều rộng $< 20\text{px}$ cần soi kỹ để loại bỏ hoặc gom `ignore_region`.
  - Không để bất kỳ bounding box nào có tọa độ âm hoặc vượt biên ảnh.
  - Các ca lưỡng lự phải bật cờ `can_review = true` thay vì gán bừa.
- **Lịch sử thay đổi:**

| Phiên bản | Thay đổi chính | Cơ sở / Tình huống ảnh |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên (sơ lược 5 lớp, thuộc tính định tính "ít/nhiều"). | [guideline_v0_starter.md](file:///c:/Users/phant/K5-L2A-Day07-PhanThanhHao-2A202603165-Guideline-Taxonomy-QC/guideline/guideline_v0_starter.md) |
| `v0.1_individual_165` | Lập bảng định lượng cho thuộc tính (ngưỡng 30%, 70%); chốt nguyên tắc tách người và xe máy; bổ sung quy tắc loại trừ tượng, ảnh in, phản chiếu gương, ma-nơ-canh; đề xuất quy tắc xe chở xe và phân tích 12 ca khó cụ thể. | Khảo sát độc lập toàn bộ 40 ảnh `img_001.jpg` – `img_040.jpg` |
