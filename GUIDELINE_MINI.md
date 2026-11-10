# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |Phương tiện cơ giới chở người, thường có 4 bánh, thiết kế chủ yếu phục vụ nhu cầu di chuyển cá nhân.|Sedan, SUV, hatchback, coupe. Không nhầm với van có khoang chở hàng riêng biệt. |
| `truck` |Phương tiện cơ giới chuyên dùng để vận chuyển hàng hóa, thường có cabin và thùng hoặc khoang chở hàng.|Xe tải thùng, xe ben, xe tải mui bạt. |
| `bus` |Phương tiện vận tải hành khách có kích thước lớn, thiết kế với khoang hành khách rộng và nhiều chỗ ngồi. |Xe buýt công cộng, xe khách, xe buýt trường học.|
| `van` |Phương tiện có thân xe dạng hộp, kích thước thường nhỏ hơn bus, có khoang chở người hoặc hàng hóa phía sau cabin.|Minivan, cargo van, passenger van.|

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding Box phải bao quanh toàn bộ phần vật thể nhìn thấy, sát các cạnh ngoài của phương tiện, hạn chế chứa nền hoặc vật thể khác.
- Vật thể bị che khuất: Vẫn gán nhãn nếu có đủ đặc điểm nhận diện lớp. Box chỉ bao quanh phần vật thể thực sự nhìn thấy, không suy đoán phần bị che khuất. Nếu vật bị che khuất quá 50% thì bỏ qua
- Vật thể bị cắt bởi rìa ảnh: Gán nhãn nếu phần còn lại đủ thông tin để nhận diện. Box kéo sát đến rìa ảnh, không mở rộng ra ngoài khung hình. Nếu như vật thể bị cắt còn dưới 50% thì bỏ qua
- Vật thể rất nhỏ/mơ hồ: Chỉ gán nhãn khi có đủ bằng chứng hình ảnh để xác định lớp. Nếu không thể phân biệt đáng tin cậy thì bỏ qua, không đoán nhãn. Nếu như vật thể có kích thước nhỏ dưới khoảng 20px mỗi chiều thì bỏ qua, vật thể bị mờ nhưng vẫn còn nhận ra được là xe gì thì gán nhãn nhưng với vật thể mà xe chỉ còn 1 khối màu thì bỏ qua.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_003` |Xe ở quá xa, kích thước nhỏ, khó quan sát đặc điểm phương tiện.|Không gán nhãn nếu không đủ bằng chứng xác định lớp. Nếu vẫn nhận diện rõ loại xe thì gán nhãn bình thường.|Schema yêu cầu xác định đúng class dựa trên đặc điểm hình ảnh. Không suy đoán khi vật thể quá nhỏ hoặc mơ hồ. |
| `drive_004` |Ảnh ban đêm, xuất hiện nhiều đèn xe xung quanh, khó xác định vật thể có phải xe hay không.|Không gán nhãn các vùng chỉ có ánh sáng hoặc đèn xe. Chỉ gán nhãn khi quan sát được đủ đặc điểm của phương tiện.|Đèn xe không phải là một đối tượng thuộc các class car, truck, bus, van. Cần có bằng chứng nhận diện phương tiện.|
| `drive_005` |Nhiều xe nối đuôi nhau trong dòng giao thông đông đúc, các xe bị che khuất một phần bởi xe phía trước hoặc bị cắt mép ảnh.|Vẽ box riêng cho từng xe ôm sát phần thân xe nhìn thấy được (visible box). Bỏ qua các xe bị che khuất trên 50% hoặc không đủ đặc điểm nhận diện lớp.|Quy định schema yêu cầu mỗi phương tiện là một box riêng biệt, box ôm sát phần nhìn thấy và không suy đoán phần bị che khuất hay phần nằm ngoài khung hình.|
