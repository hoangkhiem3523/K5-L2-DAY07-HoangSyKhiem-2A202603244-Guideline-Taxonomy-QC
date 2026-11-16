# Guideline gán nhãn — v0.1 — 270

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: 270

---

## 1. Mục tiêu

- **Mô hình sử dụng**: Mô hình 2D Bounding Box Object Detection (YOLO, Faster R-CNN) cho camera quan sát giao thông đô thị và hệ thống hỗ trợ lái xe tự động (ADAS).
- **Mục đích**: Nhận diện chính xác vị trí và phân loại các đối tượng tham gia giao thông (người đi bộ, xe máy, ô tô, xe tải, xe van) trong điều kiện đường phố thực tế.
- **Rủi ro nghiêm trọng**:
  - *Bỏ sót (False Negative)*: Bỏ sót người đi bộ (`person`) hoặc người đi xe máy (`motorcycle`) có nguy cơ gây tai nạn nghiêm trọng trong thực tế.
  - *Dự đoán nhầm (False Positive)*: Gán nhầm tượng đá, ma-nơ-canh, ảnh in trên pano quảng cáo hay bóng phản chiếu thành vật thể thật khiến hệ thống phanh gấp vô căn cứ.
  - *Nhầm lẫn lớp (Class Confusion)*: Nhầm giữa ô tô con (`car`) và xe tải chở hàng (`truck`) làm sai lệch dự đoán tải trọng và làn đường ưu tiên.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (Classes)
Mỗi vật thể tham gia giao thông độc lập chỉ thuộc đúng **một lớp**. 

| Lớp (`name`) | Định nghĩa (một câu) | Ví dụ trong 40 ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `person` | Con người thật ở mọi tư thế (đứng, ngồi, đi bộ, điều khiển xe). | Người đi bộ qua đường (`img_001`, `img_014`), người lái xe máy (`img_005`, `img_026`), người đạp xích lô (`img_006`). | Ma-nơ-canh trong tủ kính (`img_033`), tượng đá (`img_017`), hình in chân dung trên pano billboard (`img_018`), búp bê đồ chơi (`img_038`). |
| `motorcycle` | Phương tiện 2–3 bánh có gắn động cơ (xe máy số, xe tay ga, xe điện, mô tô). | Xe máy lưu thông trên đường (`img_005`, `img_008`, `img_030`), xe máy chở hàng (`img_029`), mô tô có thùng phụ sidecar (`img_022`). | Xe xích lô đạp chân (`img_006`, `img_023`), xe đạp thường (không gán). |
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi phục vụ chở người. | Xe sedan, hatchback, SUV chạy trên đường phố (`img_015`, `img_024`, `img_040`). | Xe tải thùng chở hàng (`truck`), xe van kín chở hàng (`van`), mô hình ô tô đồ chơi (`img_019`, `img_038`). |
| `truck` | Xe cơ giới chuyên dùng chở hàng hóa, có thùng hàng riêng biệt hoặc đầu kéo. | Xe vận tải chở ô tô (`img_002`), xe bán tải chở hàng đầy thùng sau (`img_012`, `img_036`), xe tải trong kho bốc dỡ (`img_007`). | Xe ô tô con gia đình (`car`), xe tuk-tuk chở khách (`img_004`, `img_035`). |
| `van` | Xe ô tô thân liền kín cỡ vừa dùng chở người hoặc hàng hóa nhẹ. | Xe ô tô khoang kín thân liền, xe cứu thương cỡ nhỏ, xe van mini. | Xe ô tô con SUV (`car`), xe tải thùng rời (`truck`). |
| `ignore_region` | Vùng không gian dày đặc, quá xa, quá tối hoặc nhòe không thể tách từng vật. | Hàng trăm xe ở hậu cảnh xa tít (`img_040`), cụm bóng người mờ tối ban đêm (`img_013`). | Vật thể riêng lẻ có kích thước > 20px nhìn rõ được viền (bắt buộc phải vẽ box riêng). |

