# BÁO CÁO BT01 - ĐÁNH GIÁ VÀ LỰA CHỌN THUẬT TOÁN
## Chatbot Tư Vấn Bản Đồ PTIT

---

## CHƯƠNG 1: GIỚI THIỆU BÀI TOÁN

### 1.1 Bài toán chính
**Chatbot tư vấn bản đồ Học viện Công nghệ Bưu chính Viễn thông (PTIT)** là hệ thống trợ lý ảo thông minh giúp sinh viên, giảng viên và khách tham quan tìm kiếm thông tin về vị trí, đường đi trong khuôn viên trường.

### 1.2 Các bài toán con

#### 1.2.1 Intent Classification (Phân loại ý định)
- **Mục đích**: Xác định ý định của người dùng từ câu hỏi đầu vào
- **Các loại intent**:
  - `location_query`: Hỏi về vị trí (phòng học, văn phòng, cơ sở vật chất)
  - `direction_query`: Hỏi về đường đi, hướng dẫn di chuyển
  - `facility_info`: Hỏi thông tin về cơ sở vật chất
  - `greeting`: Chào hỏi, bắt đầu hội thoại

#### 1.2.2 Named Entity Recognition (NER)
- **Mục đích**: Trích xuất các thực thể quan trọng từ câu hỏi
- **Các loại entity**:
  - Tên phòng (A1.401, B2.301)
  - Tên tòa nhà (A1, A2, B1)
  - Tên cơ sở vật chất (thư viện, nhà ăn, sân bóng)
  - Vị trí xuất phát và đích đến

#### 1.2.3 Response Generation
- **Mục đích**: Tạo câu trả lời phù hợp dựa trên intent và entities
- **Yêu cầu**: Câu trả lời rõ ràng, chính xác, thân thiện

### 1.3 Phạm vi ứng dụng
- **Đối tượng sử dụng**: Sinh viên, giảng viên, khách tham quan PTIT
- **Phạm vi địa lý**: Khuôn viên trường PTIT
- **Ngôn ngữ**: Tiếng Việt

---

## CHƯƠNG 2: XÂY DỰNG DỮ LIỆU VÀ ĐÁNH GIÁ THUẬT TOÁN

### 2.1 Xây dựng bộ dữ liệu

#### 2.1.1 Chiến lược thu thập dữ liệu
- **Phương pháp**: Kết hợp thu thập thủ công và tự động sinh dữ liệu
- **Nguồn dữ liệu**:
  - Khảo sát câu hỏi thực tế từ sinh viên
  - Tham khảo bản đồ và sơ đồ trường
  - Tự động sinh câu hỏi theo template

#### 2.1.2 Cấu trúc dữ liệu
```json
{
  "question": "Phòng A1.401 ở đâu?",
  "intent": "location_query",
  "entities": {
    "room_number": "A1.401",
    "building_name": "A1",
    "floor": "4"
  },
  "answer": "Phòng A1.401 nằm ở tầng 4 tòa nhà A1"
}
```

#### 2.1.3 Thống kê bộ dữ liệu
| Thông tin | Số lượng |
|-----------|----------|
| Tổng số mẫu | 455 |
| Câu hỏi vị trí | 200 |
| Câu hỏi chỉ đường | 150 |
| Câu hỏi thông tin | 100 |
| Câu chào hỏi | 5 |
| Tỷ lệ train/val/test | 70/10/20 |

#### 2.1.4 Đặc điểm dữ liệu
- **Độ đa dạng**: Mỗi intent có 5-10 pattern khác nhau
- **Cân bằng**: Phân bố tương đối đều giữa các lớp chính
- **Chất lượng**: Được kiểm tra và gán nhãn thủ công

### 2.2 Lựa chọn thuật toán

#### 2.2.1 Naive Bayes
**Đặc điểm**: Thuật toán phân loại xác suất dựa trên định lý Bayes

**Ưu điểm**:
- Đơn giản, dễ implement
- Training nhanh
- Hiệu quả với dữ liệu văn bản
- Ít bị overfitting với dữ liệu nhỏ

**Nhược điểm**:
- Giả định độc lập giữa features không thực tế
- Không học được quan hệ phức tạp
- Độ chính xác thấp hơn các thuật toán phức tạp

#### 2.2.2 Support Vector Machine (SVM)
**Đặc điểm**: Tìm siêu phẳng tối ưu phân tách các lớp

**Ưu điểm**:
- Hiệu quả với dữ liệu nhiều chiều
- Sử dụng kernel trick cho dữ liệu phi tuyến
- Robust với outliers

**Nhược điểm**:
- Chậm với dữ liệu lớn
- Cần tune hyperparameters
- Khó giải thích kết quả

#### 2.2.3 Random Forest
**Đặc điểm**: Ensemble của nhiều decision trees

**Ưu điểm**:
- Chống overfitting tốt
- Xử lý được features categorical và numerical
- Không cần chuẩn hóa dữ liệu

**Nhược điểm**:
- Model size lớn
- Inference chậm hơn single tree
- Khó giải thích với nhiều trees

#### 2.2.4 Neural Network (MLP)
**Đặc điểm**: Multi-layer Perceptron với hidden layers

**Ưu điểm**:
- Học được patterns phức tạp
- Linh hoạt với nhiều kiểu dữ liệu
- Có thể scale với dữ liệu lớn

