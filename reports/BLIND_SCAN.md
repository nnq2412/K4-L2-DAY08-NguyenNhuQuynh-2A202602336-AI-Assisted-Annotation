# Quét độc lập trước khi xem pre-label

Frame: frame_0312.jpg

Số xe nhìn thấy bằng mắt: 24

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe: Các xe ở khoảng cách rất xa (cả hai chiều): 
Hình dáng khung xe hoàn toàn bị chìm trong bóng tối, chỉ còn lại các chấm sáng từ đèn pha hoặc đèn hậu nhỏ xíu. AI rất dễ bỏ sót các vật thể này do thiếu đặc trưng về mặt hình học của xe.   
Chiếc xe ngược chiều gần nhất bên trái có đèn pha quá lóa, quầng sáng (glare) che khuất viền xe thật cũng là một rủi ro khiến AI vẽ box to hơn thực tế.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
