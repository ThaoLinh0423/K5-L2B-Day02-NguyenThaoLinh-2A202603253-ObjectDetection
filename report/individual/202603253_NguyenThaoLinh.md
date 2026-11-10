# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Thảo Linh
- MSSV: 2A202603253
- Email: nguyenlinhthao666@gmail.com
- Lớp: 2B
- Link repo bài làm: https://github.com/ThaoLinh0423/K5-L2B-Day02-NguyenThaoLinh-2A202603253-ObjectDetection.git
- Link Colab đã chạy: https://colab.research.google.com/drive/1f-cbI7SWAPGLQn06Cn5MqeGX1o0qnuxt?usp=sharing
## 2. Schema và mini guideline
**Tóm tắt ba ca mơ hồ đã quyết định:**
1. **`drive_003` (Vật thể ở quá xa, kích thước nhỏ/mờ):** Xe ở khoảng cách xa, kích thước dưới ~20px mỗi chiều hoặc chỉ là khối màu mờ. Quyết định: Không đoán nhãn, chỉ gán nhãn khi có đủ bằng chứng hình ảnh xác định đúng class; nếu không đủ đặc trưng thì kiên quyết bỏ qua.
2. **`drive_004` (Ảnh ban đêm, chùm đèn pha/đèn hậu gây nhiễu):** Xuất hiện nhiều quầng sáng, vệt sáng đèn xe chiếu xuống mặt đường. Quyết định: Chỉ gán bounding box bao quanh phần thân xe thực sự nhìn thấy, tuyệt đối không gán nhãn cho các vùng chỉ có ánh sáng đèn hoặc bóng sáng.
3. **`drive_005` (Giao thông đông đúc, che khuất và cắt khung):** Các xe nối đuôi nhau san sát, bị che khuất một phần bởi xe phía trước hoặc bị mép ảnh cắt ngang. Quyết định: Mỗi xe là một bounding box riêng biệt, ôm sát phần thân xe thực tế nhìn thấy được (visible box), không suy đoán phần bị che lấp; bỏ qua xe bị che khuất trên 50%.

**Cách áp dụng schema:** Áp dụng nghiêm ngặt 4 lớp (`0: car, 1: bus, 2: van, 3: truck`). Phân biệt xe con cá nhân (`car`) với xe tải chở hàng có thùng/khoang riêng (`truck`), xe khách kích thước lớn (`bus`), và xe dạng hộp thùng liền (`van`).

## 3. Annotation và self-QC

