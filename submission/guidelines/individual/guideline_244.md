# Guideline gán nhãn — v0.1 — 244

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: 244

---

## 1. Mục tiêu

Tập dữ liệu bounding box này dùng để huấn luyện và đánh giá mô hình thị giác máy tính 2D phát hiện **người và phương tiện giao thông** trên đường phố phục vụ hệ thống camera giám sát thông minh (CCTV) và xe tự lái (ADAS).
- **Rủi ro nguy hiểm nếu sai sót:**
  - Bỏ sót (False Negative) người đi bộ hoặc xe máy đang lưu thông trong điều kiện giao thông hỗn hợp sẽ dẫn đến nguy cơ va chạm nghiêm trọng.
  - Nhận diện nhầm (False Positive) các vật thể phi tri giác (tượng điêu khắc, ảnh in trên pano/billboard, ma-nơ-canh trong shop, hình phản chiếu qua gương/kính) thành người hoặc xe thật sẽ kích hoạt hiện tượng "phanh ma" (phantom braking) làm gián đoạn dòng xe hoặc gây tai nạn liên hoàn.
  - Sai lệch vị trí ranh giới box (IoU thấp) làm suy giảm độ chính xác ước lượng khoảng cách và quỹ đạo di chuyển của vật thể.

---

## 2. Lớp và thuộc tính

### 2.1 Định nghĩa các lớp đối tượng

Tập taxonomy bao gồm 5 lớp đối tượng thực và 1 lớp vùng đặc biệt (`ignore_region`). Mỗi đối tượng chỉ được gán duy nhất 1 lớp.

| Lớp | Định nghĩa chuẩn | Ví dụ tiêu biểu | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `car` | Ô tô con từ 4 đến 7 chỗ ngồi, bao gồm các dòng sedan, hatchback, SUV, crossover, coupe, xe mui trần, xe bán tải thân liền cỡ nhỏ. | Xe taxi 4 chỗ, xe gia đình 7 chỗ, SUV, xe cảnh sát tuần tra cỡ nhỏ. | Xe tải chở hàng có thùng rời (`truck`), xe van chở hàng kín (`van`), mô hình ô tô đồ chơi (`img_038`). |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng hóa nặng, xe tải có cabin tách biệt với thùng hàng hoặc sàn chở hàng, xe bồn, xe đầu kéo, xe chuyên dụng cứu hộ. | Xe tải 2 trục trở lên, xe đầu kéo container, xe tải chở ô tô (`img_002`), xe bán tải thùng hở chở nông sản (`img_012`, `img_028`). | Xe van thùng kín chở hàng nhẹ (`van`), xe ô tô con SUV 7 chỗ (`car`), tượng đài tưởng niệm hình xe tải (`img_027`). |
| `van` | Xe tải nhẹ hoặc xe chở khách/hàng có thùng kín liền khối với cabin (minivan, panel van). | Ford Transit, Mercedes Sprinter, xe giao hàng bưu chính thùng kín liền thân (`img_036`). | Xe tải thùng rời khung hở (`truck`), xe buýt cỡ lớn (ngoài taxonomy -> `ignore_region`). |
| `motorcycle` | Phương tiện cơ giới hai bánh hoặc ba bánh có gắn động cơ (xe máy, mô tô phân khối lớn, xe tay ga, xe máy kèm thùng phụ sidecar). | Xe số Honda Wave, xe tay ga Vespa, mô tô phân khối lớn (`img_005`, `img_011`, `img_022`). | Xe đạp thường, xe xích lô đạp (`img_006` -> không gán lớp xe), xe ba gác tự chế/tuk-tuk (`img_004`, `img_035` -> xem quy tắc ngoại lệ). |
| `person` | Con người bằng xương bằng thịt đang đứng, đi bộ, chạy, ngồi trên vỉa hè hoặc đang trực tiếp điều khiển/ngồi trên các phương tiện giao thông hở. | Người đi bộ (`img_014`), người lái và người ngồi sau xe máy (`img_005`, `img_026`), người lái/khách ngồi trên xích lô (`img_006`, `img_009`), người mặc trang phục cosplay có người thật bên trong (`img_011`). | Tượng người bằng đồng/đá (`img_017`), hình ảnh người in trên biển quảng cáo billboard (`img_018`), ma-nơ-canh trưng bày quần áo (`img_033`), bóng phản chiếu trong gương (`img_020`). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu khu vực không yêu cầu gán chi tiết từng vật thể riêng lẻ; các sai sót bỏ sót vật thể trong vùng này không bị tính lỗi. | Đám đông người/xe quá xa ở hậu cảnh (chiều cao vật thể < 20px), vùng phản chiếu bóng mờ trên kính xe, phương tiện ngoài taxonomy nhưng chiếm diện tích lớn gây nhiễu (xe xích lô, xe buýt lớn, xe tuk-tuk). | Vật thể rõ ràng ở tiền cảnh hoặc trung cảnh (chiều cao >= 20px) — bắt buộc phải vẽ box cụ thể, cấm dùng ignore_region để lười gán. |

