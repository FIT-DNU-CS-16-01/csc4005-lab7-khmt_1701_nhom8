# Lab 7 Quantization Comparison

## Kết luận nhanh
- **Baseline**: giữ accuracy tốt nhất nhưng file model lớn.
- **QInt8 (gợi ý ban đầu)**: giảm size rất mạnh, accuracy giảm nhẹ, latency tăng.
- **QUInt8 (bản điều chỉnh)**: cũng giảm size mạnh, nhưng accuracy/latency hơi kém hơn QInt8.

## Số liệu
| Model | Accuracy | Macro-F1 | Mean latency (ms) | Throughput (img/s) | Size (MB) |
|---|---:|---:|---:|---:|---:|
| Baseline | 0.7326 | 0.7042 | 407.25 | 2.4555 | 327.42 |
| QInt8 MatMul | 0.6958 | 0.6811 | 432.43 | 2.3125 | 84.43 |
| QUInt8 MatMul | 0.6984 | 0.6845 | 436.59 | 2.2905 | 84.43 |

## So sánh với baseline
### QInt8 MatMul
- Size giảm khoảng **74.21%**.
- Accuracy giảm **0.0368** điểm.
- Macro-F1 giảm **0.0231** điểm.
- Latency tăng khoảng **6.18%**.
- Throughput giảm khoảng **5.82%**.

### QUInt8 MatMul
- Size giảm khoảng **74.21%**.
- Accuracy giảm **0.0342** điểm.
- Macro-F1 giảm **0.0197** điểm.
- Latency tăng khoảng **7.20%**.
- Throughput giảm khoảng **6.72%**.

## Nhận xét
- Nếu mục tiêu chính là **giảm dung lượng lưu trữ**, quantization đáng làm vì size giảm rất mạnh.
- Nếu mục tiêu là **tăng tốc CPU inference**, trên export này quantization **không có lợi** vì latency tăng.
- Trong hai bản, **QInt8** nhỉnh hơn QUInt8 một chút về accuracy và latency.
- Vì vậy, nếu vẫn chọn quantization cho lab này, mình khuyên chọn **QInt8 MatMul** làm bản nộp chính, còn QUInt8 dùng làm bản so sánh.

## Lưu ý kỹ thuật
- Baseline ONNX dùng external data nên size phải tính cả file `models/vit_smartcampus.onnx.data`.
- Model quantized đầy đủ nên benchmark/evaluate ở `batch_size=1` để đúng với export hiện tại.