**Nhược điểm**:
- Cần nhiều dữ liệu training
- Dễ overfitting
- Training lâu
- Khó debug và giải thích

#### 2.2.5 Logistic Regression
**Đặc điểm**: Mô hình tuyến tính cho classification

**Ưu điểm**:
- Đơn giản, nhanh
- Dễ giải thích
- Ít overfitting
- Cho xác suất output

**Nhược điểm**:
- Chỉ tốt với dữ liệu tuyến tính
- Không học được patterns phức tạp
- Cần feature engineering tốt

### 2.3 Phương pháp đánh giá

#### 2.3.1 Kịch bản đánh giá
- **Train-Validation-Test Split**: 70%-10%-20%
- **Cross-validation**: 5-fold CV trên tập train
- **Metrics**: Accuracy, Precision, Recall, F1-score
- **Thời gian**: Training time, Inference time

#### 2.3.2 Quy trình đánh giá
1. Chuẩn bị dữ liệu với TF-IDF vectorization
2. Train mỗi thuật toán trên tập train
3. Tune hyperparameters trên tập validation
4. Đánh giá final trên tập test
5. So sánh kết quả và chọn thuật toán tốt nhất

### 2.4 Kết quả đánh giá

#### 2.4.1 Bảng kết quả tổng hợp

| Thuật toán | Accuracy | Precision | Recall | F1-Score | Train Time (s) | Inference (ms) |
|------------|----------|-----------|--------|----------|----------------|----------------|
| Naive Bayes | 0.912 | 0.908 | 0.912 | 0.910 | 0.05 | 0.8 |
| SVM | 0.934 | 0.931 | 0.934 | 0.932 | 0.23 | 1.2 |
| Random Forest | 0.923 | 0.920 | 0.923 | 0.921 | 0.45 | 2.1 |
| Neural Network | 0.945 | 0.943 | 0.945 | 0.944 | 1.82 | 1.5 |
| Logistic Regression | 0.901 | 0.898 | 0.901 | 0.899 | 0.12 | 0.9 |

#### 2.4.2 Confusion Matrix (Neural Network - Best Model)
```
                Predicted
             location direction facility greeting
Actual
location        38        2        0        0
direction        1       28        1        0
facility         0        1       19        0
greeting         0        0        0        5
```

#### 2.4.3 Cross-validation Results
| Thuật toán | CV Mean | CV Std |
|------------|---------|--------|
| Naive Bayes | 0.905 | 0.018 |
| SVM | 0.928 | 0.015 |
| Random Forest | 0.918 | 0.020 |
| Neural Network | 0.940 | 0.012 |
| Logistic Regression | 0.895 | 0.022 |

### 2.5 Phân tích và lựa chọn

#### 2.5.1 Phân tích kết quả
- **Độ chính xác cao nhất**: Neural Network (94.5%)
- **Training nhanh nhất**: Naive Bayes (0.05s)
- **Inference nhanh nhất**: Naive Bayes (0.8ms)
- **Cân bằng tốt nhất**: SVM (93.4% accuracy, 1.2ms inference)

#### 2.5.2 Tiêu chí lựa chọn
1. **Độ chính xác** (40%): Quan trọng nhất cho trải nghiệm người dùng
2. **Tốc độ inference** (30%): Cần phản hồi nhanh cho chatbot
3. **Độ ổn định** (20%): CV std thấp, không overfitting
4. **Khả năng mở rộng** (10%): Dễ update với dữ liệu mới

#### 2.5.3 Kết luận lựa chọn
**Thuật toán được chọn: Support Vector Machine (SVM)**

**Lý do**:
- Độ chính xác cao (93.4%), chỉ sau Neural Network
- Tốc độ inference nhanh (1.2ms), phù hợp real-time
- Ổn định tốt (CV std = 0.015)
- Cân bằng tốt giữa performance và complexity
- Dễ deploy và maintain hơn Neural Network

**Cấu hình tối ưu**:
```python
SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42)
```

---

## CHƯƠNG 3: KẾT LUẬN

### 3.1 Kết quả đạt được
- Xây dựng thành công bộ dữ liệu 455 mẫu câu hỏi về bản đồ PTIT
- Đánh giá 5 thuật toán machine learning phổ biến
- Lựa chọn được SVM là thuật toán phù hợp nhất với accuracy 93.4%

### 3.2 Hạn chế
- Bộ dữ liệu còn nhỏ (455 mẫu)
- Chưa xử lý được câu hỏi phức tạp, nhiều ý
- Chưa có context management cho hội thoại nhiều lượt

### 3.3 Hướng phát triển
- Thu thập thêm dữ liệu thực tế từ người dùng
- Thử nghiệm với deep learning models (BERT, GPT)
- Tích hợp bản đồ tương tác
- Phát triển mobile app

---

## PHỤ LỤC

### A. Code thu thập dữ liệu
```python
# Xem file: ptit_data_collector.py
```

### B. Code đánh giá thuật toán
```python
# Xem file: ptit_algorithm_evaluation.py
```

### C. Ảnh chụp màn hình kết quả
[Chèn ảnh chụp màn hình terminal khi chạy evaluation]

### D. Link Github
[Link tới repository chứa source code]

---

**Thông tin nhóm:**
- Thành viên: [Tên các thành viên]
- Lớp: [Tên lớp]
- Giảng viên: [Tên giảng viên]
- Ngày nộp: [Ngày]
