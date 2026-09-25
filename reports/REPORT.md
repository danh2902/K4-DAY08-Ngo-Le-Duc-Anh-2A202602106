# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Ngô Lê Đức Anh

Công cụ gán nhãn đã dùng: CVAT (AnyLabeling, CVAT, SAM hoặc sửa trực tiếp file nhãn)

Sao chép file này thành `reports/REPORT.md` rồi điền vào các chỗ ĐIỀN. Mọi con số phải truy được
từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc
`outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

 camera đứng một chỗ, một chiếc xe nằm trong hình vài giây. Ảnh học và ảnh kiểm tra phải cách nhau theo thời gian. Nếu trộn ngẫu nhiên, cùng một xe có thể vừa được AI học vừa được dùng để chấm. Điểm sẽ đẹp hơn sự thật.


## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
Số chính là điểm khớp khung `0.771`. Xe nhỏ chỉ được tìm thấy khoảng `0.182`, xe vừa `0.547`, xe lớn `0.561`. Nghĩa là xe ở xa bị bỏ sót nhiều hơn xe ở gần. Khung Ai bị lệch ở chiếc xe trên cùng khi khung khoanh vào 2 xe một lúc. Nhãn dùng để chấm cũng do máy vẽ, chưa có người xem từng khung, nên có thể nhãn chấm sai chứ không phải AI của bạn sai.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

mỗi ảnh có một điểm. Một nửa điểm là AI không chắc. Ba phần mười là AI vẽ nhiều khung còn lưỡng lự. Hai phần mười là ảnh có khác thời gian với ảnh khác. Hai ảnh trong cùng lô phải cách nhau ít nhất 2 giây, vì camera đứng yên, ảnh sát nhau gần như giống hệt. frame_0182.jpg, frame_0369.jpg, frame_0380.jpg, frame_0372.jpg.  điểm cao không có nghĩa sửa ảnh đó sẽ làm AI giỏi hơn.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
  `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

| vòng | model | ảnh train | box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 320 | 0.747 | -0.024 | 0.979 | 0.226 | 0.367 | 0.000 | 0.196 | 0.805 |
Giữ nguyên (accepted): 118 khung. Kéo/chỉnh lại (edited): 31 khung. Xóa khung sai (deleted – FP): 20 khung. Thêm khung còn thiếu (added – FN): 171 khung. AP50 giảm 0.024. Nhóm xe lớn (large) tốt lên rõ rệt, Recall tăng từ 0.561 → 0.805. Ngược lại, xe nhỏ và trung bình xấu đi, đặc biệt nhóm medium giảm 0.351, còn small giảm 0.182 xuống 0.
Frame 0150: một xe ở làn bên phải được kéo/chỉnh lại box so với cold start.

Quan sát độc lập cho thấy dữ liệu có các trường hợp khó như xe ở xa, thiếu sáng và bị che khuất. Kiểm tra pre-label cho thấy model ban đầu bỏ sót 171 box, cần chỉnh 31 box và xóa 20 box. Tuy nhiên, sau khi train vòng 1, AP50 giảm từ 0.771 xuống 0.747 và Recall giảm mạnh, nên chưa thể kết luận việc thêm dữ liệu đã cải thiện model. Cần kiểm tra lại các box đã sửa trước khi tiếp tục train. frame_0250.jpg – xe bị khuất ở góc dưới bên phải: AI khoanh thiếu thân xe. Theo guideline, đây là trường hợp xe bị che khuất một phần nên phải vẽ box bao quanh phần thân xe nhìn thấy, không chỉ phần đèn; do đó box được chỉnh lại cho sát với vùng xe quan sát được.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Đánh giá vòng 1 so với vòng 0
AP50: 0.771 → 0.747, giảm 0.024.
Precision @0.25: 0.925 → 0.979, tăng 0.054.
Recall @0.25: 0.489 → 0.226, giảm 0.263.
F1: 0.640 → 0.367, giảm 0.273.
Recall xe lớn: 0.561 → 0.805, tăng 0.244.
Recall xe trung bình: 0.547 → 0.196, giảm 0.351.
Recall xe nhỏ: 0.182 → 0, giảm 0.182.
Quyết định: Dừng để kiểm tra lại trước khi làm tiếp

AP50 và F1 đều giảm, đặc biệt Recall giảm mạnh. Vì vậy chưa nên tiếp tục cho AI học thêm ngay. Cần xem lại các khung đã sửa và các box được thêm trước khi tạo vòng tiếp theo.

Hai điểm còn yếu:

Xe ở xa/quá nhỏ, đôi khi chỉ còn hai chấm đèn nên rất khó xác định chính xác box. Phần chấm hiện chỉ có 20 ảnh, xe quá nhỏ không tính và các nhãn chấm chưa được người kiểm, nên chưa đủ cơ sở để kết luận model xử lý nhóm này tốt.
Xe bị cắt mép ảnh hoặc chỉ xuất hiện một phần nên box dễ bị thiếu hoặc sai vị trí, cần thêm thời gian để sửa thủ công.

Sửa thêm thì mất thời gian, và không nên chọn hai ảnh sát nhau vì chúng gần như một cảnh. Phần chấm chỉ có 20 ảnh, xe quá nhỏ không tính, nhãn chấm chưa được người kiểm. Nếu điểm giảm, hãy xem lại khung bạn đã sửa trước khi cho AI học thêm.