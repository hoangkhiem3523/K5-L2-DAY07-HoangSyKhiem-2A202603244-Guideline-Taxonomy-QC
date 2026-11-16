# Guideline gán nhãn — v0.1 — 275

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: 275

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố 2D. Mô hình cần biết vật nào đang ở trên đường để tránh va chạm.

Sai nguy hiểm nhất là bỏ người đang đi hoặc đang ngồi trên xe hở, và gán tượng, ảnh in, đồ chơi, bóng trong gương thành người hoặc xe. Box kéo theo hàng hóa làm sai kích thước vật.

## 2. Lớp và thuộc tính

### Lớp

Mỗi vật thuộc đúng một lớp. Không thêm lớp mới. Vật không thuộc lớp nào thì không gán box riêng. Nếu khối đó lớn và dễ bị nhận nhầm thành xe, khoanh một `ignore_region` quanh khối ấy. `ignore_region` không có thuộc tính.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người, cabin và thân xe là một khối, không có thùng hàng riêng phía sau cabin. | Sedan đen [img_023](../../../data/images/img_023.jpg), SUV bạc [img_012](../../../data/images/img_012.jpg), taxi xanh [img_040](../../../data/images/img_040.jpg). | Pickup có thùng sau (`truck`). Xe buýt ([img_037](../../../data/images/img_037.jpg), [img_040](../../../data/images/img_040.jpg)) không phải `car`. |
| `truck` | Xe có cabin và một thùng hàng riêng phía sau. Thùng hở, phủ bạt, hay có nắp cứng gắn trên thùng vẫn là `truck`. | Pickup trắng [img_001](../../../data/images/img_001.jpg), xe tải xanh bạt [img_036](../../../data/images/img_036.jpg), pickup nắp thùng trắng [img_037](../../../data/images/img_037.jpg). | Van thân liền [img_005](../../../data/images/img_005.jpg). Tượng đá hình bán tải [img_027](../../../data/images/img_027.jpg) không phải `truck`. |
| `van` | Xe thùng kín, cabin và khoang sau nằm trong cùng một vỏ, không có sàn hàng tách khỏi cabin. | Van bạc phía sau đoàn xe máy [img_005](../../../data/images/img_005.jpg), van taxi bạc [img_016](../../../data/images/img_016.jpg). | Minibus nhiều cửa sổ khách [img_009](../../../data/images/img_009.jpg) không phải `van`. Pickup có nắp thùng [img_037](../../../data/images/img_037.jpg) không phải `van`. |
| `motorcycle` | Xe máy, xe tay ga hoặc mô tô có bánh và khung xe nhận ra được. Sidecar gắn liền tính là một xe. | Xe tay ga đỏ [img_026](../../../data/images/img_026.jpg), mô tô kèm sidecar [img_022](../../../data/images/img_022.jpg). | Xe đạp [img_023](../../../data/images/img_023.jpg), xích lô [img_023](../../../data/images/img_023.jpg), tuk-tuk [img_004](../../../data/images/img_004.jpg). |
| `person` | Người thật. Đứng, đi, ngồi, hoặc ngồi trên xe hở đều là `person`. | Người sang đường [img_034](../../../data/images/img_034.jpg), hai người trên một xe máy [img_026](../../../data/images/img_026.jpg). | Tượng [img_017](../../../data/images/img_017.jpg), ảnh trên biển [img_018](../../../data/images/img_018.jpg), ma-nơ-canh [img_033](../../../data/images/img_033.jpg), thú bông [img_019](../../../data/images/img_019.jpg). |
| `ignore_region` | Vùng có vật thật nhưng không gán từng vật. Bỏ sót trong vùng này không tính lỗi. | Đám xe máy xa đến mức không tách được từng xe [img_021](../../../data/images/img_021.jpg). Thân tuk-tuk [img_004](../../../data/images/img_004.jpg). | Vật nhìn rõ, cao từ 20 px trở lên, thuộc một lớp ở trên. Không dùng vùng này để khỏi vẽ. |

### Thuộc tính

Chỉ box `car`, `truck`, `van`, `motorcycle`, `person` mới có thuộc tính. Ước lượng phần bị che trên diện tích của cả vật, không chỉ trên phần đang nhìn thấy.