---

### 2.2 Thuộc tính đối tượng (Attributes)

Mỗi box (ngoại trừ vùng `ignore_region`) bắt buộc phải được gán đầy đủ 3 thuộc tính:

1. **`occluded_level` (Mức độ che khuất):**
   - **`0`**: Không bị che khuất (0% diện tích nhìn thấy của vật thể bị che bởi vật thể khác).
   - **`1`**: Che khuất nhẹ (Từ >0% đến 30% diện tích vật thể bị che bởi cột đèn, cây cối, hoặc vật thể khác).
   - **`2`**: Che khuất trung bình / nặng (Từ >30% đến 80% diện tích vật thể bị che khuất, phần còn lại vẫn đủ đặc trưng nhận diện).
   - **`3`**: Che khuất gần như toàn bộ (Từ >80% đến 95% diện tích bị che, chỉ nhìn thấy một phần nhỏ như mũ bảo hiểm, bánh xe, gương chiếu hậu nhưng đoán được nhờ ngữ cảnh liên tục).

2. **`truncated` (Cắt mép ảnh):**
   - **`no`**: Toàn bộ vật thể nằm trọn trong khung hình camera (không bị viền ảnh cắt qua).
   - **`yes`**: Một phần vật thể bị cắt bởi ít nhất một trong bốn mép biên của ảnh (trái, phải, trên, dưới).

3. **`can_review` (Đánh dấu cần xem lại):**
   - **`false` (không tick)**: Box đã xác định rõ ràng, đúng lớp, ôm sát biên độ tin cậy cao.
   - **`true` (tick)**: Ca khó, ranh giới mờ, phân vân giữa 2 lớp (ví dụ xe pickup là `car` hay `truck`), hoặc tỷ lệ che khuất ở ngưỡng ranh giới; gắn cờ để cả nhóm xem xét trong phiên Calibration.

---

## 3. Quy tắc vẽ box chi tiết

- **R-01 (Kích thước tối thiểu):** Chiều cao cạnh đứng tối thiểu của bounding box phải đạt **>= 20 pixels**. Các vật thể quá xa có chiều cao < 20px không gán box đơn lẻ; nếu nằm thành cụm phức tạp thì khoanh một box `ignore_region`.
- **R-02 (Quy tắc người và xe máy):**
  - Khi một hoặc nhiều người ngồi trên xe máy (`img_005`, `img_026`): **Bắt buộc tách thành các box độc lập**.
  - Mỗi người ngồi trên xe được gán 1 box `person` ôm sát phần cơ thể người (đỉnh đầu/mũ bảo hiểm xuống tới mông/chân).
  - Chiếc xe máy được gán 1 box `motorcycle` ôm trọn khung xe và hai bánh xe (không kéo bao trùm lên đầu người lái).
  - Nếu xe chở 2 người (`img_026`), kết quả sẽ gồm: 2 box `person` + 1 box `motorcycle`.
- **R-03 (Người ngồi trong cabin kín):** Người ngồi trong cabin ô tô kín (`car`, `truck`, `van`) qua lớp kính trong suốt **KHÔNG** gán box `person`, vì kính xe và phản quang thuộc về khối phương tiện. Chỉ gán `person` nếu là xe mui trần hoặc người nhoài người ra ngoài cửa sổ lộ rõ >50% thân trên.
- **R-04 (Xe chở xe - Car carrier `img_002`):**
  - Xe tải chở hàng được gán 1 box `truck` bao toàn bộ đầu kéo và sàn chở hàng.
  - Từng chiếc ô tô con được cố định trên sàn xe tải vẫn được gán box `car` riêng rẽ (set `occluded_level` phù hợp).
