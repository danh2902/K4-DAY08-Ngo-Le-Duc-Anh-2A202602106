# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có
ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét
ảnh gần trùng hoặc trường hợp model không dự đoán được box: Nếu chỉ được sửa 5 ảnh, tôi chọn frame_0182.jpg (hạng 1, điểm 0.959), frame_0369.jpg (hạng 2, điểm 0.932), frame_0380.jpg (hạng 3, điểm 0.917), frame_0326.jpg (hạng 4, điểm 0.916) và frame_0312.jpg (hạng 7, điểm 0.915). Năm ảnh này đứng đầu danh sách. frame_0326.jpg và frame_0331.jpg, frame_0369.jpg và frame_0372.jpg cách nhau khoảng 2 giây, cảnh gần giống nhau, nên tôi không lấy cả hai.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet: frame_0182.jpg, frame_0369.jpg, frame_0380.jpg — cả 3 đều có selected = TRUE và nằm trong nhóm có score cao nhất trong CSV.
Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do: frame_0372.jpg có score = 0.9101, đứng hạng 6 nhưng selected = FALSE; vẫn nên xem để kiểm tra vì điểm cao nhưng bị loại khỏi sample, giúp đánh giá cơ chế chọn có bỏ sót các frame đáng chú ý hay không

Điều phép chọn này chưa chứng minh về chất lượng mô hình: Việc chọn frame dựa trên score chỉ cho biết frame nào được ưu tiên kiểm tra, không chứng minh model có độ chính xác cao. Muốn đánh giá chất lượng model cần đối chiếu prediction với ground truth và tính các metric như Precision, Recall, IoU, mAP...
