# Guideline gán nhãn — v1.1 — Sau Calibration

- Phiên bản: v1.1 (Bản hiệu chuẩn chính thức sau phiên Calibration)
- Ngày: 2026-11-16
- Nhóm: Nhóm 1 - AI20K K5 L2 (Các thành viên: 244, 270, 272, 275, 281)
- Người điều phối / biên tập: Thành viên 244 (Hoàng Sỹ Khiêm)

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
- **R-06 (Hình phản chiếu qua gương/kính `img_020`):** Tuyệt đối không gán box cho các hình phản chiếu ảo trong gương chiếu hậu, gương cầu hoặc vũng nước trên mặt đường. (Ca C011, C012: Thống nhất xóa bỏ box xe phản chiếu trong gương).
- **R-07 (Xe máy chở hàng cồng kềnh `img_029`):** Box `motorcycle` ôm sát thân xe máy và giá chở hàng gắn liền với cấu trúc vận hành. Không kéo box phình to bao trọn kiện hàng quá khổ làm loãng IoU.
- **R-08 (Phương tiện ngoài Taxonomy):**
  - Xe xích lô (`img_006`, `img_009`, `img_023`, `img_034`, `img_039`): Người lái và hành khách được gán box `person`. Khung xe xích lô không gán vào lớp phương tiện; khoanh `ignore_region` cho khung xe nếu chiếm diện tích lớn.
  - Xe tuk-tuk / auto-rickshaw (`img_004`, `img_035`): Khoanh `ignore_region` cho thân xe và tick `can_review`. Người ngồi nhô ra ngoài vẫn gán `person`.

---

## 4. Bảng ca khó mẫu và hiệu chuẩn (Sau Calibration)

| Ảnh | Tình huống | Rule ID | Quyết định hiệu chuẩn | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải đầu kéo chở nhiều ô tô con | R-04 | 1 box `truck` bao quanh xe tải; các ô tô con trên sàn gán từng box `car`. | Đảm bảo tính độc lập của đối tượng ô tô con phục vụ nhận diện đa lớp. |
| `img_003.jpg` | Ca C001 (IoU người đi bộ lệch nhẹ) | R-02 | Lấy box trung bình có độ ôm sát cao hơn giữa hai người gán. | Nâng cao IoU chuẩn cho người đi bộ. |
| `img_008.jpg` | Ca C002, C003 (IoU người ban đêm) | R-02 | Chốt box ôm sát thân người nhìn thấy dưới ánh đèn. | Chuẩn hóa tọa độ theo box nhìn rõ nhất. |
| `img_011.jpg` | Ca C004-C010 (Người xem lễ hội và cosplay) | R-05 | Chốt box ôm trọn cơ thể người tham quan và nhân vật cosplay. | Đạt độ đồng thuận tuyệt đối về bounding box. |
| `img_020.jpg` | Ca C011, C012 (Hình xe phản chiếu trong gương) | R-06 | XÓA BỎ toàn bộ box phản chiếu trong gương xe. | Tuân thủ tuyệt đối quy tắc R-06 về loại bỏ ảnh ảo. |
| `img_031.jpg` | Ca C013-C016 (Người đi bộ qua đường ban đêm) | R-02 | Thống nhất tọa độ box trung bình ôm sát thân người. | Đảm bảo IoU >= 0.85. |
| `img_034.jpg` | Ca C017 (Người qua đường) | R-02 | Chốt box ôm sát người đi bộ sang đường. | Thống nhất ranh giới người đi bộ. |

---

## 5. QC và cam kết phiên bản

- Phiên bản v1.1 được thông qua sau khi giải quyết trọn vẹn 17 ca bất đồng tại phiên họp Calibration ngày 2026-11-16.
- Toàn bộ nhãn cuối (`labels_calibrated.json`) đã được cập nhật đồng bộ với bộ quy tắc này.
