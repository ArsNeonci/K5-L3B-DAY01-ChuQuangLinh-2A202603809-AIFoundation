# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác

**SEED:** 7353

## 1. Bằng chứng

| Tệp | Trạng thái |
| --- | --- |
| [day01_lab.ipynb](day01_lab.ipynb) | Có notebook và output; các hàm LLM đã kiểm tra bằng giả lập, chưa gọi API thật |
| [lab1_report.json](lab1_report.json) | Có 15/15 kiểm tra đạt, 10 dự đoán và kết quả của 5 mô hình cục bộ |
| [hand_calc.jpg](hand_calc.jpg) | Đã bổ sung ảnh bài tính tay |

## 2. Kết quả chính

Dữ liệu sau khi bỏ trùng gồm 5.169 tin, trong đó có 4.516 ham và 653 spam. Tập train có 3.619 tin, validation và test mỗi tập có 775 tin. SUB gồm 40 tin cân bằng giữa hai nhãn.

| Phương án | F1 test | F1 SUB 40 | Train (s) | Tin/s cả lô | p95 (ms/tin) | USD/1.000 tin qua API |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| BoW + LogReg | 0,895 | 0,824 | 2,081 | 26.760,96 | 0,071 | 0 |
| BoW + MLP | 0,946 | 0,947 | 5,629 | 52.958,86 | 0,311 | 0 |
| MiniLM + kNN | 0,947 | 0,974 | 40,932 | 33,06 | 60,204 | 0 |
| MiniLM + LogReg | 0,941 | 0,947 | 41,426 | 61,52 | 30,919 | 0 |
| MiniLM + MLP head | 0,973 | 0,974 | 42,411 | 63,07 | 32,971 | 0 |

Các số đo trên được lấy từ lần chạy cục bộ. Chi phí API bằng 0 không có nghĩa là vận hành miễn phí, vì vẫn cần máy tính và tài nguyên xử lý. Thời gian chuẩn bị của các cách dùng MiniLM gồm bước encode tập train và huấn luyện đầu phân loại nếu có.

MiniLM + MLP head có F1 test cao nhất trong lần đo này. Tuy nhiên, SUB chỉ có 40 tin nên khoảng cách vài phần trăm F1 chưa đủ để khẳng định một cách tốt hơn hẳn. Chưa có kết quả LLM thật để so sánh zero-shot, few-shot, model lớn, model nhỏ và prompt phòng thủ.

## 3. Dự đoán và thực tế

| Dự đoán đã ghi | Kết quả | Điều rút ra |
| --- | --- | --- |
| `hard_eval`: accuracy của Embedding + LogReg khoảng 70% | Chỉ đạt 60% trên 50 case khó | Vector nghĩa vẫn có thể bỏ sót spam viết tự nhiên hoặc lẫn lời kể với nội dung dụ dỗ. Đây là chênh lệch accuracy lớn nhất trong các dự đoán số cho năm mô hình cục bộ ở stress test. |
| `emb_tricky`: Embedding + LogReg đúng nhiều hơn BoW | Cả hai cùng đúng 5/6 tin và cùng bỏ sót tin giục xác nhận thông tin để giữ phần thưởng | Mượn đặc trưng ngôn ngữ tốt hơn chưa bảo đảm sửa được mọi lỗi phân loại. |
| `tradeoff`: BoW + LogReg và một cách dùng embedding nằm trên đường Pareto | Đường Pareto còn có BoW + MLP, ngoài BoW + LogReg và MiniLM + MLP head | MLP trên BoW cũng tạo ra lựa chọn đáng cân nhắc vì tăng chất lượng trong khi độ trễ vẫn thấp. Phần dự đoán về API chưa kiểm chứng được. |

## 4. Phân tích 50 case khó

| Phương án | F1 SUB thường | F1 50 case khó | Accuracy 50 case khó |
| --- | ---: | ---: | ---: |
| BoW + LogReg | 0,824 | 0,378 | 54% |
| BoW + MLP | 0,947 | 0,410 | 54% |
| MiniLM + kNN | 0,974 | 0,714 | 68% |
| MiniLM + LogReg | 0,947 | 0,583 | 60% |
| MiniLM + MLP head | 0,974 | 0,612 | 62% |

BoW + MLP tụt nhiều nhất so với SUB: giảm khoảng 0,537 F1. Nhóm né từ khóa là một trong những nhóm mô hình này có tỉ lệ đúng thấp nhất. BoW + LogReg gặp khó ở nhóm thao túng, còn cả ba cách dùng embedding đều có một nhóm khó nhất là spam giọng tự nhiên. Khi có nhiều nhóm đồng mức thấp nhất, ở đây chỉ nêu một nhóm.

Hai ví dụ mà cả năm mô hình đều đoán sai:

- **Case 12:** tin mở đầu lịch sự, nói rằng được bạn bè cho số và đang muốn làm quen, nhưng thực chất dụ vào trang hẹn hò. Các dấu hiệu giống hội thoại cá nhân khiến mô hình nghiêng về ham.
- **Case 41:** tin OTP hợp lệ có lời nhắc bảo mật và hướng dẫn liên hệ ngân hàng lại bị coi là spam. Cách viết tự động, mã số và lời nhắc liên hệ ngân hàng có thể giống những mẫu lừa đảo đã học.

Điểm test thường cao chưa chứng minh mô hình hiểu được mọi tình huống. Các mẫu gần giống nhau vẫn có thể nằm ở cả train và test dù đã bỏ trùng tuyệt đối. Bộ khó có ích để tìm điểm yếu, nhưng vì được cố ý viết để gây nhầm lẫn nên không thể lấy trực tiếp tỉ lệ lỗi của nó làm tỉ lệ lỗi ngoài đời.

