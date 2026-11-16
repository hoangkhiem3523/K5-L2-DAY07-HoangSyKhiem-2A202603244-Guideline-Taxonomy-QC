# Guideline gán nhãn — v1.0 — Nhom01

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết / duyệt: 244, 270, 272, 275, 281

---

## 1. Mục tiêu và bối cảnh

Tập dữ liệu bounding box 2D đường phố dùng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện **người và phương tiện giao thông** (cho camera giao thông thông minh CCTV và hệ thống hỗ trợ lái xe nâng cao ADAS).
- **Rủi ro cốt tử:**
  - Bỏ sót (False Negative) người đi bộ hoặc xe máy đang lưu thông gây nguy hiểm nghiêm trọng đến an toàn giao thông.
  - Nhận diện nhầm (False Positive) các vật thể tĩnh phi tri giác (tượng điêu khắc đá/đồng, áp phích quảng cáo, ma-nơ-canh trong tiệm thời trang, bóng phản chiếu qua gương) gây lỗi kích hoạt phanh giả (phantom braking).
  - Bounding box lỏng lẻo (IoU thấp) làm lệch ước tính vị trí 3D và quỹ đạo xe.

---

## 2. Lớp và thuộc tính

### 2.1 Định nghĩa lớp đối tượng

Tập taxonomy gồm 5 lớp đối tượng giao thông thực tế và 1 lớp vùng đặc biệt (`ignore_region`). Mỗi đối tượng thuộc đúng một lớp.

| Lớp | Định nghĩa chuẩn | Ví dụ | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con 4–7 chỗ ngồi (sedan, hatchback, SUV, crossover, coupe, convertible). | Xe taxi 4 chỗ, xe cá nhân, SUV đô thị. | Xe tải thùng rời (`truck`), xe van kín (`van`), mô hình đồ chơi (`img_038`). |
| `truck` | Xe cơ giới chuyên dụng chở hàng hóa nặng, có kết cấu cabin tách rời thùng chở hàng, xe tải sàn hở, xe bồn, xe đầu kéo container, xe tải chở ô tô con. | Xe tải 2 trục, xe container, xe chở ô tô (`img_002`), xe bán tải (pickup truck có thùng hở chở hàng) (`img_012`, `img_028`). | Xe van thùng kín chở hàng nhẹ (`van`), xe SUV (`car`), tượng đá tạc hình xe tải (`img_027`). |
| `van` | Xe tải nhẹ hoặc xe vận chuyển có thùng kín liền khối với cabin (minivan, panel van). | Ford Transit, Mercedes Sprinter, xe giao hàng thùng kín liền thân (`img_036`). | Xe tải thùng rời (`truck`), xe buýt lớn (ngoài taxonomy -> `ignore_region`). |
| `motorcycle` | Phương tiện cơ giới 2–3 bánh gắn động cơ (xe số, xe ga, mô tô phân khối lớn, xe gắn máy có thùng phụ sidecar). | Xe số, xe ga, xe mô tô phân khối lớn (`img_005`, `img_011`, `img_022`). | Xe đạp thường, xe xích lô đạp (`img_006`), xe ba gác / tuk-tuk (`img_004`, `img_035`). |
| `person` | Con người bằng xương bằng thịt đang đi bộ, đứng, ngồi trên vỉa hè hoặc đang điều khiển/ngồi trên phương tiện giao thông hở. | Người đi bộ (`img_014`), người lái và hành khách trên xe máy (`img_005`, `img_026`), người lái/khách trên xích lô (`img_006`, `img_009`), người mặc trang phục cosplay có người thật bên trong (`img_011`). | Tượng người điêu khắc (`img_017`), ảnh người in trên biển quảng cáo billboard (`img_018`), ma-nơ-canh trưng bày (`img_033`), bóng phản chiếu trong gương (`img_020`). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu nơi không yêu cầu gán chi tiết từng vật thể riêng lẻ; bỏ sót ở đây không tính là lỗi. | Đám đông người/xe quá xa ở hậu cảnh (chiều cao vật < 20px), vùng phản chiếu bóng mờ trên kính xe, phương tiện ngoài taxonomy nhưng chiếm diện tích lớn gây nhiễu (xe xích lô, xe buýt lớn, xe tuk-tuk). | Vật thể rõ ràng ở tiền cảnh hoặc trung cảnh (chiều cao >= 20px) — bắt buộc phải vẽ box cụ thể. |

### 2.2 Thuộc tính đối tượng (Attributes)

Mọi box (ngoại trừ lớp `ignore_region`) bắt buộc phải gán đủ 3 thuộc tính:

