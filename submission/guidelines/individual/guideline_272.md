# Guideline gán nhãn — v0.1 — 272

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: 272

---

## 1. Mục tiêu

Guideline này quy định cách gán nhãn người và phương tiện giao thông trong ảnh đường phố nhằm xây dựng dataset phục vụ huấn luyện và đánh giá mô hình Object Detection.

Dataset được sử dụng cho các bài toán:

- Phát hiện người đi bộ và người điều khiển phương tiện.
- Phát hiện, phân loại các loại phương tiện giao thông.
- Phân biệt người lái xe (`rider`) và người đi bộ (`pedestrian`).
- Nhận diện tình trạng đội mũ bảo hiểm của người điều khiển phương tiện.
- Hỗ trợ các ứng dụng phân tích giao thông, giám sát đường phố và an toàn giao thông.

### 1.1. Những lỗi cần hạn chế

Các lỗi gán nhãn sau có thể ảnh hưởng trực tiếp đến chất lượng mô hình:

1. **Sai class:** Phân loại nhầm xe máy thành xe đạp, xe tải thành xe van hoặc người lái xe thành người đi bộ.
2. **Thiếu đối tượng:** Bỏ qua người hoặc phương tiện bị che khuất một phần.
3. **Box không chính xác:** Bounding box quá rộng, quá hẹp hoặc bao gồm nhiều đối tượng khác nhau.
4. **Sai attribute:** Xác định sai mức độ che khuất, trạng thái cắt ảnh hoặc tình trạng đội mũ bảo hiểm.
5. **Không thống nhất:** Các annotator xử lý cùng một tình huống theo những cách khác nhau.

### 1.2. Nguyên tắc chung

- Mỗi đối tượng được gán đúng một class phù hợp.
- Mỗi đối tượng riêng biệt có một bounding box riêng.
- Không gộp người và phương tiện thành một đối tượng.
- Chỉ gán nhãn những đối tượng thực sự nhìn thấy trong ảnh.
- Không suy đoán phần bị che khuất.
- Khi không chắc chắn về class hoặc ranh giới đối tượng, sử dụng `can_review`.
- Không tự tạo class mới ngoài danh sách đã thống nhất.

---

## 2. Lớp và thuộc tính

### 2.1. Danh sách lớp (Classes)

Mỗi đối tượng được gán vào một trong các class dưới đây.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới bốn bánh hoặc tương tự được thiết kế chủ yếu để chở người. | Sedan, SUV, hatchback. | Xe tải, xe van, xe buýt. |
| `truck` | Phương tiện cơ giới được thiết kế chủ yếu để vận chuyển hàng hóa, có kết cấu xe tải. | Xe tải thùng, xe ben, xe đầu kéo. | Ô tô con, xe van. |
| `van` | Phương tiện cơ giới cỡ nhỏ hoặc trung bình có khoang thân kín, dùng để chở người hoặc hàng hóa. | Minivan, cargo van. | Sedan, xe tải lớn. |
| `rv` | Phương tiện chuyên dụng có không gian sinh hoạt hoặc lưu trú được tích hợp trên xe. | Motorhome, recreational vehicle, camper van chuyên dụng. | Xe van chở hàng thông thường, xe khách. |
| `motorcycle` | Phương tiện cơ giới hai bánh hoặc ba bánh thuộc loại xe máy, được vận hành bằng động cơ. | Xe máy, mô tô, scooter, xe máy điện, xe ba bánh kiểu motorcycle. | Xe đạp, tuk-tuk, xích lô. |
| `bicycle` | Phương tiện di chuyển bằng sức người, sử dụng bàn đạp và không có động cơ chính để vận hành. | Xe đạp thường, xe đạp thể thao. | Xe máy, xe đạp điện có động cơ. |
| `other_vehicle` | Phương tiện giao thông không thuộc các class phương tiện đã định nghĩa ở trên. | Tuk-tuk, xe xích lô, xe ba bánh chở khách, xe kéo. | Xe máy thông thường, ô tô, xe tải. |
| `rider` | Người đang trực tiếp điều khiển hoặc ngồi trên phương tiện giao thông. | Người lái xe máy, người đạp xe, người lái tuk-tuk. | Người đi bộ bên đường. |
| `pedestrian` | Người đang đi bộ và không trực tiếp điều khiển phương tiện giao thông. | Người đi bộ trên vỉa hè, người băng qua đường. | Người đang lái xe máy hoặc xe đạp. |
| `ignore_region` | Vùng chứa đối tượng thuộc phạm vi dataset nhưng không thể phân biệt hoặc gán nhãn riêng một cách đáng tin cậy. | Đám đông phương tiện chồng lấn không thể tách box. | Một chiếc xe bị che khuất nhưng vẫn nhận diện được. |