*Quy định vật ngoài taxonomy*:
- Các phương tiện như xe tuk-tuk / auto rickshaw (`img_004`, `img_035`) hoặc xe buýt lớn (`img_032`) tạm thời tick `can_review` và ghi chú đề xuất nhóm bổ sung lớp ở Pha 2.
- Xe thô sơ không động cơ (xích lô: `img_006`, `img_009`): Chỉ gán người lái và hành khách (`person`), không gán xe.

### 2.2. Thuộc tính (Attributes)

| Tên thuộc tính | Kiểu | Các mức cố định | Ý nghĩa và ngưỡng cụ thể |
|---|---|---|---|
| `occluded_level` | Select | `0`, `1`, `2`, `3` | **`0`**: Thấy rõ hoàn toàn (vật thể bị che khuất < 10%).<br>**`1`**: Che khuất nhẹ (bị che từ 10% đến 35% diện tích).<br>**`2`**: Che khuất nặng (bị che từ 35% đến 80% diện tích).<br>**`3`**: Che khuất gần như toàn bộ (> 80% diện tích nhưng vẫn nhận diện được dựa vào ngữ cảnh). |
| `truncated` | Select | `no`, `yes` | **`no`**: Vật thể nằm trọn vẹn trong khung hình.<br>**`yes`**: Một phần vật thể bị cắt ngang bởi 4 cạnh mép ảnh. |
| `can_review` | Checkbox | `false`, `true` | **`false`**: Nhãn và box đã rõ ràng, chắc chắn.<br>**`true`**: Tình huống khó/mơ hồ (ngoài taxonomy, che khuất cực đoan, góc nhìn lạ), cần đưa ra phiên họp nhóm (Calibration). |

---

## 3. Quy tắc vẽ box

- **R-01 (Bao sát phần nhìn thấy)**: Đường viền bounding box phải tiếp xúc sát mép ngoài cùng của điểm ảnh nhìn thấy (`visible bounding box`), không vẽ phỏng đoán phần khuất sau vật thể khác.
- **R-02 (Quy tắc người và xe máy)**:
  - Vẽ **01 box `person`** cho từng người ngồi trên xe (ôm từ đầu/mũ bảo hiểm đến yên xe/chân chạm xe).
  - Vẽ **01 box `motorcycle`** cho chiếc xe máy.
  - Hai box này chồng lấn (overlap) tự nhiên. Nếu xe chở 2 người (`img_026`), vẽ 2 box `person` riêng biệt và 1 box `motorcycle`.
- **R-03 (Vật thể bị cắt ở mép ảnh)**: Chỉ vẽ box bao lấy phần xuất hiện bên trong khung ảnh và đặt thuộc tính `truncated = yes`.
- **R-04 (Hàng hóa chở trên xe)**:
  - Hàng hóa đặt gọn trong lòng phương tiện: Box bao quanh cả xe và phần hàng nếu thành một khối di chuyển thống nhất (`img_012`, `img_029`).
  - Xe chở xe (`img_002`): Chỉ vẽ 1 box `truck` cho xe vận tải lớn bên dưới; các ô tô con nằm trên sàn xe chỉ là kiện hàng, không vẽ box `car`.
- **R-05 (Loại trừ đối tượng tĩnh / không phải vật thật)**:
  - Tượng điêu khắc bằng đá/đồng (`img_017`, `img_027`): **Không gán**.
  - Ảnh chân dung in trên biển quảng cáo/pano pano billboard (`img_018`): **Không gán**.
  - Ma-nơ-canh trong tủ kính (`img_033`): **Không gán**.
  - Đồ chơi mô hình thu nhỏ và búp bê trong tủ kính (`img_019`, `img_038`): **Không gán**.
  - Hình ảnh phản chiếu qua vũng nước mưa (`img_013`, `img_015`) hoặc qua gương chiếu hậu (`img_020`): **Không gán**.