`occluded_level` chỉ tính vật khác che lên vật này. Mép ảnh cắt vật không cộng vào mức che. Mép ảnh ghi ở `truncated`.

| Mức | Phần diện tích bị vật khác che | Cách làm |
|---|---|---|
| `0` | Dưới 10% | Gán bình thường. |
| `1` | Từ 10% đến dưới 40% | Gán. Box ôm phần nhìn thấy và phần bị che còn suy ra được sát biên vật. |
| `2` | Từ 40% đến dưới 70% | Gán nếu còn nhận ra lớp. |
| `3` | Từ 70% trở lên, nhưng lớp vẫn nhận ra từ phần còn lại | Gán. |

Nếu phần nhìn thấy dưới khoảng 15% và lớp chỉ là đoán, không vẽ box. Ghi ảnh đó vào bảng ca khó nếu vật đó quan trọng. Hàng rào che xe ở [img_018](../../../data/images/img_018.jpg) vẫn là xe: mức che theo lượng xe bị thanh sắt đè lên, không bỏ xe vì có hàng rào.

`truncated`

- `no`: cả vật nằm trong khung ảnh.
- `yes`: một trong bốn mép ảnh cắt vật. Người áo đỏ sát mép phải [img_016](../../../data/images/img_016.jpg) là `truncated=yes`. Người đó vẫn có `occluded_level` riêng nếu tuk-tuk che thêm.

`can_review`

- Không tick: lớp và biên box đã chọn được bằng quy tắc dưới.
- Tick: hai lớp đều có lý, hoặc chưa chắc có người thật bên trong một hình người. Ca chưa có box thì không tick được. Ghi vào mục 4.

## 3. Quy tắc vẽ box

Box ôm sát vật của đúng lớp. Hàng buộc thêm, giỏ gánh, tượng chở trên xe, ảnh in và bóng phản chiếu không kéo box ra.

