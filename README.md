---
title: Safety Helmet Detection
emoji: 👷
colorFrom: yellow
colorTo: gray
sdk: docker
pinned: false
---

# Safety Helmet Detection

Safety Helmet Detection là hệ thống AI hỗ trợ giám sát việc đội mũ bảo hộ tại công trường. Dự án sử dụng mô hình phát hiện đối tượng YOLOv8s-AP để nhận diện ba trạng thái vùng đầu của công nhân: `helmet`, `head` và `non-helmet`. Hệ thống có backend FastAPI, giao diện React và có thể chạy inference trên ảnh, video tải lên hoặc luồng camera.

Demo: [Hugging Face Spaces - safety-helmet-detection](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)

## Mục Tiêu Bài Toán

Bài toán cần giải quyết là tự động phát hiện các trường hợp không tuân thủ quy định đội mũ bảo hộ trong môi trường công trường xây dựng. Thay vì chỉ phân biệt có đội mũ hoặc không đội mũ, mô hình được huấn luyện với ba lớp:

| ID | Class | Ý nghĩa |
|---:|---|---|
| 0 | `helmet` | Đội mũ bảo hộ đạt chuẩn |
| 1 | `head` | Không đội mũ, vùng đầu trần |
| 2 | `non-helmet` | Có đội mũ nhưng không phải mũ bảo hộ đạt chuẩn, ví dụ mũ vải hoặc mũ lưỡi trai |

Kết quả nhận diện được dùng để hỗ trợ cảnh báo vi phạm, lưu lịch sử sự kiện và tạo dữ liệu bổ sung cho các lần cải thiện mô hình sau này.

## Kiến Trúc Dự Án

```text
Safety-helmet-detection/
├── src/
│   ├── api/          # FastAPI routes, schemas, rate limit
│   ├── config/       # Settings, logging, environment variables
│   ├── core/         # Predictor, tracking, streaming, history, reporting
│   └── utils/        # Image and detection utilities
├── frontend/         # React + Vite UI
├── models/           # Model weights: .pt, .onnx
├── notebooks/        # Training notebook and graduation report
├── demo/             # Demo image/video assets
├── data/             # Runtime data: history, videos, reports, update pool
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

Backend dùng singleton `Predictor` để chỉ load model một lần khi server khởi động. Model mặc định là `models/yolov8s_ap.onnx`, có thể đổi qua biến môi trường `MODEL_PATH`. Nếu môi trường có CUDA, hệ thống ưu tiên GPU; nếu không, tự fallback về CPU.

## Chuẩn Bị Dataset

Theo báo cáo khóa luận, dataset được tổng hợp từ nhiều nguồn:

| Nguồn | Số ảnh | Ghi chú xử lý |
|---|---:|---|
| Kaggle Hard Hat Detection | 5.000 | Giữ `helmet`, `head`; bỏ nhãn `person` |
| Roboflow Cap dataset | 12.147 | Mapping một số lớp mũ thường về `non-helmet` |
| Roboflow Hat dataset | 950 | Mapping class gốc về `non-helmet` |
| Dữ liệu tự thu thập | 165 | Gán nhãn thủ công với 3 lớp `helmet`, `head`, `non-helmet` |

Dataset cuối cùng:

| Split | Số ảnh | Tỷ lệ |
|---|---:|---:|
| Train | 15.369 | 80% |
| Validation | 1.921 | 10% |
| Test | 1.922 | 10% |
| Tổng | 19.212 | 100% |

Phân bố bounding box:

| ID | Class | Số box | Tỷ lệ |
|---:|---|---:|---:|
| 0 | `helmet` | 19.089 | 43,88% |
| 1 | `head` | 5.875 | 13,50% |
| 2 | `non-helmet` | 18.543 | 42,62% |
|  | Tổng | 43.507 | 100% |

Dataset nên được chuẩn hóa về định dạng YOLO:

```text
dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
├── labels/
│   ├── train/
│   ├── val/
│   └── test/
└── data.yaml
```

Ví dụ `data.yaml`:

```yaml
path: /path/to/dataset
train: images/train
val: images/val
test: images/test

names:
  0: helmet
  1: head
  2: non-helmet
