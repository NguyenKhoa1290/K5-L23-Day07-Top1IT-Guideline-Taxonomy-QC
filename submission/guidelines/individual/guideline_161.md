# Guideline gán nhãn — v1

- Phiên bản: v1 (nâng cấp từ v0 sau khi xem bộ 40 ảnh `img_001`–`img_040`)
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

> Các ví dụ trong tài liệu dẫn theo tên ảnh trong bộ này (ví dụ `img_021`).
> Những điểm đánh dấu **[QUYẾT ĐỊNH TẠM]** là chỗ v0 không nói gì và mình tự chọn một cách xử lý
> để mọi người gán giống nhau. Cần nhóm/giảng viên xác nhận (xem mục 7).

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

Nguyên tắc chung: **chỉ gán vật THẬT, đang hiện diện trong cảnh, thuộc 5 lớp bên dưới.**
Mô hình học từ chỗ có box và cả chỗ không có box, nên bỏ sót hay gán nhầm đều gây hại như nhau.

Bộ ảnh này chủ yếu là đường phố Việt Nam/Thái Lan: rất nhiều xe máy, tuk-tuk, người đi bộ,
ảnh đêm, ảnh mưa, ảnh mờ do chuyển động, và vài ảnh không có vật hợp lệ nào.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Gồm | Không gồm |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, SUV, MPV, taxi 4 chỗ. | Xe ở mọi màu, kể cả taxi (`img_040`), xe cổ đang trưng bày (`img_024`), xe độ/xe diễu hành chạy được có biển số (`img_019`). | Xe đồ chơi, mô hình, tượng xe. |
| `truck` | Xe tải và xe bán tải: có thùng hàng hở/kín tách rời khỏi cabin. | Pickup (`img_001`, `img_012`, `img_037`), xe tải nhỏ thùng bạt (`img_036`), xe tải thùng kín lớn (`img_007`), xe chở ô tô (`img_002`). | Xe buýt, xe khách (xem 4.6). |
| `van` | Xe thùng kín cỡ nhỏ, thân liền khối từ cabin tới đuôi (kiểu Hiace, Transit), chở người hoặc hàng. | Van khách/van hàng (`img_005`, `img_016`). | Minibus lớn (`img_009`), xe tải thùng kín có cabin riêng (`img_007`) → là `truck`. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô, **xe ba bánh nhỏ kiểu tuk-tuk / auto-rickshaw** **[QUYẾT ĐỊNH TẠM]**. | Xe máy mọi loại (`img_006`, `img_021`), Vespa (`img_016`), mô tô có sidecar (`img_022`, gộp cả sidecar vào một box), tuk-tuk (`img_004`, `img_016`, `img_035`, `img_040`). | Xe đạp, xích lô đạp (xem 4.6). |
| `person` | Người thật: người đi bộ, người lái và người ngồi sau xe máy, người ngồi/đứng trong khung cảnh (quán, vỉa hè). | Người đội mũ bảo hiểm, đeo khẩu trang, che nón lá, người mặc đồ hóa trang (`img_019`). | Tượng, mannequin, hình trên áp phích/biển quảng cáo, hình phản chiếu (xem 4.1). |
| `ignore_region` | Vùng quá đông/quá nhỏ/quá mờ để gán từng vật (xem 3.7). | Đám đông, dãy xe xếp san sát ở xa. | Không dùng cho vật rõ ràng, đứng riêng. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị **vật khác** che (xem bảng dưới). |
| `truncated` | `no`, `yes` | `yes` nếu vật bị **mép ảnh** cắt mất một phần. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại (xem mục 5). |

**Bảng `occluded_level`** (ước lượng bằng mắt phần của vật bị che, tính trên phần còn lại nằm trong ảnh):

| Mức | Phần bị che | Dấu hiệu nhận biết |
|---|---|---|
| `0` | < 10% | Thấy rõ toàn bộ. |
| `1` | 10% – 50% | Bị che ít: thấy rõ hình dạng, thiếu một phần (ví dụ xe sau lưng người đi bộ). |
| `2` | 50% – 80% | Bị che nhiều: còn nhận ra là gì nhưng thiếu đa số (ví dụ chỉ thấy đầu và vai người trong đám đông). |
| `3` | > 80%, hoặc chỉ đoán được | Chỉ lộ vài chi tiết, phải suy luận mới biết lớp. Mức 3 **luôn kèm `can_review`**. |