### 2.2. Quy tắc phân biệt các class

**A. `rider` và `pedestrian`**

- Người đang điều khiển xe máy được gán `rider`.
- Người đang đạp xe được gán `rider`.
- Người đang điều khiển tuk-tuk hoặc xích lô được gán `rider`.
- Người đi bộ, kể cả khi đang dắt xe, được gán `pedestrian`.
- Người ngồi trên xe với vai trò hành khách không được tự động gán `rider`. Trong guideline này, chỉ gán `rider` cho người trực tiếp điều khiển phương tiện.

**B. `motorcycle`, `bicycle` và `other_vehicle`**

- Xe máy hai bánh thông thường thuộc `motorcycle`.
- Xe đạp sử dụng bàn đạp thuộc `bicycle`.
- Tuk-tuk thuộc `other_vehicle`.
- Xích lô thuộc `other_vehicle`.
- Xe ba bánh chở hàng hoặc chở khách có kết cấu không thuộc loại motorcycle thông thường thuộc `other_vehicle`.
- Không phân loại tuk-tuk hoặc xích lô thành `motorcycle` chỉ vì chúng có ba bánh.

**C. `car`, `truck`, `van` và `rv`**

- Sedan, SUV và ô tô con thuộc `car`.
- Xe tải chở hàng thuộc `truck`.
- Xe van chở người hoặc hàng hóa thuộc `van`.
- Xe có cấu trúc nhà ở hoặc không gian sinh hoạt tích hợp thuộc `rv`.
- Không phân loại xe van thông thường thành `rv` chỉ vì xe có khoang kín.

### 2.3. Vật không thuộc class nào

Nếu đối tượng xuất hiện trong ảnh nhưng không thuộc bất kỳ class nào:

1. Không tự ý gán vào class gần giống nhất.
2. Không tạo class mới khi chưa được trưởng nhóm phê duyệt.
3. Nếu đối tượng không thuộc phạm vi dataset, không cần gán nhãn.
4. Nếu đối tượng thuộc phạm vi cần nhận diện nhưng không thể phân loại đáng tin cậy, đánh dấu `can_review`.
5. Chỉ sử dụng `ignore_region` khi có một vùng đối tượng không thể phân tách hoặc gán nhãn riêng một cách đáng tin cậy.

Không sử dụng `ignore_region` để thay thế việc gán nhãn những đối tượng chỉ đơn giản là khó nhìn.

### 2.4. Thuộc tính (Attributes)

#### A. `occluded_level`

Xác định mức độ một đối tượng bị che khuất bởi các vật thể khác.

| Giá trị | Mức độ | Ngưỡng che khuất | Ý nghĩa |
|---|---|---|---|
| `0` | Không hoặc gần như không bị che | 0–10% | Hầu hết đối tượng đều nhìn thấy. |
| `1` | Che khuất nhẹ | Trên 10% đến 30% | Một phần nhỏ đối tượng bị che. |
| `2` | Che khuất trung bình | Trên 30% đến 60% | Một phần đáng kể đối tượng bị che nhưng vẫn nhận diện được. |
| `3` | Che khuất nặng | Trên 60% | Chỉ còn một phần đối tượng nhìn thấy nhưng vẫn xác định được class. |

**Quy tắc:**

- Ước lượng tỷ lệ phần trăm diện tích đối tượng bị che so với toàn bộ đối tượng.
- Chỉ đánh giá phần bị che bởi vật thể khác, không tính phần nằm ngoài khung hình.
- Nếu không thể xác định class do bị che quá nhiều, không cố gắng gán class. Đánh dấu `can_review` hoặc xử lý theo quy tắc `ignore_region`.
- Áp dụng độc lập cho từng người và phương tiện.

