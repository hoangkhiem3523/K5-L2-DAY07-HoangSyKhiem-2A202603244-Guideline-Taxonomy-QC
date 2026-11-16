# Guideline gán nhãn — v0.1 — Thành viên 270

- Phiên bản: v0.1 (Đề xuất cá nhân - Pha 1)
- Ngày: 2026-11-16
- Người viết: Thành viên 270 (Hoàng Thuận Thiên)

---

## 1. Mục tiêu

Xây dựng bộ quy chuẩn bounding box chất lượng cao phục vụ huấn luyện mô hình phát hiện người và phương tiện giao thông đường phố.
- Nguy cơ nếu gán sai:
  - Bỏ sót người đi bộ hoặc xe máy gây nguy hiểm va chạm trong hệ thống tự hành ADAS.
  - Nhận diện nhầm ma-nơ-canh (`img_033`) hoặc tượng đá (`img_017`, `img_027`) thành người/xe thật dẫn đến phanh khẩn cấp không cần thiết.

---

## 2. Lớp và thuộc tính

### 2.1 Định nghĩa lớp đối tượng

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ |
|---|---|---|---|
| `car` | Ô tô con 4-7 chỗ (sedan, SUV, hatchback). | Xe taxi, SUV đô thị. | Xe tải thùng rời (`truck`), xe van kín (`van`). |
| `truck` | Xe cơ giới chở hàng tải trọng lớn hoặc xe bán tải có thùng hở. | Xe tải chở ô tô (`img_002`), xe bán tải chở hàng (`img_012`). | Xe ô tô con (`car`), tượng bia mộ đá (`img_027`). |
| `van` | Xe tải nhẹ thùng kín liền khối với cabin. | Xe giao hàng bưu chính (`img_036`), minivan. | Xe buýt lớn, xe tải sàn hở. |
| `motorcycle` | Xe máy, mô tô 2 bánh hoặc 3 bánh có động cơ. | Xe số, xe ga, mô tô (`img_005`, `img_011`, `img_022`). | Xe đạp thường, xe xích lô đạp (`img_006`). |
| `person` | Người thật đang đi bộ hoặc đang điều khiển phương tiện hở. | Người đi bộ (`img_014`), người lái/ngồi sau xe máy (`img_005`, `img_026`), cosplay Predator (`img_011`). | Tượng người (`img_017`), hình in người trên biển quảng cáo (`img_018`), ma-nơ-canh (`img_033`). |
| `ignore_region` | Vùng không yêu cầu gán chi tiết từng đối tượng. | Đám đông quá xa (<20px), xe ngoài taxonomy như tuk-tuk (`img_004`, `img_035`). | Vật thể rõ ràng ở tiền cảnh. |

### 2.2 Thuộc tính đối tượng
- `occluded_level`:
  - `0`: Không bị che (0%)
  - `1`: Che nhẹ (1 - 30%)
  - `2`: Che trung bình (31 - 80%)
  - `3`: Che nặng (> 80%)
- `truncated`: `no` (trọn trong ảnh), `yes` (bị cắt mép ảnh).
- `can_review`: `false` / `true` (đánh dấu ca khó cần review).

---

## 3. Quy tắc vẽ box chi tiết
- **R-01:** Chiều cao box tối thiểu >= 20px. Dưới 20px không gán box đơn lẻ.
- **R-02:** Người đi xe máy: Tách riêng box `person` (người) và box `motorcycle` (xe).
- **R-03:** Người trong ô tô kín: Không gán box `person`.
- **R-04:** Xe chở xe (`img_002`): 1 box `truck` cho xe tải + các box `car` cho xe trên sàn.
- **R-05:** Vật thể phi tri giác (tượng `img_017`, quảng cáo `img_018`, ma-nơ-canh `img_033`, tượng xe `img_027`, đồ chơi `img_038`): KHÔNG gán.
- **R-06:** Hình phản chiếu trong gương/kính (`img_020`): KHÔNG gán.
- **R-07:** Xe máy chở hàng cồng kềnh (`img_029`): Bo sát thân xe máy.
- **R-08:** Phương tiện ngoài taxonomy (xích lô `img_006`, tuk-tuk `img_004`, `img_035`): Không gán lớp xe sai; người vẫn gán `person`.

---

## 4. Ca khó thực tế
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở ô tô | R-04 | 1 truck, nhiều car | Tách xe con độc lập |
| `img_011.jpg` | Predator cosplay | R-05 | Gán person | Người thật hóa trang |
| `img_017.jpg` | Tượng Wellington | R-05 | Không gán | Tượng điêu khắc tĩnh |
| `img_018.jpg` | Billboard ảnh người | R-05 | Không gán person | Ảnh in 2D trên biển |
| `img_020.jpg` | Cầu vồng trong gương | R-06 | Không gán | Ảnh ảo phản chiếu |
| `img_026.jpg` | Cặp đôi đi xe máy | R-02 | 2 person + 1 motorcycle | Tách từng người và xe |
| `img_027.jpg` | Tượng bia mộ xe tải | R-05 | Không gán | Tượng đá nghĩa trang |
| `img_038.jpg` | Đồ chơi trong tủ kính | R-05 | Không gán | Mô hình đồ chơi tĩnh |

---

## 5. QC và lịch sử phiên bản
- Cam kết tuân thủ ngưỡng IoU >= 0.85 (vật lớn) và >= 0.70 (vật nhỏ), độ chính xác lớp >= 95%.
- Phiên bản v0.1 cá nhân khởi tạo từ v0.