```

Các bước chuẩn bị chính:

1. Thu thập ảnh từ các nguồn công khai và dữ liệu tự chụp.
2. Lọc các nhãn không phục vụ bài toán, ví dụ `person`.
3. Mapping nhãn về đúng ba lớp chuẩn.
4. Kiểm tra lại bounding box bị sai, trùng hoặc lệch khỏi đối tượng.
5. Chia train/validation/test theo tỷ lệ 8:1:1.
6. Chuẩn hóa cấu trúc thư mục và file `data.yaml` theo định dạng YOLO.

## Train Và Evaluate

Mô hình chính trong báo cáo là YOLOv8s-AP, được phát triển từ YOLOv8s và bổ sung:

| Thành phần | Mục đích |
|---|---|
| ASFF ở phần neck | Cải thiện trộn đặc trưng đa tỉ lệ, hữu ích cho mũ/vùng đầu nhỏ hoặc ở xa camera |
| Pseudo-DCNv2 ở detection head | Giúp mô hình thích nghi tốt hơn với hình dạng bị biến dạng, che khuất hoặc góc nhìn khó |

Môi trường huấn luyện trong báo cáo:

| Thành phần | Cấu hình |
|---|---|
| Nền tảng | Kaggle Notebook |
| GPU | NVIDIA Tesla P100 16 GB |
| Python | 3.10 |
| Image size | 640x640 |
| Epochs | 50 |
| Batch size | 16 |
| Workers | 2 |

Notebook huấn luyện nằm tại:

```text
notebooks/Model_trainer.ipynb
```

Ví dụ lệnh train baseline YOLOv8s với Ultralytics:

```bash
yolo detect train model=yolov8s.pt data=/path/to/data.yaml epochs=50 imgsz=640 batch=16 workers=2
```

Evaluate trên tập test:

```bash
yolo detect val model=/path/to/best.pt data=/path/to/data.yaml split=test imgsz=640
```

Export sang ONNX để deploy CPU/cloud nhẹ hơn:

```bash
yolo export model=/path/to/best.pt format=onnx imgsz=640
```

Repo cũng có script export:

```bash
python export_onnx.py
```

## Metric Đạt Được

Kết quả đánh giá mô hình YOLOv8s-AP trên tập kiểm thử:

| Class | Precision | Recall | mAP@50 | mAP@50-95 |
|---|---:|---:|---:|---:|
| All | 0.960 | 0.912 | 0.963 | 0.706 |
| Helmet | 0.965 | 0.907 | 0.969 | 0.647 |
| Head | 0.952 | 0.901 | 0.947 | 0.650 |
| Non-Helmet | 0.964 | 0.930 | 0.973 | 0.820 |

Theo báo cáo, mô hình hội tụ ổn định sau khoảng 20-30 epoch. Ma trận nhầm lẫn chuẩn hóa cho thấy tỷ lệ dự đoán đúng xấp xỉ 94% với `helmet`, 91% với `head` và 92% với `non-helmet`.

## Cài Đặt Và Chạy Local

Yêu cầu:

| Thành phần | Phiên bản khuyến nghị |
|---|---|
| Python | 3.11 |
| Node.js | 20+ |
| Model | `models/yolov8s_ap.onnx` hoặc file `.pt` tương thích |

Cài backend:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Chạy FastAPI:

```powershell
uvicorn src.api.main:app --host 0.0.0.0 --port 8000 --reload
```

Kiểm tra API:

```powershell
curl http://localhost:8000/health
```

Chạy frontend:

```powershell
cd frontend
npm install
npm run dev
```

Mặc định frontend dev server chạy tại:

```text
http://localhost:5173
```

## Inference Ảnh

Gọi API `/predict` để nhận JSON gồm tọa độ bounding box, confidence và class:

```powershell
curl -X POST "http://localhost:8000/predict" -F "file=@models/test/01.jpg"
```

Endpoint frontend đang dùng cho ảnh là:

```text
POST /api/detect/image
```

Endpoint này ngoài detection còn có thể lưu sự kiện vào lịch sử nếu phát hiện `head` hoặc `non-helmet`.

## Inference Video

Cách đơn giản nhất là chạy giao diện React, mở tab xử lý video và upload file video. Backend sẽ đọc video bằng OpenCV, lấy mẫu khoảng 10 FPS và dùng luật cửa sổ trượt 30 frame với ngưỡng 20/30 để giảm cảnh báo nhiễu.

Các API chính:

| Endpoint | Mục đích |
|---|---|
| `POST /api/detect_video` | Phân tích video offline và trả kết quả tổng hợp |
| `POST /api/upload-video` | Upload video để stream lại |
| `GET /api/stream/video` | Stream video đã vẽ bounding box dạng MJPEG |
| `GET /api/stream/video/alerts` | Lấy danh sách cảnh báo của video |

## Inference Camera

Hệ thống hỗ trợ webcam từ frontend và luồng camera qua RTSP/RTMP/HTTP.

Các API chính:

| Endpoint | Mục đích |
|---|---|
| `POST /api/live/webcam/frame` | Gửi từng frame webcam từ trình duyệt để inference |
| `POST /api/live/start` | Đăng ký URL camera hoặc stream |
| `GET /api/live/stream` | Nhận luồng MJPEG đã vẽ detection |
| `GET /api/live/alerts/{live_id}` | Lấy cảnh báo từ luồng camera |

Với camera IP, có thể nhập URL stream trong giao diện, ví dụ:

```text
rtsp://user:password@camera-ip:554/stream
```

## Deploy

### Docker

Build image:

```bash
docker build -t safety-helmet-detection .
```

Run container:

```powershell
docker run --rm -p 8000:7860 ^
  -e MODEL_PATH=/app/models/yolov8s_ap.onnx ^
  -v "%cd%/models:/app/models" ^
  -v "%cd%/data:/app/data" ^
  safety-helmet-detection