Trong phần bổ sung, cả năm case số 100–104 đều đánh lừa được MiniLM + MLP head. Chúng là lời kể hoặc trích dẫn vô hại nhưng chứa nhiều dấu hiệu nhận thưởng và dịch vụ tính phí. Kết quả của 55 tin được báo riêng để không làm thay đổi bảng 50 case gốc. Các case bổ sung có AI hỗ trợ xây dựng.

Chưa có phép đo LLM thật nên chưa thể kết luận prompt phòng thủ có cải thiện nhóm prompt injection hay gây chặn nhầm ở nhóm khác hay không.

## 5. Thử thách cá nhân

SEED 7353 ứng với đề số 4: thay k của kNN bằng 1, 5 và 25 rồi so sánh F1.

| k | Precision | Recall | F1 test |
| ---: | ---: | ---: | ---: |
| 1 | 0,891 | 0,947 | 0,918 |
| 5 | 0,957 | 0,937 | 0,947 |
| 25 | 0,874 | 0,947 | 0,909 |

Trong ba giá trị được thử, k = 5 cho F1 cao nhất. k = 1 dễ phụ thuộc vào một hàng xóm thiếu đại diện. Khi k quá lớn, nhiều hàng xóm ít liên quan cũng được tham gia bỏ phiếu, có thể làm mất ranh giới giữa hai nhãn. Nếu chọn k cho sản phẩm, cần chọn trên validation rồi đánh giá cuối cùng trên test.

Phần thử thách hiện có AI hỗ trợ, chưa phải bằng chứng làm độc lập trong 15 phút.

## 6. Phương án cho sản phẩm thật

Với điều kiện p95 dưới 50 ms/tin, BoW + LogReg, BoW + MLP, MiniLM + LogReg và MiniLM + MLP head đạt yêu cầu trong lần đo tuần tự. MiniLM + kNN có p95 khoảng 60,204 ms nên vượt ngưỡng.

BoW + MLP là phương án ban đầu đáng cân nhắc khi ưu tiên tốc độ: F1 test khoảng 0,946 và p95 khoảng 0,311 ms. MiniLM + MLP head tăng F1 test lên khoảng 0,973 với p95 khoảng 32,971 ms, nhưng thông lượng cả lô chỉ khoảng 63 tin/giây, thấp hơn mức trung bình 116 tin/giây của 10 triệu SMS/ngày. Vì vậy cần thêm tài nguyên, số worker và thử tải trước khi xem nó là phương án đủ sức phục vụ toàn bộ lưu lượng. Kết quả trên bộ khó cũng cho thấy chất lượng vẫn cần cải thiện.

Một thiết kế kết hợp là cho mô hình BoW xử lý các tin chắc chắn, chuyển tin còn phân vân sang embedding. Chỉ một phần nhỏ các ca bất đồng cần đến LLM hoặc người kiểm tra. Nếu bắt buộc trả nhãn trong 50 ms thì LLM nên dùng cho hậu kiểm bất đồng bộ, tránh để một lần gọi API chậm làm vỡ yêu cầu độ trễ. Các ngưỡng chuyển tuyến cần được chọn trên validation theo mức chấp nhận chặn nhầm và bỏ sót.

Theo giá giả định của đề, mỗi SMS có 100 token input và 1 token output:

- Model nhỏ: 10 triệu tin/ngày × 30 ngày × (100 × 0,10 + 1 × 0,50) / 1.000.000 = **3.150 USD/tháng**.
- Model lớn có đơn giá gấp 10 lần trong đề: **31.500 USD/tháng**.

Đây là phép tính theo giả định, chưa phải chi phí đo từ API. Prompt dài hơn, few-shot và token suy luận có thể làm tổng tiền thay đổi.

## 7. AI log và điều đã học

| Phạm vi | Yêu cầu với AI | Hỗ trợ nhận được | Kiểm chứng và giới hạn |
| --- | --- | --- | --- |
| Sau Lab 1.B đến cuối notebook | Hoàn thiện bài còn trống; không thêm chú thích trong code; nhận xét bằng tiếng Việt tự nhiên | Bổ sung code, dự đoán, câu trả lời, case thử và thử thách kNN | Đã chạy năm mô hình cục bộ; 15/15 kiểm tra đạt. API LLM chỉ được kiểm tra bằng giả lập. |
| Chuẩn bị bài nộp | Đọc hướng dẫn, sắp xếp file và điền báo cáo từ kết quả có sẵn | Tạo bộ file trong thư mục nộp và tổng hợp số liệu | Báo cáo được đối chiếu với JSON và output notebook. Ảnh tính tay đã được bổ sung; chưa có kết quả LLM thật. |
| Hiệu đính nội dung | Sửa các lỗi nội dung được phát hiện khi rà soát | Sửa cách giải thích data leakage, nhận xét learning rate và lỗi lặp câu; thống nhất cách gọi SMS | Giữ nguyên ý dự đoán, thời gian ghi dự đoán, code mô hình và kết quả; đồng bộ notebook với JSON. |

Phần 2–3, dự đoán, case bổ sung, câu trả lời và thử thách có AI hỗ trợ; người học chưa xác nhận tự kiểm chứng toàn bộ hoặc làm lại độc lập. Cần xử lý các phần yêu cầu tự làm theo quy định của lớp trước khi nộp.

Kết quả cho thấy F1 cao trên test cùng nguồn chưa đủ để kết luận mô hình xử lý tốt các tình huống mới. Thêm lớp ẩn có thể có lợi, nhưng chọn phương án cho sản phẩm còn phải xem độ trễ, thông lượng và kiểu lỗi. Embedding giúp biểu diễn ý nghĩa tốt hơn BoW, song vẫn có thể nhầm lời trích dẫn hoặc spam viết giống hội thoại cá nhân.