- **R-06 (Ngưỡng kích thước tối thiểu)**: Bỏ qua các vật thể có chiều cao hoặc chiều rộng nhỏ hơn **20 pixels** hoặc gom cả cụm vào `ignore_region`.
- **R-07 (Xe chủ - Ego vehicle)**: Nếu nắp capo hoặc gương của xe chở camera xuất hiện ở mép dưới ảnh, không gán nhãn phương tiện cho xe chủ.

---

## 4. Ca khó và quyết định (Khảo sát từ 40 ảnh)

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe chuyên dụng chở xe (`car carrier`) đang chở 6–8 chiếc ô tô con trên sàn rơ-moóc. | `R-01`, `R-04` | Vẽ 01 box `truck` bao toàn bộ xe vận chuyển; **không vẽ box `car`** cho các xe con bên trên. | Ô tô trên sàn chở là hàng hóa tĩnh, không phải phương tiện độc lập đang tham gia giao thông. |
| `img_004`, `img_035` | Xe Tuk-tuk và Auto Rickshaw chở khách trên phố. | Ngoài taxonomy | Đánh dấu `can_review = true`, tạm gán theo quy ước nhóm hoặc đề xuất thêm lớp `tuk_tuk`. | Taxonomy v0 chưa có lớp phù hợp; gán vào `car` hay `motorcycle` đều gây sai bản chất xe. |
| `img_006`, `img_023`, `img_039` | Xích lô du lịch (Cyclo) chở khách ở phố cổ. | `R-01`, `R-02` | Vẽ box `person` cho người lái và hành khách; **không vẽ box phương tiện**. | Xích lô là xe thô sơ đạp chân, không có động cơ nên không thuộc lớp `motorcycle`. |
| `img_011` | Người mặc trang phục hóa trang Predator tham gia lễ hội mô tô. | `R-01` | Vẫn vẽ 01 box `person` ôm trọn cơ thể nhân vật hóa trang. | Bản chất là người thật mặc trang phục, có hình dáng và hành vi di chuyển trong không gian giao thông. |
| `img_013`, `img_015` | Đèn xe và bóng người phản chiếu rõ trên mặt đường ướt sau cơn mưa. | `R-05` | **Không vẽ box** cho vùng phản chiếu trên mặt đường. | Bóng phản chiếu trên vũng nước là hiện tượng quang học, không phải vật thể thật cản trở giao thông. |
| `img_017` | Tượng đá danh nhân Duke of Wellington cưỡi ngựa đội nón chóp giao thông. | `R-05` | **Không gán box** cho người và ngựa bằng đá. | Là công trình điêu khắc tĩnh, mô hình không được nhầm lẫn với người đi bộ thật. |
| `img_018` | Pano quảng cáo chính trị / thương mại in hình chân dung người rất lớn cạnh đường. | `R-05` | **Không gán box** `person` cho ảnh in 2D trên biển hiệu. | Ảnh 2D in phẳng trên biển hiệu không có nguy cơ va chạm. |
| `img_019`, `img_038` | Tủ kính trưng bày bộ sưu tập đồ chơi xe hơi và búp bê trẻ em. | `R-05`, `R-06` | **Không gán box** cho toàn bộ mô hình và búp bê trong tủ kính. | Là hàng hóa đồ chơi tĩnh trong nhà, nằm ngoài phạm vi giao thông. |
| `img_020` | Khung cảnh xe cộ và cầu vồng phản chiếu qua gương chiếu hậu ô tô. | `R-05` | **Không vẽ box** cho các đối tượng bên trong mặt gương chiếu hậu. | Vật thể trong gương là ảnh ảo phản chiếu góc sau, không tồn tại ở tọa độ thực trước mặt camera. |
| `img_022` | Mô tô phân khối lớn có gắn thùng phụ chở người bên cạnh (`sidecar`). | `R-02` | Vẽ 01 box `motorcycle` ôm trọn cả thân xe và thùng sidecar; vẽ box `person` riêng cho người ngồi trong thùng. | Xe gắn liền thành một khối phương tiện cơ giới 3 bánh di chuyển đồng bộ. |
| `img_027` | Bia tưởng niệm nghĩa trang tạc hình chiếc xe bán tải bằng đá hoa cương. | `R-05` | **Không gán box** `truck` hay `car`. | Là khối đá điêu khắc trong khuôn viên nghĩa trang, không phải xe cơ giới. |
| `img_028` | Xe bán tải chở tượng con ngựa giả trên thùng xe. | `R-04`, `R-05` | Vẽ box `truck` cho xe bán tải; tượng ngựa trên thùng xem như hàng hóa trên xe (không gán nhãn sinh vật). | Phân biệt rõ vật thật và mô hình tĩnh. |
| `img_029` | Xe máy chở hàng hóa cồng kềnh phía sau che lấp gần như toàn bộ người lái. | `R-02`, `R-04` | Vẽ 01 box `motorcycle` ôm xe và kiện hàng. **Chỉ vẽ `person`** nếu nhìn thấy phần đầu/mũ bảo hiểm. | Khi không có đặc trưng thị giác của con người, không suy đoán vẽ box để tránh phạt sai mô hình. |
| `img_032` | Xe buýt lớn chở khách và đám đông hành khách tại trạm xe buýt giờ cao điểm. | `R-01`, `R-06` | Xe buýt tick `can_review` (đề xuất thêm `bus`); đám đông quá chen chúc ở xa gộp vào `ignore_region`. | Giải quyết xung đột nhãn giữa các annotator đối với phương tiện công cộng lớn. |
| `img_033` | Ma-nơ-canh mặc áo quần thời trang đứng sau tủ kính cửa hàng mặt phố. | `R-05` | **Không gán box** `person` cho ma-nơ-canh. | Ma-nơ-canh tĩnh trong nhà không phải đối tượng người đi bộ. |
| `img_037` | Xe bán tải có gắn mui thùng kín phía sau (Pickup with canopy cap). | Phân lớp | Gán `truck` (quy ước: mọi xe bán tải dù có mui hay không mui đều tính là `truck`). | Giúp loại bỏ sự phân vân giữa `car`, `van` và `truck` giữa các thành viên. |

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC áp dụng
- **Ngưỡng đo IoU đối với các cặp box ghép đôi**:
  - Đối tượng kích thước lớn ($> 50 \text{ px}$): $\text{IoU} \ge 0.85$.
  - Đối tượng kích thước nhỏ ($\le 50 \text{ px}$): $\text{IoU} \ge 0.70$.
