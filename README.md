# ML_Fraud_detection
# Feature Engineering Phân Tán & Modeling Theo Cost
## 1. Tổng quan
Dự án này xây dựng **pipeline phát hiện gian lận (fraud detection) end-to-end** trên dữ liệu giao dịch lớn, kết hợp:
* **Apache Spark** cho tính toán phân tán (feature engineering, temporal features, graph)
* **Pandas / NumPy + LightGBM / XGBoost** cho huấn luyện model và tối ưu ngưỡng theo chi phí
Thiết kế bám sát **kiến trúc thực tế trong ngân hàng / fintech**:
> **Spark xử lý nặng & dữ liệu lớn → bảng feature gọn → model tabular mạnh → threshold theo business cost**
Mục tiêu chính:
* **Recall cao**: hạn chế bỏ sót fraud
* **Giảm FP**: tránh làm phiền khách hàng không cần thiết
## 2. Dữ liệu
* Dữ liệu giao dịch (transaction-level)
* Tỷ lệ gian lận cực thấp (< 1%) → bài toán **class imbalance nặng**
Các thực thể chính:
* `nameOrig`: tài khoản nguồn
* `nameDest`: tài khoản đích
* `amount`: số tiền giao dịch
* `step`: mốc thời gian rời rạc
* `isFraud`: nhãn gian lận

Chỉ giữ các loại giao dịch có ý nghĩa gian lận (ví dụ: `TRANSFER`, `CASH_OUT`).

## 3. Kiến trúc hệ thống

Raw CSV (hàng triệu giao dịch)
        ↓
Apache Spark (Distributed)
  - Lọc dữ liệu
  - Aggregation theo account
  - Temporal / window features
  - Graph-based PageRank
        ↓
Bảng feature (đã cô đặc)
        ↓
Pandas / NumPy
        ↓
Model (LightGBM / XGBoost)
        ↓
Chọn threshold theo cost + recall
        ↓
Đánh giá & deploy

Spark chỉ dùng **đúng chỗ cần phân tán**, model training giữ ở single-node để tối ưu linh hoạt.


## 4. Feature Engineering (Spark)

### 4.1 Feature giao dịch cơ bản

* Chênh lệch số dư:

  * `orig_diff = oldbalanceOrg - newbalanceOrig`
  * `dest_diff = newbalanceDest - oldbalanceDest`
* Chuẩn hóa theo amount
* Tổng số giao dịch, tổng amount theo account

### 4.2 Feature temporal (hành vi theo thời gian)

Sử dụng sliding window (N giao dịch gần nhất):

* `tx_cnt_10`
* `tx_amt_sum_10`
* `tx_amt_mean_10`
* `tx_amt_std_10`
* `time_since_last_tx`

→ Bắt được **hành vi bất thường / thay đổi đột ngột**.

### 4.3 Feature đồ thị – Fraud PageRank (custom)

Triển khai **PageRank tùy biến cho fraud network** bằng Spark (không dùng GraphFrames):

* Node: tài khoản
* Edge: giao dịch
* Trọng số cạnh:

  * Chuẩn hóa theo tổng amount đi ra của node
* Khởi tạo không đều (personalized):

  * Dựa trên số giao dịch + tổng amount

Sinh ra:

* `orig_pagerank`
* `dest_pagerank`

Ý nghĩa: **lan truyền rủi ro trong mạng lưới giao dịch**, không phải PageRank thuần lý thuyết.

---

## 5. Feature set cuối cùng (không leakage)

```
FEATURES = [
    "step", "amount",
    "orig_diff", "dest_diff",
    "amount_diff_orig", "amount_diff_dest",
    "orig_tx_count", "dest_tx_count",
    "orig_unique_dest", "dest_unique_orig",
    "orig_sum_amount", "dest_sum_amount",
    "orig_pagerank", "dest_pagerank",
    "tx_cnt_10",
    "tx_amt_sum_10",
    "tx_amt_mean_10",
    "tx_amt_std_10",
    "time_since_last_tx",
]
```

Tất cả feature đều được tính **từ quá khứ**, đảm bảo không leak thông tin tương lai.

---

## 6. Huấn luyện model

Model sử dụng:

* LightGBM
* XGBoost
* RandomForest
* Logistic Regression (baseline)

Không dùng Spark MLlib vì:

* Khó tối ưu PR-AUC
* Không linh hoạt với cost-based threshold
* Hiệu quả kém hơn GBDT cho fraud

## 7. Metric đánh giá

Do dữ liệu mất cân bằng mạnh → **KHÔNG dùng accuracy**.

Metric chính:

* **PR-AUC**: chất lượng xếp hạng
* **Recall**: khả năng bắt fraud
* **Precision**: ảnh hưởng tới khách hàng

Metric theo nghiệp vụ:

* **Recall@TopK%**: bắt được bao nhiêu fraud khi chỉ soi K% giao dịch rủi ro nhất
* **Lift@K**: tốt hơn random bao nhiêu lần

## 8. Chọn threshold theo cost (Cost-based)

Không dùng threshold = 0.5.

Áp dụng:

* Ràng buộc: Recall ≥ 85%
* Trong vùng đó, chọn threshold **minimize expected cost**

```
Expected Cost = COST_FN × FN + COST_FP × FP
```

Đảm bảo:

* Không bỏ sót fraud quan trọng
* Giảm false positive ở mức tối đa có thể


## 9. Kết quả điển hình

* Recall: ~0.88 – 0.91
* Precision: ~0.15 – 0.18 (bình thường với fraud)
* PR-AUC: ~0.80+
* Lift@Top0.1% cao

Precision thấp là **bản chất bài toán**, không phải lỗi model.

## 10. Triển khai thực tế

Pipeline phù hợp cho:

* Offline training
* Online scoring

Spark dùng cho batch feature
Model deploy dạng service để scoring real-time.

## 11. Tổng kết

* Dùng distributed computing đúng chỗ
* Modeling linh hoạt, sát business
* Đánh giá theo góc nhìn fraud thật
* Kiến trúc tương đương hệ thống production

## 12. Hướng phát triển tiếp

* Community / ring detection
* Graph temporal features
* Streaming fraud detection
* GNN khi hạ tầng cho phép

## 13. Lưu ý

Dự án mang tính học thuật – nghiên cứu, được xây dựng trên dataset có sẵn, chưa thay thế hệ thống fraud production nếu chưa có monitoring, rule engine và kiểm soát nghiệp vụ.
á và so sánh các mô hình với nhau.
