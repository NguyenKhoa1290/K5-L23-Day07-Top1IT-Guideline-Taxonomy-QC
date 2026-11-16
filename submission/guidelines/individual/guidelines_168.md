# Guideline gán nhãn — bản cá nhân


- Phiên bản: …
- Ngày: …
- Mã thành viên: …

## 1. Mục tiêu

Xây dựng dữ liệu gán nhãn người và phương tiện trong ảnh đường phố bằng bounding box, nhằm hỗ trợ huấn luyện mô hình phát hiện đối tượng. Nhãn cần giúp người gán xác định nhất quán lớp và vị trí của từng đối tượng nhìn thấy trong ảnh.

## 2. Lớp và thuộc tính

### 2.1. Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `person` | Người thật xuất hiện trong ảnh, bao gồm người đi bộ, đứng, ngồi hoặc đang ở trên phương tiện. | Người đi bộ; người lái xe máy. | Tượng người, ma-nơ-canh, hình người in trên biển quảng cáo. |
| `car` | Ô tô con dùng để chở người, bao gồm các kiểu xe như sedan, hatchback và SUV. | Ô tô con trên đường; ô tô con được xe tải chở. | Xe buýt, xe tải, xe van; mô hình hoặc tượng ô tô. |
| `truck` | Phương tiện cơ giới được thiết kế để chở hàng, có thùng hàng hoặc cấu trúc chuyên chở hàng. | Xe tải thùng; xe tải chở ô tô. | Ô tô con; hàng hóa đặt trên xe. |
| `van` | Xe cơ giới thân kín cỡ nhỏ, có khoang chở người hoặc hàng liền với thân xe. | Xe van chở người hoặc hàng. | Xe buýt lớn; xe tải có ca-bin và thùng hàng tách biệt. |
| `motorcycle` | Xe máy hoặc mô tô hai bánh có động cơ. | Xe máy đang chạy hoặc đỗ. | Xe đạp không động cơ; xe ba bánh tuk-tuk. |
| `ignore_region` | Vùng ảnh được đánh dấu để bỏ qua khi không thể gán từng đối tượng một cách đáng tin cậy; đây là vùng, không phải một đối tượng. | Vùng đám đông quá nhỏ và dày đặc để phân biệt từng người. | Một người hoặc phương tiện nhìn rõ và có thể gán riêng. |

### 2.2. Thuộc tính

| Thuộc tính | Giá trị | Định nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Tỷ lệ đối tượng bị che: `0` = 0%; `1` = trên 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70%. |
| `truncated` | `no`, `yes` | Cho biết đối tượng có bị cắt bởi mép ảnh hay không: `yes` là bị cắt, `no` là không bị cắt. |
| `can_review` | tick / không tick | Đánh dấu đối tượng cần được xem lại khi người gán chưa chắc về lớp, box hoặc thuộc tính. |



## 3. Quy tắc vẽ box

Mỗi đối tượng được nhận dạng rõ có một box chữ nhật ôm sát các phần nhìn thấy; không mở rộng box để dự đoán phần nằm ngoài ảnh.
1. Gán box cho mọi người và phương tiện trong ảnh, bao gồm vật cản
2. Box ôm sát vật.
3. Chọn đúng lớp và thuộc tính.
4. Không gán vật khi không còn đủ bằng chứng thị giác
5. Không gán các vật mà không phải là vật thật và không phải là vật cản

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| 003 | Người gánh hàng rong và đằng xa nhiều xe cộ | 4 | Chỉ đánh các xe có thể nhận diện được | điền tất cả có thể gây nhiễu |
014| Ảnh người đi xe máy bị mở do tốc độ màn chập chậm|1| Đánh nhãn theo bằng chứng ảnh, cả bóng ma|Có thể nhận diện được khi thực tế xảy ra| 
027| Ảnh mô hình xe| 1|Vẫn gán nhãn| Nó vẫn là một vật cản trong thực tế
038| Ảnh mô hình đồ chơi xxe ô tô qua kính| 5|Không gán| Đồ chơi không phải xe thật 

## 5. QC và phiên bản


| Phiên bản | Thay đổi | Lý do |
|---|---|---|
|  | | |