1. **`occluded_level` (Mức độ che khuất):**
   - **`0`**: Không bị che (0% diện tích bị che khuất).
   - **`1`**: Che nhẹ (>0% đến 30% diện tích bị che khuất bởi cột điện, cành cây, vật thể khác).
   - **`2`**: Che trung bình / nặng (>30% đến 80% diện tích bị che khuất).
   - **`3`**: Che gần như toàn bộ (>80% đến 95% diện tích bị che khuất nhưng vẫn nhận diện được qua ngữ cảnh).
2. **`truncated` (Cắt bởi viền ảnh):**
   - **`no`**: Vật thể nằm trọn vẹn trong ảnh.
   - **`yes`**: Ít nhất một cạnh của vật thể bị cắt bởi mép biên của khung hình ảnh.
3. **`can_review` (Cần kiểm tra lại):**
   - **`false`**: Tự tin, rõ ràng.
   - **`true`**: Nghi ngờ, ranh giới mờ, cần đưa ra phiên Calibration thảo luận.

---

## 3. Bộ quy tắc vẽ box thống nhất (Rules)

- **R-01 (Ngưỡng kích thước tối thiểu):** Chiều cao cạnh đứng tối thiểu của bounding box phải đạt **>= 20 pixels**. Các đối tượng nhỏ hơn 20px ở hậu cảnh xa không gán box đơn lẻ; nếu nằm trong khu vực đông đúc thì gom vào một box `ignore_region`.
- **R-02 (Tách riêng người và xe máy):**
  - Mọi người ngồi trên xe máy (`img_005`, `img_026`): **Bắt buộc vẽ thành các box độc lập**.
  - Mỗi người ngồi trên xe được gán 1 box `person` ôm trọn cơ thể người (từ đỉnh đầu/mũ bảo hiểm xuống tới bàn chân/mông tiếp xúc yên xe).
  - Chiếc xe máy được gán 1 box `motorcycle` ôm trọn kết cấu khung xe và 2 bánh xe (không kéo trùm lên đầu người lái).
  - Nếu xe chở 2 người (`img_026`): Gồm 2 box `person` (người lái và người ngồi sau) + 1 box `motorcycle`.
- **R-03 (Người trong cabin kín):** Người ngồi trong cabin ô tô kín (`car`, `truck`, `van`) nhìn qua lớp kính trong suốt **KHÔNG** gán box `person`. Chỉ gán `person` nếu xe mui trần mở hoàn toàn hoặc người nhoài người ra ngoài cửa sổ thấy rõ >50% thân trên.
- **R-04 (Xe chở xe - Car carrier `img_002`):**
  - Xe tải chở hàng được gán 1 box `truck` bao toàn bộ đầu kéo và sàn chở hàng.
  - Từng chiếc ô tô con được cố định trên sàn xe tải vẫn được gán box `car` riêng rẽ (set `occluded_level` tương ứng).
- **R-05 (Vật thể tĩnh phi tri giác):**
  - Tượng đài người/ngựa (`img_017`): KHÔNG gán `person`.
  - Hình in 2D trên biển quảng cáo/billboard (`img_018`): KHÔNG gán `person`.
  - Ma-nơ-canh trong tiệm thời trang (`img_033`): KHÔNG gán `person`.
  - Bia mộ/tượng đá tạc hình xe bán tải (`img_027`): KHÔNG gán `truck`/`car`.
  - Tượng ngựa trang trí đặt trên thùng xe (`img_028`): Box `truck` ôm xe, tượng ngựa không gán nhãn.
  - Mô hình ô tô đồ chơi, búp bê trong tủ kính (`img_038`): KHÔNG gán.
  - Người mặc trang phục hóa trang cosplay (`img_011`): Gán `person`.
- **R-06 (Hình phản chiếu qua gương/kính `img_020`):** Tuyệt đối không gán box cho các hình phản chiếu ảo trong gương chiếu hậu, gương cầu hoặc vũng nước trên mặt đường.
- **R-07 (Xe máy chở hàng cồng kềnh `img_029`):** Box `motorcycle` ôm sát thân xe máy và giá chở hàng gắn liền với cấu trúc vận hành. Không kéo box phình to bao trọn kiện hàng quá khổ làm loãng IoU.
- **R-08 (Phương tiện ngoài Taxonomy):**
  - Xe xích lô (`img_006`, `img_009`, `img_023`, `img_034`, `img_039`): Người lái và hành khách được gán box `person`. Khung xe xích lô không gán vào lớp phương tiện; khoanh `ignore_region` cho khung xe nếu chiếm diện tích lớn.
  - Xe tuk-tuk / auto-rickshaw (`img_004`, `img_035`): Khoanh `ignore_region` cho thân xe và tick `can_review`. Người ngồi nhô ra ngoài vẫn gán `person`.

---

## 4. Bảng ca khó mẫu cho toàn nhóm tham chiếu