**`occluded_level` và `truncated` là hai thứ khác nhau:** bị vật khác che → `occluded_level`; bị mép ảnh cắt → `truncated`. Một vật có thể có cả hai.
Ví dụ: xe đỏ ở mép trái `img_015` → `truncated = yes`; người ở mép dưới `img_016` → `truncated = yes`.

## 3. Quy tắc vẽ box

### 3.1 Gán cái gì
1. Gán box cho **mọi** vật thật thuộc 5 lớp, đang hiện diện trong cảnh, **trừ** các trường hợp ở mục 3.7 và 4.
2. Mỗi vật đúng **một** box; không vẽ hai box chồng lên cùng một vật.
3. Ảnh có thể **không có vật hợp lệ nào** (`img_017`, `img_020`, `img_033`, `img_038`). Khi đó để trống, không gượng gán; ảnh vẫn tính là đã làm xong.

### 3.2 Box ôm sát thế nào
1. Box ôm sát **phần nhìn thấy** của vật: cạnh box chạm điểm ngoài cùng của vật, lệch tối đa khoảng 2 px so với ảnh gốc (không phóng to quá mức để chấm từng pixel).
2. **Không** vẽ suy đoán phần bị che (không "đoán" phần thân xe sau lưng người). Chỉ vẽ phần thấy. Phần bị che ghi bằng `occluded_level`.
3. Vật bị mép ảnh cắt: kéo box sát mép ảnh, không vẽ ra ngoài; tick `truncated = yes`.
4. Bao gồm: gương chiếu hậu, mũ bảo hiểm, tay chân của người, thùng xe, bạt phủ xe, tay lái. Không bao gồm: bóng đổ, vệt phản chiếu dưới đường ướt (`img_015`, `img_031`), cột đèn/biển báo đứng sau.
5. **Vật mang vác/hàng chất nhô ra ngoài thân xe hoặc người không tính vào box**: cây dài trên xe máy (`img_029`), gánh hàng (`img_003`), hàng bao chất cao trên tuk-tuk (`img_035`). Box chỉ ôm xe/người. Hàng nằm gọn trong thùng xe tải/pickup (`img_012`) thì nằm trong box xe, không tách.

### 3.3 Người và xe máy
1. Người đi xe máy: **hai (hoặc nhiều) box riêng**: một `motorcycle` ôm thân xe, một `person` cho mỗi người ngồi trên xe (cả người lái và người ngồi sau). Box `person` và `motorcycle` được phép chồng lên nhau nhiều.
2. Ba người trên một xe (`img_005`) → 1 `motorcycle` + 3 `person`.
3. Người dắt xe/đứng cạnh xe máy: vẫn là 1 `person` + 1 `motorcycle` (`img_010`).
4. Người ngồi trên xe máy đang đậu, xe quay lưng (`img_030`): vẫn gán cả hai.
5. Xe máy không người: chỉ có `motorcycle`.

### 3.4 Người trong/trên xe khác
1. Người ngồi ở chỗ **hở, thấy rõ ít nhất đầu và vai** (tuk-tuk mở, thùng pickup, ngồi trên xe): gán `person` (`img_004`, `img_019`).
2. Người ngồi **trong ô tô/van kín, chỉ thấy mờ qua kính**: **không** gán.
3. Người đi xe đạp/xích lô đạp: gán `person` cho người; **không** gán box cho xe đạp/xích lô (vì chưa có lớp tương ứng, xem 4.6) (`img_003`, `img_006`, `img_023`).

### 3.5 Xe kéo, xe chở xe
1. Xe đầu kéo + rơ-moóc nối liền: **một** box `truck` ôm cả cụm (`img_002`).
2. Ô tô thật đang nằm trên xe chở xe: **vẫn gán `car` riêng cho từng xe nhìn thấy được**, bị che ít/nhiều thì ghi `occluded_level`. Xe chồng khuất gần hết (`occluded_level = 3`) kèm `can_review`.
3. Mô tô có sidecar: một box `motorcycle` ôm cả xe + sidecar (`img_022`).

