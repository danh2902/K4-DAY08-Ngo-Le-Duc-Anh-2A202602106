# Quét độc lập trước khi xem pre-label

Frame: frame_0227 tên một ảnh trong `to_label/round1/images/train/`

Số xe nhìn thấy bằng mắt: 26

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe: xe ở xa, khổng đủ ánh sáng, chỉ thấy đèn hậu và bị che khuất nên dễ bị bỏ sót hoặc vẽ sai. Các xe che khuất lẫn nhau hoặc có thiết kế đèn hậu đặc biệt( khồng chỉ có 2 đèn hậu màu đỏ ở đuôi xe) nên sẽ dễ bị vẽ sai hoặc bỏ sót

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