- Số file nhãn train: 50 file.
- Kết quả checker và lỗi đã xử lý:
  - Ban đầu khi chạy `python tools/check_yolo_labels.py`, phát hiện cả 50 file ảnh đều qua.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_004`, `drive_005`, `drive_009`, `drive_015`, `drive_024`, `drive_027`, `drive_030`, `drive_041`, `drive_048`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Phát hiện một số xe ở xa trong ảnh mật độ đông (`drive_048`, `drive_041`) bị bỏ sót; đã bổ sung box nếu xe trên 50% đặc trưng nhận diện.
  - Một số box ban đầu vẽ hơi lỏng bị dính mặt đường và cả phần bóng đổ của xe; đã thu hẹp lại để box ôm sát viền thân xe nhìn thấy.
  - Rà soát kỹ để đảm bảo không gán nhầm người đi bộ, mô tô hoặc cột biển báo.
- Commit khóa nhãn ban đầu: `08245f0` (Commit "Gán nhãn 50 ảnh" trước khi mở Gold labels).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 514 | - |
| FP | 106 | - |
| FN | 139 | - |
| Precision | 0.8290 | - |
| Recall | 0.7871 | - |
| F1 | 0.8075 | - |
| mean_iou_of_matches | 0.9313 | - |
| passed | true | - |

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
  - Môi trường: Google Colab
  - Phần cứng: GPU Tesla T4 (device: "0")
  - Phiên bản: Ultralytics mới nhất (YOLO26 / YOLOv8 architecture), PyTorch hỗ trợ CUDA.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên toàn bộ cấu hình chuẩn của notebook 
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3611 |
| Recall | 0.3660 |
| mAP50 | 0.3686 |
| mAP50–95 | 0.2594 |
Trả lời bốn câu hỏi cuối notebook: 

1.Lớp nào recall thấp nhất? Ảnh nào minh họa?
   - Lớp có Recall thấp nhất là `bus` và `truck`. Nguyên nhân trực tiếp là do sự mất cân bằng dữ liệu cực kỳ lớn: Trong 50 ảnh train có tới hơn 80% xe `car`.
   - Minh họa: Trong ảnh drive_051.jpg, có một chiếc xe buýt lớn chạy ở làn trung tâm nhưng mô hình nhận diện nhầm thành `car` hoặc bỏ sót do chưa học đủ đặc trưng tổng quát của `bus` từ tập train nhỏ.

2.Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
   - Chỉ số mAP50 chỉ yêu cầu ngưỡng chồng lấn IoU 0.50. Trong khi đó, mAP50-95 tính trung bình trên các ngưỡng IoU khắt khe từ 0.50 đến 0.95 với bước nhảy 0.05.
   - Nếu mAP50 cao nhưng mAP50-95 thấp, điều đó chứng tỏ mô hình định vị được vật thể nhưng chất lượng vẽ biên của bounding box chưa khít, box bị vẽ quá rộng hoặc quá hẹp. Về mặt dữ liệu, cần kiểm tra lại độ bám sát của box trong tập gán nhãn train để đảm bảo box ôm sát phần biên nhìn thấy của phương tiện.

3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
   - Lỗi nhầm lẫn giữa `car`-`van` và `van`-`truck`.
   - Dữ kiện hỗ trợ:
     - Dưới góc máy camera CCTV đặt trên cao nhìn nghiêng, các xe minivan/van chở hàng có góc nhìn từ phía sau hoặc chéo rất khó phân biệt với dòng SUV/hatchback lớn (`car`) hoặc xe tải nhỏ thùng kín (`truck`).
     - Ma trận nhầm lẫn cho thấy nhiều xe `van` bị dự đoán nhầm thành `car`. Điều này phản ánh sự kết hợp giữa: guideline cần bổ sung thêm tiêu chí nhận diện đặc thù cho xe van ở góc nhìn từ trên cao và bias của mô hình nghiêng về lớp chiếm đa số tuyệt đối là `car`.

4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
   -  Đây là nguyên tắc nền tảng của Machine Learning nhằm ngăn chặn hiện tượng rò rỉ dữ liệu và overfitting. Việc giữ lại tập dữ liệu chưa từng thấy là bắt buộc để đánh giá khách quan khả năng tổng quát hóa thực tế của mô hình.
   - Vai trò validation trong notebook 10 ảnh eval đóng vai trò tập kiểm định :
     + Được dùng để theo dõi độ hội tụ và hiệu năng mô hình qua từng epoch.
     + Làm thước đo để tự động chọn ra checkpoint tốt nhất dựa trên mAP.
     + Giúp người phát triển quan sát được tác động trực tiếp của chất lượng nhãn dữ liệu lên chất lượng dự đoán của mô hình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Annotation | Các xe ở xa và xe trong cảnh đêm (`drive_003`, `drive_004`) rất khó xác định ranh giới và loại xe | Đặt ngưỡng lọc: chỉ gán box khi kích thước 20px và nhìn rõ kết cấu thân xe; loại bỏ quầng sáng đèn | Nhãn nhất quán hơn, đạt F1 = 0.8075 trên gold set |
| Huấn luyện Colab | Dữ liệu mất cân bằng nặng, dẫn đến recall các lớp `bus`, `truck` thấp | Giữ nguyên split chuẩn và seed thực nghiệm để phân tích khách quan hạn chế của tập dữ liệu nhỏ | Đã xuất đầy đủ chỉ số và biểu đồ phục vụ báo cáo |

## 7. Tự đánh giá theo rubric

Đối chiếu theo [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10đ): Đạt. Đủ 50 file nhãn train, chuẩn 5 cột, normalized, đúng 4 class ID, vượt qua kiểm tra cấu trúc `check_yolo_labels.py`.
- Đầy đủ vật thể (20đ): Đạt. Gán đầy đủ các xe thuộc 4 lớp trong tầm nhận diện, xử lý các ca che khuất/cắt góc theo đúng quy tắc.
- Độ bám sát và hình học box (20đ): Đạt. Box ôm sát phần vật thể nhìn thấy, không lấn nền, đạt mean IoU trên gold set là 0.9313.
- Phân lớp nhất quán (15đ): Đạt. Phân biệt đúng 4 lớp `car`, `truck`, `bus`, `van`, không gán vật thể ngoài phạm vi bài lab.
- Mini guideline và quyết định mơ hồ (10đ): Đạt. Đã điền đầy đủ định nghĩa lớp, quy tắc vẽ box và phân tích 3 ca mơ hồ (`drive_003`, `drive_004`, `drive_005`) trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
- Gold-set annotation QC (15đ): Đạt. F1 = 0.8075, mean IoU = 0.9313, trạng thái `passed: true`.
- Pipeline YOLO và phân tích (10đ): Đạt. Đã huấn luyện YOLO26n trên Colab, không dùng eval để train, xuất đầy đủ metrics/ảnh dự đoán và trả lời chi tiết 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng AI Assistant để hỗ trợ sắp xếp cấu trúc thư mục nộp bài, và rà soát các câu trả lời phân tích kỹ thuật. Toàn bộ quá trình gán nhãn, kiểm tra ảnh bằng mắt và chạy thực nghiệm Colab do học viên tự thực hiện và đối chiếu.
