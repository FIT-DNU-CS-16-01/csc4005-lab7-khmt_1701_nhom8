# CSC4005 Lab 7 Report – Compression: KD + Quantization Trade-offs

## 1. Thông tin

- Họ tên: Đinh Trọng Quỳnh
- Mã sinh viên: 17710*****
- Lớp: KHMT 1701
- Link GitHub repo:
- Kỹ thuật chọn: Quantization
- Link W&B: https://wandb.ai/dinhtrongquynh99-dainam-vietnam/csc4005-lab7-compression/runs/muoicezq
- Link model nếu không commit trực tiếp: `models/`

## 2. Mô tả baseline model

| Nội dung | Giá trị |
|---|---|
| Bài toán | Smart Campus Scene Classification |
| Dataset | MIT Indoor Scenes 67 subset 5 lớp |
| Số lớp | 5 |
| Baseline model | Vision Transformer |
| Baseline format | ONNX |
| Baseline checkpoint/ONNX | `models/vit_smartcampus.onnx` |
| Baseline model size | 327.42 MB |

## 3. Kỹ thuật nén đã chọn

### Quantization

| Thông tin | Giá trị |
|---|---|
| Loại quantization | Dynamic |
| Input model | `models/vit_smartcampus.onnx` |
| Output model (gợi ý) | `models/vit_smartcampus_dynamic_int8.onnx` |
| Output model (điều chỉnh) | `models/vit_smartcampus_dynamic_uint8.onnx` |
| Dạng dữ liệu sau nén | INT8 / UINT8 |
| Công cụ | `onnxruntime.quantization` |

Mô tả ngắn:

```text
Mình triển khai dynamic quantization cho ONNX baseline. Bản gợi ý ban đầu dùng QInt8, và bản điều chỉnh dùng QUInt8. Để tránh lỗi shape inference của ORT trên model này, mình chỉ quantize nhóm MatMul và giữ default tensor type là FLOAT. Model baseline dùng external data nên size phải tính cả file .onnx.data.
```

## 4. Kết quả đánh giá

| Model | Accuracy | Macro-F1 | Model size (MB) |
|---|---:|---:|---:|
| Baseline | 0.7326 | 0.7042 | 327.42 |
| QInt8 MatMul | 0.6958 | 0.6811 | 84.43 |
| QUInt8 MatMul | 0.6984 | 0.6845 | 84.43 |

Nhận xét:

- Accuracy giảm khoảng **0.0368** điểm với QInt8 và **0.0342** điểm với QUInt8.
- Macro-F1 giảm khoảng **0.0231** điểm với QInt8 và **0.0197** điểm với QUInt8.
- Mức giảm là chấp nhận được nếu ưu tiên dung lượng lưu trữ, nhưng latency không giảm.

## 5. Kết quả benchmark

| Model | Batch size | Mean latency (ms) | P95 latency (ms) | Throughput (img/s) | Size (MB) |
|---|---:|---:|---:|---:|---:|
| Baseline | 1 | 407.25 | 629.07 | 2.4555 | 327.42 |
| QInt8 MatMul | 1 | 432.43 | 1174.83 | 2.3125 | 84.43 |
| QUInt8 MatMul | 1 | 436.59 | 747.25 | 2.2905 | 84.43 |

Ghi chú: model ONNX này chạy ổn định ở batch size 1, nên benchmark/report tập trung vào batch size 1 để đảm bảo công bằng.

## 6. Bảng trade-off

| Model | Accuracy | Macro-F1 | Mean latency @bs=1 | Throughput @bs=1 | Size | Nhận xét |
|---|---:|---:|---:|---:|---:|---|
| Baseline | 0.7326 | 0.7042 | 407.25 | 2.4555 | 327.42 | Mốc so sánh |
| QInt8 MatMul | 0.6958 | 0.6811 | 432.43 | 2.3125 | 84.43 | Nhỏ hơn nhiều, nhưng chậm hơn |
| QUInt8 MatMul | 0.6984 | 0.6845 | 436.59 | 2.2905 | 84.43 | Tương tự QInt8, hơi kém hơn |

## 7. Phân tích

1. Mô hình sau nén nhỏ hơn khoảng **74.21%**.
2. Latency **tăng** thay vì giảm: QInt8 +6.18%, QUInt8 +7.20%.
3. Throughput **giảm**: QInt8 -5.82%, QUInt8 -6.72%.
4. Accuracy/F1 giảm nhẹ, khoảng 0.02–0.04 điểm.
5. Nếu triển khai trên CPU hoặc edge device, mình chỉ chọn model sau nén khi mục tiêu là **giảm dung lượng lưu trữ**; còn nếu ưu tiên tốc độ thì baseline tốt hơn.
6. Trong hai bản, **QInt8 MatMul** nhỉnh hơn QUInt8 một chút về accuracy và latency.

## 8. Khi nào chọn KD, khi nào chọn Quantization?

- **Quantization** phù hợp khi đã có model tốt, muốn nén nhanh, giảm size nhanh, và chấp nhận kiểm tra lại accuracy sau nén.
- **KD** phù hợp khi cần student model nhỏ hơn rõ rệt, có thời gian train lại, và muốn tối ưu cho triển khai lâu dài hơn.
- Nếu được làm lại cho hệ thống Smart Campus, mình vẫn chọn **Quantization** vì phù hợp với mục tiêu nén nhanh và dễ triển khai.

## 9. Kết luận

Mình đã dùng dynamic quantization cho baseline ONNX của bài toán Smart Campus và đẩy toàn bộ kết quả đánh giá lên W&B. Bản nén giảm size khoảng 74.21%, nhưng latency không cải thiện mà còn tăng nhẹ. So sánh hai biến thể, QInt8 MatMul tốt hơn QUInt8 MatMul một chút về accuracy và latency, nên nếu phải chọn một bản để nộp thì mình chọn QInt8. Bài lab cho thấy quantization rất hiệu quả về dung lượng, nhưng không phải lúc nào cũng cải thiện tốc độ suy luận. Bài học chính là phải nhìn đủ ba yếu tố: accuracy, latency, và size.