- **R-05 (Vật thể phi tri giác / Không có thật):**
  - Tượng đài người/ngựa (`img_017`): KHÔNG gán `person`.
  - Hình người hoặc xe in trên pano/billboard quảng cáo (`img_018`): KHÔNG gán.
  - Ma-nơ-canh trưng bày trang phục trong cửa hàng (`img_033`): KHÔNG gán `person`.
  - Bia mộ/tượng đá hình xe bán tải (`img_027`): KHÔNG gán `truck`/`car`.
  - Tượng ngựa trang trí đặt trên thùng xe (`img_028`): Box `truck` ôm xe, tượng ngựa không gán.
  - Mô hình ô tô đồ chơi, búp bê trong tủ kính (`img_038`): KHÔNG gán.
  - Người hóa trang/cosplay có người thật bên trong điều khiển (`img_011`): Gán `person`.
- **R-06 (Hình ảnh phản chiếu - Reflection `img_020`):** Không gán box cho các hình ảnh phản chiếu của người hoặc xe trong gương chiếu hậu, kính cửa sổ hoặc vũng nước phản quang.
- **R-07 (Xe máy chở hàng cồng kềnh `img_029`):** Box `motorcycle` ôm sát khung sườn vật lý của xe máy và giá chở hàng gắn liền với cấu trúc vận hành của xe. Tránh kéo phình to box bao trọn cả kiện hàng khổng lồ tách biệt làm giảm IoU chuẩn.
- **R-08 (Phương tiện ngoài Taxonomy):**
  - Xe xích lô (`img_006`, `img_009`, `img_023`, `img_034`, `img_039`): Người lái và hành khách được gán box `person`. Chiếc xích lô không thuộc 4 lớp phương tiện nên không gán box xe; nếu ở cự ly gần gây che khuất thì khoanh vùng `ignore_region` cho khung xích lô.
  - Xe tuk-tuk / auto-rickshaw (`img_004`, `img_035`): Không gán nhầm vào `car` hay `motorcycle`. Tạm thời đưa vào `ignore_region` và tick `can_review` để nhóm thảo luận ở Pha 2.

---

## 4. Ca khó thực tế (Khảo sát 40 ảnh)