### 3.6 Ảnh khó về chất lượng
1. **Ảnh đêm / thiếu sáng / mưa** (`img_007`, `img_013`, `img_031`): vẫn gán những vật mà bạn nhận ra được bằng mắt thường. Vật chỉ là đốm sáng, không nhận ra lớp → không gán (hoặc dùng `ignore_region`, 3.7).
2. **Mờ do chuyển động** (`img_014`, `img_015`): gán nếu **vẫn nhận ra lớp**; ranh giới box lấy theo phần thân chính, không kéo theo vệt mờ. Quá mờ để nhận ra lớp → để vào `ignore_region`.
3. **Ảnh đen trắng** (`img_022`, `img_033`): gán bình thường như ảnh màu.
4. **Vật rất nhỏ**: nếu cạnh ngắn của box dưới khoảng 10 px (trên ảnh hiển thị đủ khung) thì **không gán từng vật**; nếu có nhiều vật như vậy tụ lại, dùng `ignore_region` (`img_013`, `img_025`).

### 3.7 Khi nào dùng `ignore_region`
Dùng khi **nhiều vật dính sát, không thể tách thành từng box đáng tin**, theo bất kỳ điều kiện nào sau:
- đám đông (người) mà phần lớn là đầu và vai chồng lên nhau (`img_019`, `img_032`);
- dãy xe máy dày đặc ở xa/giữa nền, phần lớn `occluded_level = 3` (`img_021` phần xa, `img_030` phần xa, `img_008` phần nền);
- cụm vật quá nhỏ hoặc quá mờ, theo 3.6.

Quy tắc kèm theo:
1. Vật **thấy rõ ở gần** (≥ ~30% hiện rõ, đủ lớn) **vẫn gán riêng**; chỉ phần còn lại phía sau mới gom vào `ignore_region`.
2. Vẽ `ignore_region` ôm cả cụm, **không trùm lên** các box đã gán riêng.
3. Không dùng `ignore_region` để "lười" cho những vật lẻ, rõ ràng.
4. Một vùng đông chia thành **một** vùng chung nếu liền nhau; chia làm nhiều nếu bị tách bởi khoảng trống rõ.

## 4. Ca khó

Mỗi ca ghi: **tình huống → cách làm → ảnh minh hoạ.**

### 4.1 Vật "giống thật" nhưng không phải vật thật → **KHÔNG gán**
| Tình huống | Cách làm | Ảnh |
|---|---|---|
| Tượng, tượng người, tượng ngựa | Không gán `person`. | `img_017`, `img_028` (ngựa trên thùng xe) |
| Mannequin ở cửa kính | Không gán `person`. | `img_033` |
| Hình người in trên biển quảng cáo/áp phích | Không gán. | `img_018`, `img_030` (áp phích phía sau) |
| Xe đồ chơi, mô hình trong tủ kính | Không gán. | `img_038` |
| Xe làm bằng đá/bia mộ hình xe | Không gán. | `img_027` |
| Tác phẩm điêu khắc hình xe máy / sinh vật bằng kim loại | Không gán (tick `can_review` nếu bạn không chắc đó là xe thật hay tượng). | `img_011` |
| Tranh tường/pano vẽ tàu, xe, người | Không gán; chỉ gán người/xe **thật** đi phía trước. | `img_026` |
| Hình phản chiếu (gương chiếu hậu, kính cửa hàng) | Không gán phản chiếu. | `img_020`, `img_033` |
| Xe mô hình nhỏ đặt trên nóc xe thật | Không gán xe mô hình; chỉ gán xe thật bên dưới. Nếu không chắc là đồ trưng bày hay xe thật → `can_review`. | `img_024` |
| Nón lá, cọc tiêu, rào chắn | Không phải lớp nào. | `img_032`, `img_004` |