| Ảnh | Tình huống | Rule ID | Quyết định thống nhất | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải đầu kéo chở nhiều ô tô con | R-04 | 1 box `truck` bao quanh xe tải; các ô tô con trên sàn gán từng box `car`. | Đảm bảo tính độc lập của đối tượng ô tô con phục vụ nhận diện đa lớp. |
| `img_004.jpg` | Hai xe tuk-tuk đỗ trên vỉa hè | R-08 | Khoanh `ignore_region` cho thân xe tuk-tuk. | Taxonomy v1.0 chưa có lớp cho xe 3 bánh; tránh gán sai vào car/motorcycle. |
| `img_006.jpg` | Đoàn xích lô chở khách du lịch | R-02, R-08 | Gán box `person` cho từng người; khung xích lô không gán phương tiện. | Người là thực thể sống cần nhận diện; xe xích lô không thuộc taxonomy. |
| `img_011.jpg` | Nhân vật Predator đứng cạnh dàn xe mô tô | R-05 | Gán 1 box `person` cho nhân vật; gán từng box `motorcycle` cho xe máy. | Người hóa trang (cosplay) có người thật bên trong, camera phải nhận diện là người để đảm bảo an toàn. |
| `img_012.jpg` | Xe bán tải (pickup) chở đầy rau củ | R-01, R-07 | Gán lớp `truck`, box ôm toàn bộ cabin và thùng chở hàng. | Thùng hở chở hàng đặc trưng cho dòng xe tải nhẹ (`truck`). |
| `img_017.jpg` | Tượng đài Công tước Wellington đội cọc tiêu | R-05 | KHÔNG gán box `person` cho tượng người/ngựa. | Tượng điêu khắc tĩnh, gán sẽ kích hoạt phanh giả. |
| `img_018.jpg` | Biển quảng cáo billboard in ảnh chân dung | R-05 | KHÔNG gán box `person`. | Ảnh in 2D trên biển hiệu quảng cáo, không phải người thật. |
| `img_020.jpg` | Cầu vồng và vệt xe phản chiếu trong gương xe | R-06 | KHÔNG gán box vào mặt gương chiếu hậu. | Ảnh ảo phản chiếu không có tọa độ vật lý trước xe. |
| `img_022.jpg` | Xe mô tô cổ có thùng phụ sidecar | R-02 | Gán 1 box `motorcycle` bao toàn bộ xe và thùng phụ; 2 box `person`. | Thùng phụ gắn liền kết cấu xe, tạo thành phương tiện cơ giới 3 bánh. |
| `img_026.jpg` | Cặp đôi cùng ngồi trên một xe máy | R-02 | Gán 2 box `person` độc lập và 1 box `motorcycle` cho xe máy. | Phân tách ranh giới rõ ràng từng cá nhân và phương tiện. |
| `img_027.jpg` | Bia mộ đá điêu khắc hình xe bán tải | R-05 | KHÔNG gán `truck`/`car`. | Tượng điêu khắc đá cố định trong nghĩa trang. |
| `img_028.jpg` | Xe bán tải chở tượng con ngựa lớn trên thùng | R-05 | Gán 1 box `truck` cho xe tải; tượng con ngựa không gán. | Tượng ngựa là hàng hóa trang trí phi sinh học. |
| `img_029.jpg` | Xe máy chở kiện hàng cồng kềnh | R-07 | Gán 1 box `person` cho người lái; box `motorcycle` ôm sát thân xe máy. | Giữ IoU chuẩn của xe máy, không kéo theo kiện hàng cồng kềnh. |
| `img_033.jpg` | Ma-nơ-canh trưng bày váy trong tủ kính | R-05 | KHÔNG gán box `person`. | Ma-nơ-canh là vật thể tĩnh nhân tạo trong nhà. |
| `img_035.jpg` | Xe Auto Rickshaw chở quá tải | R-08 | Khoanh `ignore_region` cho thân xe; gán box `person` cho hành khách. | Xe 3 bánh ngoài taxonomy, gán ignore_region để tránh phạt model. |
| `img_038.jpg` | Tủ kính đồ chơi trưng bày mô hình ô tô | R-05 | KHÔNG gán box `car` hay `person`. | Mô hình đồ chơi thu nhỏ tĩnh trong shop. |

---

## 5. QC và cam kết phiên bản

- Cả nhóm cam kết tuân thủ nghiêm ngặt Guideline v1.0 trong suốt quá trình gán nhãn độc lập ở Pha 3.
- Không tự ý sửa đổi quy tắc trong khi gán độc lập. Mọi trường hợp nghi vấn bắt buộc đánh dấu `can_review: true` để đưa vào danh mục xem xét hiệu chuẩn (Calibration) ở Pha 5.
