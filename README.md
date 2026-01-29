"# ML_Fraud_detection" 

I. Giới thiệu

      Project ML_fraud_detection là một bài toán Machine Learning nhằm phát triển mô hình phân loại giao dịch – phát hiện gian lận (fraud detection) trên dữ liệu giao dịch rentalz. Mục tiêu của dự án là xây dựng pipeline từ khám phá dữ liệu (EDA), xử lý dữ liệu, huấn luyện mô hình đến đánh giá hiệu năng dự đoán để hỗ trợ hệ thống nhận diện giao dịch bất thường. Project được xây dựng trên google colab dưới dạng Notebook để dễ dàng theo dõi toàn bộ quá trình phân tích, tiền xử lý dữ liệu, lựa chọn mô hình và đánh giá kết quả.
        
+Mục tiêu

    -Phân tích dữ liệu giao dịch (transaction data) từ rentalz để hiểu cấu trúc và hành vi của dữ liệu.
    
    -Xử lý các vấn đề dữ liệu phổ biến: thiếu giá trị, biến phân loại, chuẩn hoá/chuẩn hoá biến số.
    
    -Huấn luyện và đánh giá các mô hình Machine Learning nhằm phát hiện giao dịch gian lận (Fraud Detection).
    
    -Trực quan hóa kết quả, giải thích hiệu năng và khả năng áp dụng mô hình vào môi trường thực tế.

II. Nội dung chính
   +Tiền xử lý dữ liệu:

      -Làm sạch dữ liệu: điền thiếu, loại bỏ biến không cần thiết.
      
      -Mã hóa biến phân loại (categorical encoding).
      
      -Chuẩn hóa/tiêu chuẩn hóa các biến số.
    
    +Ở bản update:
      
      - Thêm các cải thuật tính đểm dựa trên đồ thị để biểu diễn tương quan mối quan hệ giữa các giao dịch với nhau, nhằm tạo ra những feature mới để hỗ trợ mô hình học thêm cách phát hiện những nhóm giao dịch gian lận có tổ chức. 
      
      - Đánh giá được mức độ liên quan của các tài khoản có trong mạng lưới giao dịch gian lận, xác định được tài khoản nào là tài khoản chính, tài khoản trung gian.
      - Phát hiện được thêm các giao dịch gian lận mới nhờ đánh giá qua các điểm liên quan giữa tài khoản mới và tải khoản cũ. 
      - Sử dụng Sparksql thay vì pandas dể tối ưu tốc độ tính toán các feature. 
      - Customize Page rank init theo trọng số và không share đều cho các user để các giao dịch có gian lận được tính score cao hơn từ đó giúp training đạt hiệu quả cao hơn. 
    

III. Xây dựng mô hình và so sánh hiệu suất của các mô hình. 

        -So sánh nhiều thuật toán phân loại (ví dụ: Logistic Regression, Random Forest, XGBoost).
        
        -Sử dụng cross‑validation để ước tính hiệu năng dự đoán thực tế.
        
        -Đánh giá mô hình:
        
        -Đánh giá các chỉ số Accuracy, Precision, Recall, F1‑Score, ROC‑AUC của các mô hình. 
        
        -Trực quan hóa kết quả ROC Curve, ma trận nhầm lẫn để đánh giá và so sánh các mô hình với nhau.
