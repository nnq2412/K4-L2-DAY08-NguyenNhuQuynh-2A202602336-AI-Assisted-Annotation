# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyễn Như Quỳnh

Công cụ gán nhãn đã dùng: CVAT, SAM và sửa file nhãn trực tiếp

Sao chép file này thành `reports/REPORT.md` rồi trả lời các câu hỏi. Mọi con số phải truy được
từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc
`outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

Camera đứng một chỗ, một chiếc xe sẽ xuất hiện liên tục trong khung hình vài giây. Ảnh học (train) và ảnh kiểm tra (test) phải được chia theo trục thời gian và cách nhau một khoảng nhất định. Nếu chia ngẫu nhiên, cùng một chiếc xe ở cùng một góc độ có thể vừa nằm trong tập học vừa nằm trong tập kiểm tra. Khi đó, điểm kiểm thử sẽ cao giả tạo (lệch theo hướng tích cực) vì mô hình chỉ học thuộc lòng (overfit) đặc điểm của chiếc xe đó thay vì học được cách nhận diện xe nói chung.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

- Vòng 0: AP50 = 0.771.
- Độ phủ theo kích thước: xe nhỏ (R small) = 0.182, xe vừa (R medium) = 0.547, xe lớn (R large) = 0.561.
Số liệu này cho thấy mô hình bỏ sót rất nhiều xe ở xa (kích thước nhỏ) so với xe ở gần.
Nhãn tham chiếu dùng để chấm điểm cũng do máy (mô hình lớn hơn) vẽ ra và chưa có sự kiểm tra của con người. Do đó, có những trường hợp như các đốm sáng mờ (đèn xe ở rất xa) hoặc xe bị khuất ở mép ảnh mà mô hình khởi đầu lạnh nhận sai hoặc nhận đúng nhưng lệch với nhãn test. Cần có người rà lại nhãn tham chiếu ở tập test trước khi kết luận chắc chắn rằng mô hình sai.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

Công thức tính điểm (score) là sự kết hợp của độ bất định của mô hình (U - chiếm 50%), độ lưỡng lự khi vẽ chồng chéo nhiều khung (A - 30%) và khoảng cách thời gian (D - 20%).
`MIN_GAP_S` có vai trò quan trọng: đảm bảo hai ảnh được chọn phải cách nhau ít nhất một khoảng thời gian (ví dụ 2 giây). Vì camera cố định, các ảnh quá sát giờ nhau sẽ gần như giống hệt nhau, không mang lại kiến thức mới.
Ví dụ: 3 ảnh frame_0182.jpg, frame_0099.jpg, và frame_0107.jpg đều có điểm trên 0.88 vì AI vẽ nhiều khung nhưng còn phân vân. Ảnh frame_0372.jpg có điểm rất cao (0.910), nhưng bị hệ thống bỏ qua vì nó sát giờ với frame_0369.jpg; nếu sửa cả hai sẽ tốn công gán nhãn mà mô hình ít học thêm được gì mới.
Điểm cao chỉ có nghĩa là AI đang phân vân, chứ không bảo đảm sửa xong ảnh đó AI sẽ thông minh hơn, nhất là khi nguyên nhân phân vân đến từ nhiễu hoặc vật thể quá nhỏ ngoài tầm nhận diện.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
  `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

- Trong 12 ảnh vòng 1, tôi đã: giữ nguyên (accepted) 155 box, sửa (edited) 6 box, xóa (deleted) 8 box, và thêm mới (added) 158 box. 
- Điểm AP50 của vòng 1 (0.503) đã giảm đáng kể -0.268 so với cold start (0.771).
- Theo kích thước xe: R_small giảm từ 0.182 xuống 0.000, R_medium giảm từ 0.547 xuống 0.122, R_large giữ nguyên ở 0.561.
Điều này xảy ra có thể là do việc ta thêm tới 158 khung (phần lớn là xe rất nhỏ/xa mà pre-label bỏ sót), làm lệch phân phối so với tập test (tập test chưa được người sửa tay nên vẫn đang thiếu các khung nhỏ đó). Khi mô hình học được cách nhận diện thêm các xe nhỏ này, nó có thể vẽ ra rất nhiều box ở tập test, nhưng vì tập test không có các box đó ở nhãn tham chiếu, nó bị tính là False Positive, làm giảm AP50. 
Một ca khó tiêu biểu: Xe bị khuất ở mép khung hình (chỉ lộ một phần đèn và cản trước). Theo guideline, ta kéo khung ôm sát phần nhìn thấy, kể cả đèn.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Kết quả vòng 1 thấp hơn rõ rệt so với cold start. Tôi quyết định dừng ở đây thay vì làm tiếp vòng 2.
Hai ca còn yếu là: xe máy chạy sát lề và ô tô quá xa ở cuối đường. Chi phí rà nhãn cho các ca này rất cao vì xe quá nhỏ, dễ nhầm với nhiễu, nguy cơ chọn phải ảnh gần trùng cảnh là rất lớn.
Việc tập kiểm thử chỉ có 20 ảnh và nhãn tham chiếu là nhãn tự động chưa qua kiểm duyệt thủ công đã bóp méo hoàn toàn kết quả đánh giá. Khi con người gán nhãn rất chi tiết (thêm 158 box) ở tập học, nhưng tập test lại chưa được sửa tương ứng, thì nỗ lực của ta lại làm điểm số giảm đi. Do đó, nếu AP50 giảm, điều kiện tiên quyết trước khi train thêm vòng 2 là phải tiến hành rà soát lại (kiểm tra chéo) toàn bộ nhãn tham chiếu của 20 ảnh kiểm thử test set.