```

Trong Dockerfile, biến `PORT` mặc định là `7860` để phù hợp Hugging Face Spaces. Khi chạy local có thể override port theo nhu cầu.

### Docker Compose

Trong repo hiện tại Dockerfile nằm ở thư mục gốc. Nếu dùng `docker-compose.yml`, cần bảo đảm trường `build.dockerfile` đang trỏ đúng tới Dockerfile này, sau đó chạy:

```bash
docker compose up --build
```

Sau khi deploy, kiểm tra:

```text
http://localhost:8000/health
```

### Biến Môi Trường Quan Trọng

| Biến | Mặc định | Ý nghĩa |
|---|---|---|
| `MODEL_PATH` | `models/yolov8s_ap.onnx` | Đường dẫn model `.onnx`, `.pt` hoặc `.engine` |
| `CONFIDENCE_THRESHOLD` | `0.25` | Ngưỡng confidence khi inference |
| `IMAGE_SIZE` | `640` | Kích thước ảnh đưa vào model |
| `PORT` | `8000` local, `7860` Dockerfile | Port chạy API |
| `CORS_ORIGINS` | `http://localhost:5173,http://127.0.0.1:5173` | Domain frontend được phép gọi API |
| `LOG_LEVEL` | `INFO` | Mức log |

## Giới Hạn Của Model

Các giới hạn cần lưu ý khi dùng trong thực tế:

1. Độ chính xác có thể giảm khi ánh sáng yếu, ngược sáng, mưa bụi hoặc camera rung.
2. Các vùng đầu bị che khuất mạnh bởi vật liệu, máy móc hoặc người khác vẫn có thể bị bỏ sót.
3. Lớp `head` có ít mẫu hơn hai lớp còn lại, nên cần tiếp tục bổ sung dữ liệu cho các góc nhìn khó.
4. Lớp `non-helmet` có thể nhầm với một số vật thể có hình dáng giống mũ nếu dữ liệu huấn luyện chưa bao phủ đủ.
5. Tốc độ xử lý thời gian thực phụ thuộc nhiều vào phần cứng, đặc biệt là GPU, độ phân giải video và số lượng camera đồng thời.
6. Chức năng fine-tune tự động từ dữ liệu người dùng duyệt mới dừng ở mức thiết kế/quy trình, chưa nên xem là pipeline MLOps hoàn chỉnh.

## Hướng Cải Thiện

Các hướng phát triển hợp lý cho dự án:

1. Bổ sung dữ liệu thực tế tại nhiều công trường, nhiều điều kiện ánh sáng, thời tiết và góc camera.
2. Tăng mẫu cho lớp `head` và các trường hợp khó như đầu nhỏ, xa camera, bị che khuất một phần.
3. Đánh giá thêm trên video dài từ camera thật, không chỉ ảnh/test split tĩnh.
4. Tối ưu inference bằng ONNX Runtime GPU, TensorRT hoặc quantization nếu deploy trên edge device.
5. Hoàn thiện pipeline active learning: lưu mẫu nghi ngờ, duyệt nhãn, fine-tune theo lô và kiểm thử trước khi phát hành model mới.
6. Mở rộng nhận diện các PPE khác như áo phản quang, khẩu trang, găng tay hoặc dây an toàn.
7. Tích hợp hệ thống quản lý an toàn, dashboard thống kê và cảnh báo qua email/webhook.

## Tài Liệu Liên Quan

| File | Nội dung |
|---|---|
| `notebooks/report.pdf` | Báo cáo khóa luận, mô tả dataset, kiến trúc YOLOv8s-AP và kết quả thực nghiệm |
| `notebooks/Model_trainer.ipynb` | Notebook huấn luyện mô hình |
| `src/core/predictor.py` | Singleton predictor cho YOLO inference |
| `src/api/routes/inference.py` | API health và image prediction |
| `src/api/routes/stream.py` | API video, webcam và live stream |
| `src/config/settings.py` | Cấu hình runtime và biến môi trường |