### 4.2 Vật thật nhưng "lạ" → vẫn gán
| Tình huống | Cách làm | Ảnh |
|---|---|---|
| Xe cổ đang trưng bày trong bảo tàng/triển lãm | Gán `car` (xe thật). | `img_024`, `img_028` (gán `truck` cho pickup cổ) |
| Xe diễu hành/lễ hội chạy được, có biển số | Gán `car`; người ngồi lộ thiên gán `person`; linh vật (nhân vật hoạt hình, mascot) thì không. | `img_019` |
| Người mặc đồ hóa trang vẫn là người | Gán `person`. | `img_019` |
| Ảnh chụp lại cảnh cũ (ảnh đen trắng cổ) | Gán như bình thường. | `img_022` |

### 4.3 Phân biệt `car` / `van` / `truck`
Hỏi theo thứ tự:
1. Có **thùng hàng/thùng xe tách riêng khỏi cabin**, hoặc đầu kéo, hoặc là xe bán tải? → `truck`.
2. Thân xe **liền khối cabin đến đuôi**, nóc cao, cỡ nhỏ, kiểu van khách/van hàng? → `van`.
3. Xe con thông thường (kể cả SUV/hatchback/MPV/taxi 4 chỗ)? → `car`.
4. Vẫn khó phân biệt giữa `van` và `car`/`truck` → chọn lớp gần nhất và tick `can_review`.

### 4.4 Tuk-tuk / auto-rickshaw → `motorcycle` **[QUYẾT ĐỊNH TẠM]**
Bộ lớp chưa có xe ba bánh. Do xe ba bánh nhỏ có phần đầu là xe máy, tạm gán là `motorcycle`
(`img_004`, `img_016`, `img_035`, `img_040`). Người lái và khách ngồi lộ thiên gán `person` riêng.
Hàng chất ngoài thân xe không tính vào box (3.2.5). Cần xác nhận với nhóm.

### 4.5 Vật bị che, vật bị cắt, vật chồng chéo
1. Người ở sau xe đang đậu, chỉ thấy nửa trên: box ôm phần nửa trên thấy được, `occluded_level` theo bảng.
2. Xe sau hàng rào/lưới (`img_018`): vẫn gán, `occluded_level = 1` hoặc `2` tuỳ mức che của rào.
3. Mảng lớn của xe ở sát mép ảnh (`img_015`, `img_005`, `img_040`): gán, `truncated = yes`.
4. Vật bị che hơn 80% (chỉ thấy tay lái hoặc một phần bánh): hoặc gán `occluded_level = 3` kèm `can_review`, hoặc bỏ qua nếu **không chắc chắn** vật đó thuộc lớp nào.

### 4.6 Phương tiện/vật **ngoài** 5 lớp → KHÔNG gán **[QUYẾT ĐỊNH TẠM]**
Các thứ sau chưa có lớp riêng trong v1, nên **không vẽ box**:
- **Xe buýt, minibus, xe khách lớn** (`img_009`, `img_031`, `img_037` nền trái, `img_040` nền).
- **Xe đạp, xích lô đạp** (`img_003`, `img_006`, `img_023`). Người đi vẫn gán `person`.
- Xe nôi, xe đẩy, xe kéo tay, xe lăn.

Hậu quả cần biết: mô hình sẽ học xe buýt/xe đạp là nền. Nếu muốn mô hình phát hiện chúng, phải thêm lớp mới (`bus`, `bicycle`) và nâng lên phiên bản mới (mục 7).

### 4.7 Quy tắc ưu tiên khi hai quy tắc cùng áp dụng
1. Ca ở mục 4.1 (không thật) ưu tiên hơn mọi quy tắc khác: không thật → không gán.
2. Vật thật nhưng không chắc **lớp nào** trong 5 lớp → chọn lớp gần nhất và tick `can_review` (không bỏ sót).
3. Không chắc vật đó có **phải vật thật** hay không (tượng/đồ chơi/xe trưng bày) → tick `can_review` nếu bạn quyết định gán; nếu nghiêng về "không thật" thì không gán.

## 5. QC và phiên bản

### Ca mơ hồ
- Ca mơ hồ: tick `can_review`. Dùng khi: (a) không chắc lớp (`van`/`car`/`truck`), (b) không chắc có phải vật thật không (tượng/đồ chơi/xe trưng bày), (c) `occluded_level = 3`, (d) vật rất nhỏ/rất mờ nhưng bạn vẫn quyết định gán.
- Không tick `can_review` cho vật rõ ràng; tick quá nhiều làm mất tác dụng. Mục tiêu: `can_review` dưới khoảng 10% tổng số box.

