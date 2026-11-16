# Guideline gán nhãn — v0.1 — 275

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: 275

---

## 1. Mục tiêu

Chuẩn hóa dữ liệu gán nhãn cho camera giám sát và bài toán ADAS. Sai sót phân loại hoặc bỏ sót người đi bộ trên đường có thể dẫn tới rủi ro va chạm nghiêm trọng.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp
| Lớp | Định nghĩa | Ví dụ | Phản ví dụ |
|---|---|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV dưới 9 chỗ. | Xe cá nhân, taxi. | Xe tải chở hàng (`truck`), xe van kín (`van`). |
| `truck` | Xe tải, xe đầu kéo, xe bán tải thùng hở. | Xe tải chở ô tô (`img_002`), xe bán tải (`img_012`, `img_028`). | Xe van (`van`), xe con (`car`). |
| `van` | Xe tải thùng kín cỡ nhỏ chở hàng hoặc khách. | Xe van giao hàng bưu cục (`img_036`). | Xe tải lớn, xe buýt. |
| `motorcycle` | Xe hai bánh hoặc ba bánh có gắn động cơ. | Xe mô tô, xe tay ga, xe số (`img_005`, `img_011`, `img_022`). | Xe đạp, xích lô (`img_006`). |
| `person` | Người thật đang đi bộ, đứng, hoặc ngồi trên xe hở. | Người đi bộ (`img_014`), người đi xe máy (`img_005`, `img_026`), cosplay Predator (`img_011`). | Tượng người (`img_017`), billboard (`img_018`), ma-nơ-canh (`img_033`). |
| `ignore_region` | Vùng bỏ qua không gán chi tiết. | Đám đông quá xa (<20px), tuk-tuk (`img_004`, `img_035`). | Vật thể rõ ở tiền cảnh. |

### 2.2 Thuộc tính
- `occluded_level`: 0 (rõ 0%), 1 (che nhẹ 1-30%), 2 (che nhiều 31-80%), 3 (che nặng >80%).
- `truncated`: no (không cắt mép), yes (cắt mép ảnh).
- `can_review`: false (rõ), true (nghi ngờ cần review).

---

## 3. Quy tắc vẽ box
- **R-01:** Chiều cao tối thiểu box >= 20px.
- **R-02:** Người đi xe máy: Tách rời box `person` và box `motorcycle`.
- **R-03:** Người trong ô tô đóng kín kính: Không gán box `person`.
- **R-04:** Xe chở xe (`img_002`): 1 box `truck` và từng box `car` cho xe trên sàn.
- **R-05:** Vật thể không có tri giác (tượng `img_017`, quảng cáo `img_018`, ma-nơ-canh `img_033`, tượng xe `img_027`, đồ chơi `img_038`): KHÔNG gán.
- **R-06:** Hình phản chiếu gương xe (`img_020`): KHÔNG gán.
- **R-07:** Xe máy chở hàng cồng kềnh (`img_029`): Bo sát thân xe máy.
- **R-08:** Xe ngoài taxonomy (xích lô `img_006`, tuk-tuk `img_004`, `img_035`): Không gán nhầm lớp cơ giới; người gán `person`.

---

## 4. Ca khó thực tế
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở ô tô | R-04 | 1 truck + nhiều car | Phân tách ô tô độc lập |
| `img_011.jpg` | Predator cosplay | R-05 | Gán person | Người thật mặc đồ hóa trang |
| `img_017.jpg` | Tượng Wellington | R-05 | Không gán | Tượng kiến trúc tĩnh |
| `img_018.jpg` | Billboard ảnh người | R-05 | Không gán person | Ảnh in 2D trên biển hiệu |
| `img_020.jpg` | Cầu vồng trong gương | R-06 | Không gán | Ảnh ảo phản chiếu |
| `img_026.jpg` | Cặp đôi đi xe máy | R-02 | 2 person + 1 motorcycle | Tách từng người và xe |
| `img_027.jpg` | Tượng đá bia mộ xe | R-05 | Không gán | Tượng điêu khắc đá |
| `img_038.jpg` | Đồ chơi trong tủ kính | R-05 | Không gán | Mô hình đồ chơi tĩnh |

---

## 5. QC và phiên bản
- Tuân thủ ngưỡng đo lường chất lượng: IoU >= 0.85 (lớn), >= 0.70 (nhỏ), độ chính xác lớp >= 95%.