#### B. `truncated`

Xác định đối tượng có bị cắt bởi cạnh ảnh hay không.

| Giá trị | Ý nghĩa |
|---|---|
| `no` | Toàn bộ đối tượng nằm trong khung hình. |
| `yes` | Một phần đối tượng nằm ngoài khung hình do bị cắt bởi một hoặc nhiều cạnh ảnh. |

**Quy tắc:**

- Nếu đối tượng chạm cạnh ảnh nhưng vẫn nhìn thấy toàn bộ, chọn `no`.
- Nếu một phần đối tượng thực sự nằm ngoài ảnh, chọn `yes`.
- Không nhầm vật bị một đối tượng khác che khuất với vật bị cắt khỏi ảnh.
- Đối tượng vừa bị che khuất vừa bị cắt ảnh phải được đánh giá cả hai thuộc tính độc lập.

#### C. `helmet`

Xác định người điều khiển phương tiện có đội mũ bảo hiểm hay không.

**Phạm vi áp dụng:** Chỉ áp dụng cho class `rider`.

| Giá trị | Ý nghĩa |
|---|---|
| `yes` | Người có đội mũ bảo hiểm và có thể nhận diện rõ. |
| `no` | Người không đội mũ bảo hiểm, phần đầu nhìn thấy rõ. |
| `unknown` | Không thể xác định do che khuất, ảnh mờ hoặc kích thước quá nhỏ. |

**Quy tắc:**

1. Chỉ đánh giá mũ trên đầu của người.
2. Mũ bảo hiểm phải có hình dạng phù hợp và nhìn thấy đủ rõ.
3. Mũ lưỡi trai, mũ vải, mũ thời trang và nón thông thường không được tính là mũ bảo hiểm.
4. Nếu đầu bị che khuất hoặc không đủ chi tiết để xác định, chọn `unknown`.
5. Không suy đoán tình trạng đội mũ dựa trên những người xung quanh.
6. Không áp dụng attribute này cho `pedestrian` hoặc các class phương tiện.

#### D. `can_review`

Xác định đối tượng cần người khác kiểm tra lại.

| Giá trị | Ý nghĩa |
|---|---|
| `false` | Người gán nhãn chắc chắn về class, box và thuộc tính. |
| `true` | Có điểm chưa chắc chắn cần trưởng nhóm hoặc annotator khác xem lại. |

**Đánh dấu `can_review = true` khi:**

- Không chắc chắn giữa hai class.
- Không thể xác định rõ ranh giới bounding box.
- Đối tượng bị che khuất nghiêm trọng.
- Không thể phân biệt loại phương tiện.
- Không chắc chắn về tình trạng đội mũ bảo hiểm.
- Có trường hợp không được quy định rõ trong guideline.

Không sử dụng `can_review` thay cho việc cố gắng hoàn thành một nhãn bình thường.

---

## 3. Quy tắc vẽ Bounding Box

### 3.1. Nguyên tắc chung

- Bounding box phải là hình chữ nhật bao sát đối tượng.
- Box phải bao phủ toàn bộ phần đối tượng nhìn thấy.
- Không bao gồm vùng nền không thuộc đối tượng.
- Không vẽ box quá rộng hoặc quá hẹp.
- Mỗi đối tượng riêng biệt có một box riêng.
- Không gộp nhiều người hoặc nhiều phương tiện thành một box.

### 3.2. Đối tượng bị che khuất (Occlusion)

- Vẫn gán nhãn đối tượng nếu có thể nhận diện được class.
- Box chỉ bao quanh phần đối tượng thực sự nhìn thấy.
- Không tự vẽ thêm phần bị che khuất.
- Gán `occluded_level` theo tỷ lệ bị che.
- Nếu không thể nhận diện đối tượng đáng tin cậy, đánh dấu `can_review`.

**Ví dụ:** Trong ảnh giao thông đông đúc, một chiếc xe máy bị chiếc xe khác che mất phần thân nhưng vẫn nhìn thấy bánh xe và phần đầu xe đủ để nhận diện. Vẫn phải gán box `motorcycle`.