| Ảnh | Tình huống quan sát | Rule ID liên quan | Quyết định đề xuất | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải đầu kéo đang chở nhiều xe ô tô con trên giá đỡ | R-04 | 1 box `truck` bao toàn xe tải; các xe con trên sàn gán từng box `car` độc lập. | Xe con là đối tượng độc lập, mô hình thị giác cần nhận diện được hình dạng ô tô dù đang được vận chuyển. |
| `img_004.jpg` | Hai chiếc xe tuk-tuk đỗ trên vỉa hè ban đêm | R-08 | Khoanh `ignore_region` cho từng chiếc tuk-tuk, tick `can_review`. | Taxonomy hiện tại không có lớp `tuk_tuk` hoặc `three_wheeler`. Gán vào `car` hay `motorcycle` đều vi phạm định nghĩa hình học và phân loại. |
| `img_006.jpg` | Đoàn xích lô chở khách du lịch tại Huế | R-02, R-08 | Gán box `person` cho từng người lái và du khách; không gán box phương tiện cho khung xích lô. | Người là đối tượng sinh học có thực cần tránh va chạm; xích lô là xe thô sơ ngoài 4 lớp cơ giới. |
| `img_011.jpg` | Nhân vật quái vật Predator đứng cạnh dàn xe mô tô độ | R-05 | Gán 1 box `person` cho nhân vật Predator; gán từng box `motorcycle` cho xe máy. | Đây là người thật mặc trang phục hóa trang (cosplay) đang đứng trên đường, camera tự hành phải nhận diện là người để đảm bảo an toàn. |
| `img_012.jpg` | Xe bán tải (pickup truck) chở rau củ đầy thùng hàng | R-01, R-07 | Gán lớp `truck`, box ôm toàn bộ cabin và thùng chở hàng. | Xe bán tải có kết cấu thùng chở hàng hở phía sau, phù hợp định nghĩa lớp `truck` hơn là `car` gia đình. |
| `img_017.jpg` | Tượng đài Công tước Wellington đội cọc tiêu giao thông | R-05 | KHÔNG gán box `person` cho bức tượng người hoặc tượng ngựa. | Tượng đá/kim loại là công trình tĩnh kiến trúc, gán `person` sẽ gây kích hoạt phanh giả (phantom braking). |
| `img_018.jpg` | Biển quảng cáo pano (billboard) in ảnh chân dung chính trị gia | R-05 | KHÔNG gán box `person` cho ảnh in 2D trên biển. | Đây là mặt phẳng đồ họa 2D trên cao, không phải vật thể vật lý chuyển động trên đường. |
| `img_020.jpg` | Cầu vồng và vệt xe phản chiếu trong gương chiếu hậu | R-06 | KHÔNG gán bất kỳ box nào vào mặt gương phản chiếu. | Ảnh ảo trong gương không phải là vật thể tồn tại ở vị trí không gian thực tế phía trước camera. |
| `img_022.jpg` | Xe mô tô cổ có gắn thêm thùng phụ chở người bên cạnh (sidecar) | R-02 | Gán 1 box `motorcycle` ôm trọn cả thân mô tô và thùng phụ sidecar; gán 2 box `person` cho 2 người. | Thùng phụ gắn cố định vào hệ khung gầm xe máy, hoạt động như một thể thống nhất khi lưu thông. |
| `img_026.jpg` | Cặp đôi nam nữ cùng ngồi trên một chiếc xe máy trong hầm | R-02 | Gán 2 box `person` độc lập (1 nam, 1 nữ) và 1 box `motorcycle` cho xe máy. | Giúp mô hình đếm chính xác số lượng người đi xe và xác định đúng trọng tâm từng đối tượng. |
| `img_027.jpg` | Tượng đài bằng đá nguyên khối tạc hình xe bán tải trong nghĩa trang | R-05 | KHÔNG gán `truck` hay `car`. | Đây là tượng điêu khắc đá cố định (monument), không phải phương tiện giao thông. |
| `img_028.jpg` | Xe bán tải chở tượng con ngựa lớn trên thùng xe | R-05, R-07 | Gán 1 box `truck` cho xe bán tải; tượng con ngựa trên thùng không gán nhãn. | Tượng ngựa là hàng hóa trang trí, không phải sinh vật sống hay phương tiện có tri giác. |
| `img_029.jpg` | Xe máy chở khối hàng cồng kềnh khổng lồ phía sau | R-07 | Gán 1 box `person` cho người lái; box `motorcycle` ôm sát thân xe máy và giá đỡ hàng chính. | Giữ IoU box xe máy chuẩn xác, tránh làm loãng đặc trưng nhận diện của xe máy. |
| `img_033.jpg` | Ma-nơ-canh mặc váy trong tủ kính cửa hàng thời trang | R-05 | KHÔNG gán box `person`. | Ma-nơ-canh là tượng nhựa tĩnh trong nhà, gán người sẽ gây lỗi hệ thống. |
| `img_035.jpg` | Xe chở khách 3 bánh (Auto Rickshaw) chở quá tải tại Ấn Độ | R-08 | Khoanh `ignore_region` cho thân xe; gán box `person` cho các hành khách nhô ra ngoài. | Xe 3 bánh ngoài taxonomy, gán `ignore_region` để tránh phạt mô hình khi nhận diện nhầm. |
| `img_038.jpg` | Tủ kính trưng bày bộ sưu tập đồ chơi, búp bê và mô hình ô tô | R-05 | KHÔNG gán bất kỳ box `car` hay `person` nào. | Mô hình tĩnh thu nhỏ sau kính shop, không thuộc đối tượng giao thông đường phố. |

---

## 5. QC và quản lý phiên bản

### 5.1 Tiêu chuẩn nghiệm thu chất lượng (QC)
- **Độ chính xác nhãn lớp (Class Accuracy):** Phải đạt **>= 95%** khi đối chiếu với ground-truth và cross-check giữa các thành viên.
- **Độ trùng khớp hình học (IoU):**
  - Đối tượng lớn (chiều cao >= 50px): IoU phải đạt **>= 0.85**.
  - Đối tượng nhỏ (chiều cao < 50px): IoU phải đạt **>= 0.70**.
- **Tỉ lệ lỗi trên ảnh:** Không vượt quá **2%** trên toàn bộ lô gán nhãn.

### 5.2 Lịch sử phiên bản

| Phiên bản | Thay đổi chính | Lý do / Căn cứ |
|---|---|---|
| `v0` | Bản khởi đầu từ giảng viên | Quá sơ sài, chưa có quy tắc tách người/xe máy, chưa có ngưỡng thuộc tính, thiếu hướng dẫn ca biên. |
| `v0.1` | Đề xuất cá nhân thành viên 101: Bổ sung định nghĩa chi tiết 6 lớp, ngưỡng phần trăm cho `occluded_level`, bổ sung 8 quy tắc vẽ box (R-01 đến R-08) và giải quyết 16 ca khó từ 40 ảnh thật. | Khảo sát thực tế toàn bộ 40 ảnh trong `data/images/` phát hiện nhiều ca dị biến (tượng, tranh quảng cáo, ma-nơ-canh, tuk-tuk, car-carrier, sidecar). |