- **H-01. Kích thước và đám đông.** Không vẽ box riêng khi chiều cao box dưới 20 px. Nhiều vật dưới ngưỡng đó dính thành một đám thì một `ignore_region` cho cả đám. Vật cao từ 20 px và tách được từng người hoặc từng xe thì vẽ từng box, kể cả trong dòng xe đông [img_021](../../../data/images/img_021.jpg).
- **H-02. Người trên xe máy.** Mỗi người một box `person`, từ mũ đến phần người nhìn thấy, kể cả chân. Xe một box `motorcycle` ôm khung, yên, bánh, tay lái. Hai box được chồng lên nhau ở chỗ chân và yên. Không vẽ một box chung cho người và xe. [img_026](../../../data/images/img_026.jpg): 2 `person` + 1 `motorcycle`. Thử lại trên [img_006](../../../data/images/img_006.jpg): xe tay ga bên trái cũng 2 `person` + 1 `motorcycle`. Xe đạp không được box `motorcycle`. Người đạp xe vẫn là `person`.
- **H-03. Người trong xe kín và xe hở.** Người chỉ thấy qua kính của `car`, `truck`, `van` thì không vẽ `person`. Người trên xe máy, xích lô, tuk-tuk, xe đạp thì vẽ `person`. Người lái xe diễu hành mui hở [img_019](../../../data/images/img_019.jpg) là `person`.
- **H-04. Xe chở xe.** Cả đầu kéo và rơ-moóc là một box `truck`, vì rơ-moóc không tự đi. Mỗi ô tô trên sàn mà còn nhận ra là ô tô thì một box `car`, với `occluded_level` theo lượng xe bị khung sàn che. Người đứng ở đuôi xe [img_002](../../../data/images/img_002.jpg) là `person`.
- **H-05. Không phải người hoặc xe đang đi trên đường.** Không vẽ box cho tượng người và ngựa [img_017](../../../data/images/img_017.jpg), ảnh người trên biển [img_018](../../../data/images/img_018.jpg), ma-nơ-canh và ảnh in trong tủ kính [img_033](../../../data/images/img_033.jpg), tượng bán tải trong nghĩa trang [img_027](../../../data/images/img_027.jpg), tượng ngựa trên thùng xe [img_028](../../../data/images/img_028.jpg), mô hình ô tô và búp bê [img_038](../../../data/images/img_038.jpg), thú bông trên xe [img_019](../../../data/images/img_019.jpg). Xe thật phía sau hàng rào [img_018](../../../data/images/img_018.jpg) vẫn vẽ. Tượng [img_027](../../../data/images/img_027.jpg) không cần `ignore_region` vì trong ảnh không có xe thật để bỏ sót.
- **H-06. Phản chiếu và thân xe gắn camera.** Không vẽ vật nằm trong gương [img_020](../../../data/images/img_020.jpg), vũng nước [img_013](../../../data/images/img_013.jpg), [img_015](../../../data/images/img_015.jpg), [img_031](../../../data/images/img_031.jpg), hoặc kính cửa [img_033](../../../data/images/img_033.jpg). Vỏ gương và phần xe của camera không phải một xe khác.
- **H-07. Hàng hóa không nới box.** Box người không gồm quang gánh [img_003](../../../data/images/img_003.jpg). Box xe máy không gồm thanh kim loại thò ra [img_029](../../../data/images/img_029.jpg). Box bán tải gồm cabin và thùng, không vẽ từng túi đồ [img_012](../../../data/images/img_012.jpg), và không kéo lên tượng ngựa [img_028](../../../data/images/img_028.jpg).
- **H-08. Ngoài năm lớp phương tiện và người.** Xe buýt ([img_009](../../../data/images/img_009.jpg), [img_031](../../../data/images/img_031.jpg), [img_037](../../../data/images/img_037.jpg), [img_040](../../../data/images/img_040.jpg)), xe đạp, xích lô ([img_006](../../../data/images/img_006.jpg), [img_023](../../../data/images/img_023.jpg)), tuk-tuk ([img_004](../../../data/images/img_004.jpg), [img_016](../../../data/images/img_016.jpg), [img_040](../../../data/images/img_040.jpg)), xe lam ba bánh ([img_035](../../../data/images/img_035.jpg)) không gán vào `car`, `van`, `truck` hay `motorcycle`. Vẽ `person` cho người nhìn thấy. Khoanh `ignore_region` quanh thân xe và hàng gắn trên xe đó, không khoanh lên người. Chó [img_009](../../../data/images/img_009.jpg) không gán.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| [img_002](../../../data/images/img_002.jpg) | Xe đầu kéo chở nhiều ô tô trên hai tầng | H-04 | 1 `truck` cho cả đoàn xe. Mỗi ô tô còn nhận ra là 1 `car`. Người ở đuôi xe là `person`. | Ô tô trên sàn vẫn có hình ô tô. Rơ-moóc không phải một xe thứ hai. |
| [img_003](../../../data/images/img_003.jpg) | Người gánh quang hàng to hơn người | H-07 | Box `person` chỉ ôm người. Không ôm rổ. | Rổ không phải người. Kéo box theo rổ làm sai kích thước. |
| [img_004](../../../data/images/img_004.jpg) | Hai tuk-tuk, người ngồi trong cabin hở | H-03, H-08 | `person` cho người ngồi. `ignore_region` cho từng thân tuk-tuk. | Không có lớp tuk-tuk. Người thì có thật và đang ở trên đường. |
| [img_009](../../../data/images/img_009.jpg) | Minibus nhiều cửa sổ, người và xe máy nhỏ hai bên | H-01, H-08 | `ignore_region` cho minibus. Xe máy và người cao từ 20 px thì vẽ riêng. Không vẽ chó. | Minibus không phải `van`. Người sau kính xe không tách ra. |
| [img_011](../../../data/images/img_011.jpg) | Tượng hoặc mô hình sinh vật lớn cạnh xe máy độ | H-05 | Không vẽ `person` cho hình sinh vật. Xe máy nào còn đủ bánh và khung thì `motorcycle`. Cần hỏi reviewer. | Nhìn gần giống tượng hơn người. Nếu bên trong là người mặc đồ, quyết định này phải đổi khi họp. |
| [img_019](../../../data/images/img_019.jpg) | Xe vàng diễu hành, người lái, thú bông, đám đông sau rào | H-01, H-03, H-05 | Xe vàng là `car`, tick `can_review`. Người lái là `person`. Thú bông không vẽ. Đám tách được từng người thì từng `person`. Chỗ dính thành khối thì một `ignore_region`. | Xe có bánh và đang chạy. Vỏ xe là đồ hóa trang nên lớp `car` dễ bị cãi. |
| [img_020](../../../data/images/img_020.jpg) | Gương chiếu hậu, trong gương có một ô tô | H-06 | Không vẽ xe trong gương. Không vẽ vỏ gương thành một xe. | Ảnh trong gương không nằm ở vị trí thật trước camera. |
| [img_021](../../../data/images/img_021.jpg) | Dòng xe máy rất đông, càng xa càng nhỏ | H-01, H-02 | Xe và người ở gần, tách được và cao từ 20 px: từng cặp `person` + `motorcycle`. Đám xa không tách được: một `ignore_region`. | V0 không nói lúc nào phải tách, lúc nào được gom. |
| [img_026](../../../data/images/img_026.jpg) | Hai người trên một xe tay ga | H-02 | 2 `person` + 1 `motorcycle`. Box được chồng ở chân và yên. | Đếm người và xe là hai việc khác nhau. |
| [img_027](../../../data/images/img_027.jpg) | Tượng đá hình bán tải trong nghĩa trang | H-05 | Không vẽ box. | Hình xe nhưng không phải xe chạy. |
| [img_028](../../../data/images/img_028.jpg) | Bán tải chở tượng ngựa cao hơn cabin | H-05, H-07 | 1 `truck` ôm cabin và thùng. Không vẽ ngựa. SUV đỏ phía sau là `car`. | Ngựa là tượng. Thùng xe vẫn là dấu hiệu của `truck`. |
| [img_029](../../../data/images/img_029.jpg) | Xe máy và xe đạp khiêng thanh kim loại dài | H-02, H-07 | Người lái xe máy là `person`, xe là `motorcycle`, box không chạy theo thanh sắt. Người đạp xe là `person`, xe đạp không có lớp. | Thanh sắt là hàng, không phải khung xe. |
| [img_033](../../../data/images/img_033.jpg) | Ma-nơ-canh, ảnh người in trên kính, gương xe | H-05, H-06 | Không vẽ `person`. Không vẽ xe từ hình trong gương. | Không có người thật hay xe thật trong khung chính. |
| [img_035](../../../data/images/img_035.jpg) | Xe ba bánh chất bao cao, cạnh xe máy và ô tô | H-07, H-08 | `ignore_region` bao thân xe ba bánh và đống bao. Người lái nếu thấy mặt là `person`. Xe máy là `motorcycle` cộng `person`. Ô tô trắng là `car`. | Xe ba bánh ngoài taxonomy. Đống bao không phải `truck`. |
| [img_037](../../../data/images/img_037.jpg) | Pickup có nắp thùng cứng, phía sau là xe buýt | H-08 | Pickup là `truck`, tick `can_review` vì nắp thùng trông giống `van`. Xe buýt là `ignore_region`. Người đi bộ là `person`. | Nắp cứng vẫn là nắp của thùng riêng, không phải thân van liền. |
| [img_038](../../../data/images/img_038.jpg) | Tủ đồ chơi toàn ô tô mô hình và búp bê | H-05 | Không vẽ box nào. | Không có xe thật hay người thật. |

## 5. QC và phiên bản

Ngưỡng lấy từ mục `rules` của taxonomy, không đổi ở bản cá nhân:

- Chiều cao box dưới 20 px thì không vẽ riêng.
- IoU khi so hai người gán: vật cao từ 50 px dùng 0,85. Vật thấp hơn dùng 0,70.
- Đồng thuận lớp mục tiêu 0,95. Tỉ lệ ảnh lỗi tối đa 0,02.

Ca mơ hồ mà đã có box thì tick `can_review`. Ca không có box, hoặc ca ngoài lớp, ghi ở mục 4 để họp Pha 2. Người điều phối chốt khi họp. Bản này chưa phải bản khóa.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0.1 | Bản đề xuất cá nhân | Khởi tạo từ v0 và phân tích 40 ảnh |
