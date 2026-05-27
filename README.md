---
title: Safety Helmet Detection
emoji: 👷
colorFrom: yellow
colorTo: gray
sdk: docker
pinned: false
---

<div align="center">

# Safety Helmet Detection

**A full-stack AI system for real-time safety helmet compliance monitoring**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Hugging%20Face%20Spaces-FFD21E?style=for-the-badge)](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://react.dev)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com)
[![YOLOv8](https://img.shields.io/badge/YOLOv8-ONNX-purple?style=for-the-badge)](https://docs.ultralytics.com)

</div>

---

## Tiếng Việt

### Giới thiệu

Safety Helmet Detection là hệ thống AI full-stack hỗ trợ giám sát việc đội mũ bảo hộ trong môi trường công trường. Ứng dụng sử dụng mô hình YOLOv8s-AP để nhận diện các đối tượng `helmet`, `head` và `non_helmet` trên ảnh, video và luồng camera, từ đó phát hiện các trường hợp có nguy cơ vi phạm an toàn lao động.

Mục tiêu của dự án là tự động hóa quá trình kiểm tra tuân thủ PPE, giảm phụ thuộc vào giám sát thủ công và cung cấp bằng chứng trực quan cho các tình huống không đội mũ bảo hộ hoặc vùng đầu không được bảo vệ.

### Chức năng chính

| Nhóm chức năng | Mô tả |
|----------------|-------|
| Nhận diện ảnh | Phân tích ảnh tải lên và trả về vị trí, độ tin cậy, nhãn lớp của từng đối tượng |
| Nhận diện video | Xử lý video ngoại tuyến, hiển thị bounding box và theo dõi vi phạm qua nhiều frame |
| Nhận diện camera | Hỗ trợ webcam, IP camera hoặc RTSP stream cho bài toán giám sát gần thời gian thực |
| Cảnh báo vi phạm | Ghi nhận `head` và `non_helmet` làm sự kiện vi phạm hoặc nghi ngờ vi phạm |
| Lịch sử và báo cáo | Lưu ảnh bằng chứng, lịch sử phát hiện và tạo báo cáo hỗ trợ đánh giá an toàn |

### Công nghệ sử dụng

| Thành phần | Công nghệ |
|------------|-----------|
| Mô hình AI | YOLOv8s-AP, YOLOv8, ONNX |
| Backend | FastAPI, Python, OpenCV |
| Frontend | React, Vite, TypeScript |
| Suy luận | ONNX Runtime, hỗ trợ tự động chọn CPU/GPU tùy môi trường |
| Triển khai | Docker, Docker Compose, Hugging Face Spaces |

### Mô hình và kết quả

Mô hình chính là **YOLOv8s-AP**, được phát triển từ YOLOv8s và cải tiến bằng ASFF ở phần neck cùng Pseudo-DCNv2 ở detection head. Thiết kế này giúp cải thiện khả năng nhận diện mũ bảo hộ, vùng đầu nhỏ, bị che khuất hoặc biến dạng trong bối cảnh công trường.

Mô hình được export từ `.pt` sang `.onnx` để giảm phụ thuộc vào PyTorch, rút ngắn thời gian khởi động và phù hợp hơn với môi trường triển khai cloud CPU-only.

| Chỉ số | Giá trị |
|--------|--------:|
| Tổng ảnh | 19.212 |
| Tổng bounding boxes | 43.507 |
| Train / Validation / Test | 15.369 / 1.921 / 1.922 |
| Precision | 0.960 |
| Recall | 0.912 |
| mAP@50 | 0.963 |
| mAP@50-95 | 0.706 |

### Demo trực tuyến

Trải nghiệm hệ thống tại: **[tr-th-khoi/safety-helmet-detection trên Hugging Face Spaces](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)**.

---

## English

### Introduction

Safety Helmet Detection is a full-stack AI system for monitoring safety helmet compliance in construction environments. The application uses a YOLOv8s-AP model to detect `helmet`, `head`, and `non_helmet` objects in images, videos, and camera streams, helping identify potential workplace safety violations.

The goal of the project is to automate PPE compliance inspection, reduce dependence on manual supervision, and provide visual evidence for cases where workers are not wearing helmets or where the head area is not properly protected.

### Main Features

| Feature group | Description |
|---------------|-------------|
| Image detection | Analyze uploaded images and return bounding boxes, confidence scores, and class labels |
| Video detection | Process offline videos, draw detection boxes, and track violations across frames |
| Camera detection | Support webcam, IP camera, or RTSP streams for near real-time monitoring |
| Violation alerts | Log `head` and `non_helmet` detections as violation or suspected violation events |
| History and reports | Store evidence images, detection history, and safety assessment reports |

### Technology Stack

| Component | Technology |
|-----------|------------|
| AI model | YOLOv8s-AP, YOLOv8, ONNX |
| Backend | FastAPI, Python, OpenCV |
| Frontend | React, Vite, TypeScript |
| Inference | ONNX Runtime, with automatic CPU/GPU provider selection |
| Deployment | Docker, Docker Compose, Hugging Face Spaces |

### Model and Results

The main detector is **YOLOv8s-AP**, built from YOLOv8s and enhanced with ASFF in the neck and Pseudo-DCNv2 in the detection head. This architecture improves detection of small, occluded, or deformed helmet and head regions in construction-site scenes.

The model is exported from `.pt` to `.onnx` to reduce PyTorch runtime dependency, shorten cold start time, and fit CPU-only cloud deployment environments.

| Metric | Value |
|--------|------:|
| Total images | 19,212 |
| Total bounding boxes | 43,507 |
| Train / Validation / Test | 15,369 / 1,921 / 1,922 |
| Precision | 0.960 |
| Recall | 0.912 |
| mAP@50 | 0.963 |
| mAP@50-95 | 0.706 |

### Live Demo

Try the application at: **[tr-th-khoi/safety-helmet-detection on Hugging Face Spaces](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)**.