### 3.3. Đối tượng bị cắt bởi cạnh ảnh (Truncation)

- Vẽ box đến sát cạnh ảnh tại vị trí đối tượng bị cắt.
- Không mở rộng box ra ngoài khung hình.
- Gán `truncated = yes`.
- Nếu đối tượng vẫn nhận diện được, vẫn phải gán class bình thường.

### 3.4. Phản chiếu, hình in, tượng và đồ chơi

Không gán nhãn cho:

- Người hoặc phương tiện chỉ xuất hiện trong gương phản chiếu.
- Hình ảnh người hoặc xe in trên poster, biển quảng cáo hoặc màn hình.
- Tượng người hoặc tượng động vật.
- Đồ chơi mô phỏng phương tiện.
- Hình vẽ hoặc mô hình không phải phương tiện thật đang tham gia giao thông.

**Lý do:** Dataset hướng đến nhận diện người và phương tiện thực tế trong môi trường giao thông.

### 3.5. Xe chủ (Ego vehicle)

- Không gán nhãn bộ phận của chính phương tiện chứa camera nếu chỉ nhìn thấy một phần nội thất hoặc thân xe của camera.
- Nếu một phương tiện khác xuất hiện trong ảnh và nhìn thấy rõ, vẫn gán nhãn bình thường.
- Nếu không xác định được phương tiện nào là ego vehicle, đánh dấu `can_review`.

### 3.6. Đối tượng quá nhỏ

- Đối tượng có chiều rộng hoặc chiều cao nhỏ hơn 8 pixel được xem là quá nhỏ để gán nhãn đáng tin cậy.
- Không tạo box cho những đối tượng không thể xác định class.
- Nếu đối tượng nhỏ nhưng vẫn nhận diện rõ và có thể vẽ box chính xác, vẫn gán nhãn và đánh dấu `can_review` nếu cần.
- Không sử dụng `ignore_region` chỉ vì đối tượng nhỏ.

### 3.7. Đám đông và nhiều phương tiện chồng lấn

- Gán box riêng cho từng người và phương tiện có thể nhận diện được.
- Không gộp toàn bộ đám đông thành một box.
- Không gán một box bao gồm nhiều xe máy.
- Nếu một số đối tượng không thể phân biệt ranh giới do chồng lấn quá nhiều, sử dụng `ignore_region` cho đúng vùng không thể phân tách.
- Không để vùng `ignore_region` bao phủ những đối tượng đã có box hợp lệ.

### 3.8. Một người và một phương tiện

Người và phương tiện luôn được gán nhãn độc lập.

Ví dụ:

- Người điều khiển xe máy: một box `rider`.
- Xe máy: một box `motorcycle`.
- Người điều khiển xe đạp: một box `rider`.
- Xe đạp: một box `bicycle`.
- Người điều khiển tuk-tuk: một box `rider`.
- Tuk-tuk: một box `other_vehicle`.

Không gộp người và phương tiện thành một box duy nhất.

### 3.9. Hành khách trên phương tiện

- Người trực tiếp điều khiển phương tiện được gán `rider`.
- Người ngồi phía sau hoặc ngồi trong phương tiện với vai trò hành khách không được gán `rider`.
- Hành khách nhìn thấy rõ được gán `pedestrian` theo quy ước class người của dataset này.
- Không gán một người vừa là `rider` vừa là `pedestrian`.

### 3.10. Phương tiện đặc biệt

- Tuk-tuk: `other_vehicle`.
- Xe xích lô: `other_vehicle`.
- Xe ba bánh chở hàng: `other_vehicle`, trừ trường hợp thuộc loại motorcycle rõ ràng.
- Xe đạp thường: `bicycle`.
- Xe đạp điện: `motorcycle` nếu là phương tiện có động cơ vận hành chính; nếu không xác định được cấu tạo, đánh dấu `can_review`.
- Xe RV/camper chuyên dụng: `rv`.
- Xe van thông thường: `van`.