- **Độ chính xác phân lớp (Class Agreement)**: $\ge 95\%$.
- **Tỷ lệ lỗi trên ảnh tối đa**: $\le 2\%$.
- **Quy trình xử lý ca mơ hồ**:
  - Khi gán độc lập: Đánh dấu `can_review = true` trên box hoặc ghi vào nhật ký lỗi.
  - Họp nhóm Pha 2 / Pha 5 (Calibration): Reviewer cùng các thành viên mở trực tiếp ảnh đối chiếu, thống nhất quy tắc và ghi vào `calibration_decisions.csv`.

### 5.2. Lịch sử thay đổi (Changelog)

| Phiên bản | Thay đổi chính | Lý do (căn cứ ca nào) |
|---|---|---|
| `v0` | Khung khởi tạo chung của giảng viên. | Bản sơ khởi, chưa xử lý ca biên và che khuất. |
| `v1.0` | - Định nghĩa rõ quy tắc người và xe máy (`R-02`).<br>- Quy định loại trừ tượng (`img_017`, `img_027`), pano quảng cáo (`img_018`), ma-nơ-canh (`img_033`), đồ chơi (`img_019`, `img_038`), phản chiếu (`img_013`, `img_020`).<br>- Quy định xe chở xe (`img_002`) và hàng cồng kềnh (`img_029`).<br>- Đưa xe tuk-tuk (`img_004`, `img_035`) và xe buýt (`img_032`) vào diện `can_review`. | Kết quả khảo sát độc lập toàn diện trên 40 ảnh thực tế `img_001` – `img_040` (Pha 1). |