### Tự kiểm trước khi nộp mỗi ảnh
- [ ] Mọi người và phương tiện **thật, thuộc 5 lớp** đều có box (người trên xe máy có đủ box người + box xe).
- [ ] Không có box cho tượng, mannequin, đồ chơi, áp phích, phản chiếu, xe buýt, xe đạp.
- [ ] Mỗi vật đúng một box; không box trùng.
- [ ] Box ôm sát phần thấy; không kéo theo hàng nhô ra, bóng, phản chiếu.
- [ ] `occluded_level` và `truncated` được điền cho **mọi** box (không để mặc định bừa).
- [ ] `ignore_region` không trùm lên box đã gán; không dùng cho vật lẻ rõ ràng.
- [ ] Ca mơ hồ đã tick `can_review`.

### Cross-check giữa người gán
- Chọn ngẫu nhiên khoảng 10 ảnh để người thứ hai gán độc lập, so sánh: số box, lớp, IoU (box trùng khi IoU ≥ 0.7).
- Mọi chỗ **lệch lớp** hoặc **một người gán – một người bỏ** là bằng chứng guideline mơ hồ → phải sửa guideline, không chỉ sửa nhãn.

### Phiên bản
- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Sửa chính tả/lời giải thích không đổi cách gán → có thể giữ nguyên phiên bản (ghi chú vào nhật ký).
- Ghi nhật ký thay đổi ở mục 6. Khi đổi guideline, **gán lại** (hoặc rà lại) các ảnh đã làm theo phiên bản cũ bị ảnh hưởng, và ghi phiên bản đã dùng cho mỗi ảnh.

## 6. Nhật ký thay đổi

| Phiên bản | Thay đổi | Lý do (ảnh minh hoạ) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v1 | Làm rõ định nghĩa 5 lớp, bảng `occluded_level` theo %, phân biệt `occluded` và `truncated`. | v0 mơ hồ; nhiều xe bị che/cắt (`img_005`, `img_015`, `img_021`). |
| v1 | Thêm quy tắc người + xe máy (box riêng, được chồng lên nhau); người trong xe kín/xe hở. | `img_005`, `img_010`, `img_030`. |
| v1 | Thêm quy tắc vẽ box: phần thấy, không tính hàng nhô ra, không tính bóng/phản chiếu. | `img_029`, `img_035`, `img_015`. |
| v1 | Thêm quy tắc `ignore_region`: khi nào dùng, không trùm lên box đã gán. | `img_019`, `img_021`, `img_032`. |
| v1 | Thêm 4.1: không gán tượng, mannequin, đồ chơi, áp phích, tranh tường, phản chiếu. | `img_017`, `img_033`, `img_038`, `img_018`, `img_020`, `img_027`. |
| v1 | **[QUYẾT ĐỊNH TẠM]** tuk-tuk → `motorcycle`; xe buýt và xe đạp → không gán. | `img_004`, `img_035`, `img_009`, `img_023`. |
| v1 | Thêm ngưỡng vật nhỏ (~10 px) và quy tắc ảnh đêm/mờ/đen trắng. | `img_013`, `img_014`, `img_022`, `img_031`. |

## 7. Câu hỏi còn mở (cần nhóm/giảng viên chốt)

1. **Tuk-tuk / xe ba bánh** có được coi là `motorcycle` không, hay cần lớp mới? (hiện: `motorcycle`)
2. **Xe buýt/minibus** và **xe đạp/xích lô**: để trống (nền) hay thêm lớp `bus`, `bicycle`? Nếu thêm lớp → nâng v2 và rà lại ảnh `img_003`, `img_006`, `img_009`, `img_023`, `img_031`, `img_037`.
3. Xe thật **đang trưng bày** (bảo tàng, `img_024`) có tính là phương tiện cần phát hiện không? (hiện: có)
4. Xe diễu hành có linh vật/trang trí quá mức (`img_019`) có coi là `car` không? (hiện: có)
5. Ngưỡng vật nhỏ ~10 px và ngưỡng che có hợp lý theo độ phân giải thực tế của ảnh đưa vào mô hình không?