---

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| `img_037` | Hai xe tuk-tuk ba bánh xuất hiện cùng người điều khiển. | Gán từng xe là `other_vehicle`, người điều khiển là `rider`. | Tuk-tuk không thuộc loại motorcycle thông thường. |
| `img_021` | Giao thông đông đúc với nhiều xe máy và người điều khiển bị che khuất. | Gán `motorcycle` và `rider` riêng cho từng đối tượng nhận diện được. Dùng `ignore_region` cho vùng không thể phân tách. | Tránh bỏ sót đối tượng và tránh box sai. |
| `img_003` | Xe đạp xuất hiện cùng người đang đạp xe. | Gán `bicycle` cho xe và `rider` cho người. | Người và phương tiện là hai đối tượng độc lập. |
| `img_003` | Người đi bộ bên cạnh xe máy. | Gán `pedestrian` cho người đi bộ và `motorcycle` cho xe. | Không nhầm người đi bộ với người điều khiển xe. |
| `img_005` | Người điều khiển xe máy đội mũ bảo hiểm. | Gán `helmet = yes` cho `rider`. | Phần đầu nhìn thấy rõ và có mũ bảo hiểm. |
| `img_008` | Người điều khiển xe máy không đội mũ bảo hiểm. | Gán `helmet = no` cho `rider`. | Phần đầu nhìn thấy rõ và không có mũ bảo hiểm. |
| `img_021` | Người điều khiển xe máy bị che khuất phần đầu. | Gán `helmet = unknown`. | Không đủ thông tin để xác định. |
| `img_021` | Một xe máy bị xe khác che khuất một phần. | Gán box cho phần nhìn thấy, chọn `occluded_level` phù hợp. | Đối tượng vẫn nhận diện được. |
| `img_021` | Một chiếc xe bị cắt ở cạnh ảnh. | Gán box đến sát cạnh ảnh và `truncated = yes`. | Đối tượng nằm ngoài một phần khung hình. |
| `img_021` | Nhiều xe máy chồng lấn, không thể xác định ranh giới từng xe. | Gán nhãn những xe nhận diện được; vùng không thể tách dùng `ignore_region`. | Không tạo box giả hoặc box gộp. |
| `img_017` | Tượng người cưỡi ngựa hoặc tượng người đặt bên đường. | Không gán `person` hoặc `rider`. | Đây không phải người thực tế. |
| `img_018` | Người xuất hiện trong gương hoặc hình quảng cáo. | Không gán nhãn. | Đây là hình ảnh phản chiếu hoặc hình in, không phải đối tượng thực tế. |
| `img_037` | Xe van được cải tạo thành xe lưu trú. | Gán `rv` nếu có cấu trúc sinh hoạt tích hợp rõ ràng; nếu không chắc, `can_review`. | Phân biệt RV với van chở hàng thông thường. |
| `img_021` | Một phương tiện rất nhỏ ở xa. | Gán nếu nhận diện rõ và box có kích thước tối thiểu; nếu không, không cố đoán. | Đảm bảo chất lượng nhãn. |

### 4.1. Quy trình xử lý ca chưa rõ

Khi gặp trường hợp không có quy tắc cụ thể:

1. Không tự suy đoán class.
2. Gán `can_review = true` cho đối tượng liên quan nếu có thể tạo nhãn tạm thời đáng tin cậy.
3. Ghi lại tên ảnh và mô tả vấn đề.
4. Gửi trường hợp cho trưởng nhóm xem xét.
5. Chỉ cập nhật nhãn sau khi có quyết định thống nhất.
6. Nếu quyết định làm thay đổi quy tắc chung, cập nhật phiên bản guideline.

---

## 5. QC và phiên bản

### 5.1. Quy trình kiểm tra chất lượng (Quality Control)

Tất cả nhãn phải được kiểm tra trước khi xuất dataset.

**A. Kiểm tra class**

- 100% đối tượng phải thuộc class hợp lệ.
- Không nhầm `rider` với `pedestrian`.
- Không nhầm `motorcycle` với `bicycle`.
- Tuk-tuk và xích lô phải thuộc `other_vehicle`.
- RV phải được phân biệt với van thông thường.

**B. Kiểm tra bounding box**

