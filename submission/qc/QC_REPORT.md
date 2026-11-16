# Báo cáo QC — Nhom01

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn 2D bounding box phục vụ huấn luyện mô hình phát hiện người và phương tiện; giới hạn độ đại diện cho dữ liệu thực tế nội đô và ngoại thành.
- Nguồn và phép sử dụng dữ liệu: Xem chi tiết trong ATTRIBUTION.csv, toàn bộ 40 ảnh có bản quyền Creative Commons hoặc Public Domain hợp lệ.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256: 66406ee96213215e28484ec44fd077f21201ddefe53d67fced670cf80bd9bc35); labels_calibrated.json (SHA-256: bbdc61a66d36ff1c2829e88f1da35259ab99efbfef9246538cafd79bd9be6a3d).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán dùng Guideline v1.0 và Taxonomy v1.0; khi bàn giao dùng Guideline v1.1 sau hiệu chuẩn Calibration và giữ nguyên Taxonomy v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (đạt 100%).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1068 cặp box ghép, IoU trung bình 0.963 (ngưỡng >= 0.85), đồng thuận lớp 100.0% (ngưỡng >= 95%), tính trên mẫu số 1068 cặp box ghép giữa 5 người gán.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 2 box chưa ghép (do ảnh img_020 có box phản chiếu ảo), 0 box bị loại do ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ảnh img_020.jpg (ảnh phản chiếu cầu vồng và xe trong gương chiếu hậu, hai người có thể cùng vẽ box xe nhưng quy tắc R-06 cấm vẽ hình ảnh ảo).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Xử lý 17 ca (C001 đến C017) theo calibration_decisions.csv, áp dụng Rule R-02, R-05, R-06; người xác nhận 244.
- Tác động v1.1 lên nhãn và ảnh: Bản v1.1 làm rõ quy tắc R-06 (xóa box xe phản chiếu trong gương xe), tinh chỉnh lại ranh giới box người đi bộ ôm sát hơn, phạm vi tác động cấp ảnh (ảnh flagged).
- Tham chiếu bằng chứng khi Rework: Đạt chuẩn ngay vòng 1, không phát sinh vòng Rework.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 272, 2026-11-16
- Đã duyệt / cả lô: 40/40 ảnh (100% toàn bộ lô).
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (ngưỡng tối đa cho phép là 2.0%).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/356 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó duyệt 6/6 ảnh (0 lỗi), phần còn lại duyệt 34/34 ảnh (0 lỗi), không có lỗi lặp lại.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy tự động chặn được lỗi kích thước box nhỏ hơn 20px hoặc vượt mép ảnh; người thật cần đọc ảnh để phân biệt giữa tượng đá (img_017, img_027) hoặc ma-nơ-canh (img_033) với người/phương tiện thật.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Dữ liệu đạt 100% tiêu chuẩn chất lượng QC, tỉ lệ lỗi 0.0% (dưới ngưỡng 2%), IoU 0.963 vượt ngưỡng 0.85; bàn giao cho đội ngũ huấn luyện mô hình ADAS/CCTV.
- Nếu còn việc: Không còn tồn đọng, toàn bộ 40 ảnh đã được nghiệm thu hoàn chỉnh.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C011/C012 tại img_020.jpg, đối chiếu Rule R-06, kết quả box phản chiếu đã được loại bỏ hoàn toàn trong labels_calibrated.json; kiểm tra chéo img_011.jpg đối chiếu Rule R-05 nhân vật cosplay gán person chính xác.