- Box phải bao sát phần đối tượng nhìn thấy.
- Không bao gồm quá nhiều nền.
- Không gộp nhiều đối tượng.
- Không bỏ sót đối tượng có thể nhận diện.
- Box phải nằm trong phạm vi ảnh.

**C. Kiểm tra attributes**

- Tất cả đối tượng phải có `occluded_level` và `truncated`.
- `helmet` chỉ áp dụng cho `rider`.
- `helmet` phải thuộc một trong ba giá trị `yes`, `no`, `unknown`.
- `can_review` phải được đánh dấu khi có nghi vấn.

**D. Kiểm tra các ca khó**

- Kiểm tra riêng các ảnh có nhiều đối tượng chồng lấn.
- Kiểm tra các đối tượng nhỏ hoặc bị che khuất.
- Kiểm tra các phương tiện đặc biệt như tuk-tuk, xích lô và RV.
- Kiểm tra các nhãn có `can_review = true`.

### 5.2. Ngưỡng QC đề xuất

| Tiêu chí | Ngưỡng chấp nhận |
|---|---|
| Class chính xác | ≥ 98% |
| Bounding box hợp lệ | ≥ 95% |
| Attribute chính xác | ≥ 95% |
| Đối tượng bị bỏ sót | ≤ 2% |
| Nhãn sai định dạng | 0% |
| Nhãn chưa xử lý `can_review` trước khi hoàn thành | 0% |

Các tỷ lệ được tính trên mẫu ảnh được kiểm tra. Với class ít xuất hiện như `rv` và `other_vehicle`, cần kiểm tra toàn bộ mẫu có class đó nếu số lượng nhỏ.

### 5.3. Người chịu trách nhiệm QC

- **Annotator:** Gán nhãn theo guideline và tự kiểm tra kết quả.
- **Reviewer:** Kiểm tra class, box và attributes; phản hồi các lỗi.
- **Trưởng nhóm:** Quyết định những ca mơ hồ, giải quyết bất đồng và phê duyệt cập nhật guideline.

Nếu hai annotator có cách hiểu khác nhau về cùng một ảnh, không tự ý chọn một cách. Cần chuyển trường hợp cho trưởng nhóm quyết định.

### 5.4. Lịch sử thay đổi phiên bản

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Guideline ban đầu với các class car, truck, van, motorcycle, person và ignore_region. | Template ban đầu của giảng viên. |
| v1 | Bổ sung `rider`, `pedestrian`, `bicycle`, `rv` và `other_vehicle`. | Cần phân biệt người đi bộ, người điều khiển phương tiện và các loại phương tiện đặc biệt. |
| v1 | Bổ sung attribute `helmet` với ba mức yes, no, unknown. | Phục vụ bài toán phân tích an toàn giao thông. |
| v1 | Chuẩn hóa ngưỡng `occluded_level`, quy tắc `truncated` và `can_review`. | Giảm sự khác biệt giữa các annotator. |
| v1 | Bổ sung quy tắc box cho đám đông, vật thể nhỏ, tượng, phản chiếu và phương tiện ba bánh. | Xử lý các tình huống thực tế trong dataset. |

### 5.5. Nguyên tắc quản lý phiên bản

- Mọi thay đổi ảnh hưởng đến class, attributes hoặc cách vẽ box phải được ghi lại.
- Thay đổi nhỏ về câu chữ nhưng không làm thay đổi quyết định gán nhãn không bắt buộc tăng phiên bản.
- Thay đổi quy tắc gán nhãn phải được trưởng nhóm phê duyệt.
- Tất cả annotator phải sử dụng cùng một phiên bản guideline.
- Không tự ý thay đổi nhãn đã hoàn thành nếu chưa có quyết định thống nhất.

---

**Kết luận**

Guideline này được xây dựng nhằm đảm bảo việc gán nhãn người và phương tiện giao thông có tính nhất quán, rõ ràng và có thể tái lập giữa các annotator. Các trường hợp khó cần được đánh dấu `can_review` và chuyển cho trưởng nhóm thay vì tự suy đoán. Dataset sau khi gán nhãn phải trải qua quy trình QC trước khi được sử dụng để huấn luyện mô hình.