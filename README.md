---
title: Safety Helmet Detection
emoji: 👷
colorFrom: yellow
colorTo: gray
sdk: docker
pinned: false
---

<div align="center">

# High-Performance Safety Helmet Detection System

**A full-stack AI production system for real-time PPE compliance monitoring**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Hugging%20Face%20Spaces-FFD21E?style=for-the-badge)](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://react.dev)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com)
[![YOLOv8](https://img.shields.io/badge/YOLOv8-ONNX-purple?style=for-the-badge)](https://docs.ultralytics.com)

</div>

---

## Tiếng Việt

### Demo trực tuyến

**[tr-th-khoi/safety-helmet-detection trên Hugging Face Spaces](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)**

### Tổng quan

Safety Helmet Detection là hệ thống AI full-stack giám sát việc đội mũ bảo hộ tại công trường. Ứng dụng gồm frontend React + Vite, backend FastAPI, mô hình YOLOv8s-AP chạy bằng ONNX Runtime và Docker để triển khai thống nhất.

| Nhóm chức năng | Nội dung |
|----------------|----------|
| Ảnh / video / camera | Nhận diện trên ảnh tải lên, video ngoại tuyến và luồng webcam/IP camera/RTSP |
| Cảnh báo vi phạm | Theo dõi đối tượng qua nhiều frame, ghi nhận `head` và `non_helmet` làm sự kiện vi phạm/nghi ngờ |
| Lịch sử và báo cáo | Lưu ảnh toàn cảnh, ảnh crop, lịch sử JSONL, báo cáo AI tiếng Việt và PDF |
| Tối ưu triển khai | Mô hình `.onnx`, static frontend phục vụ cùng FastAPI, hạn chế CORS và giảm cold start |

### Kiến trúc hệ thống

```text
Browser
  |
  | React + Vite UI
  v
FastAPI backend
  |-- Image/video/live-stream APIs
  |-- History, alert, report, PDF APIs
  |-- Static frontend serving
  v
YOLOv8s-AP ONNX inference + ByteTrack
  |
  +-- Optional Gemini report generation
```

Luồng xử lý chính: frontend gửi ảnh, video hoặc URL camera đến FastAPI; backend giải mã dữ liệu, chạy suy luận ONNX, theo dõi vi phạm qua nhiều frame, lưu sự kiện và trả kết quả JSON hoặc MJPEG stream về giao diện.

### Mô hình, dữ liệu và kết quả

Mô hình chính là **YOLOv8s-AP**, xây dựng trên YOLOv8s và bổ sung **ASFF** ở phần neck cùng **Pseudo-DCNv2** ở detection head để cải thiện nhận diện mũ/vùng đầu nhỏ, bị che khuất hoặc biến dạng trong bối cảnh công trường.

Nguồn số liệu: [`notebooks/report.pdf`](notebooks/report.pdf) và [`notebooks/Model_trainer.ipynb`](notebooks/Model_trainer.ipynb).

| Hạng mục | Giá trị |
|----------|--------:|
| Tổng ảnh | 19.212 |
| Tổng bounding boxes | 43.507 |
| Train / Validation / Test | 15.369 / 1.921 / 1.922 |
| Epochs / Batch / Image size | 50 / 16 / 640x640 |
| Môi trường train | Kaggle, NVIDIA Tesla P100 16 GB, Python 3.10 |

| Lớp | Số bbox | Precision | Recall | mAP@50 | mAP@50-95 |
|-----|--------:|----------:|-------:|-------:|----------:|
| All | 43.507 | 0.960 | 0.912 | 0.963 | 0.706 |
| `helmet` | 19.089 | 0.965 | 0.907 | 0.969 | 0.647 |
| `head` | 5.875 | 0.952 | 0.901 | 0.947 | 0.650 |
| `non_helmet` | 18.543 | 0.964 | 0.930 | 0.973 | 0.820 |

Mô hình được export từ `.pt` sang `.onnx` bằng `export_onnx.py` để giảm phụ thuộc PyTorch, rút ngắn cold start và phù hợp môi trường cloud CPU-only.

---

## English

### Live Demo

**[tr-th-khoi/safety-helmet-detection on Hugging Face Spaces](https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection)**

### Overview

Safety Helmet Detection is a full-stack AI system for construction-site helmet compliance monitoring. The app combines a React + Vite frontend, a FastAPI backend, YOLOv8s-AP inference through ONNX Runtime, and Docker-based deployment.

| Feature group | Scope |
|---------------|-------|
| Image / video / camera | Detect on uploaded images, offline videos, and webcam/IP camera/RTSP streams |
| Violation alerts | Track objects across frames and log `head` or `non_helmet` as violation/suspicion events |
| History and reports | Store full-frame evidence, cropped evidence, JSONL history, Vietnamese AI reports, and PDFs |
| Deployment optimization | `.onnx` model, frontend served by FastAPI, simpler CORS, and lower cold-start overhead |

### System Architecture

```text
Browser
  |
  | React + Vite UI
  v
FastAPI backend
  |-- Image/video/live-stream APIs
  |-- History, alert, report, PDF APIs
  |-- Static frontend serving
  v
YOLOv8s-AP ONNX inference + ByteTrack
  |
  +-- Optional Gemini report generation
```

Main flow: the frontend sends an image, video, or camera URL to FastAPI; the backend decodes the input, runs ONNX inference, tracks violations across frames, persists events, and returns JSON results or an MJPEG stream to the UI.

### Model, Data, and Results

The main detector is **YOLOv8s-AP**, built on YOLOv8s with **ASFF** in the neck and **Pseudo-DCNv2** in the detection head to improve detection of small, occluded, or deformed helmet/head regions in construction-site scenes.

Sources: [`notebooks/report.pdf`](notebooks/report.pdf) and [`notebooks/Model_trainer.ipynb`](notebooks/Model_trainer.ipynb).

| Item | Value |
|------|------:|
| Total images | 19,212 |
| Total bounding boxes | 43,507 |
| Train / Validation / Test | 15,369 / 1,921 / 1,922 |
| Epochs / Batch / Image size | 50 / 16 / 640x640 |
| Training environment | Kaggle, NVIDIA Tesla P100 16 GB, Python 3.10 |

| Class | BBoxes | Precision | Recall | mAP@50 | mAP@50-95 |
|-------|-------:|----------:|-------:|-------:|----------:|
| All | 43,507 | 0.960 | 0.912 | 0.963 | 0.706 |
| `helmet` | 19,089 | 0.965 | 0.907 | 0.969 | 0.647 |
| `head` | 5,875 | 0.952 | 0.901 | 0.947 | 0.650 |
| `non_helmet` | 18,543 | 0.964 | 0.930 | 0.973 | 0.820 |

The model is exported from `.pt` to `.onnx` with `export_onnx.py` to remove the heavy PyTorch runtime from deployment, reduce cold start, and fit CPU-only cloud environments.

---

## Project Structure

```
.
├── src/                    # Backend — FastAPI application
│   ├── api/
│   │   ├── routes/         #   inference, history, stream, update, report
│   │   ├── schemas.py      #   Pydantic models (all request/response types)
│   │   └── main.py         #   App factory, lifespan, CORS, routers
│   ├── core/
│   │   ├── predictor.py    #   ONNX singleton loader (helmet model)
│   │   ├── person_detector.py  #   Singleton YOLOv8n person detector + ByteTrack
│   │   ├── person_first.py     #   Two-stage pipeline: person → helmet crop
│   │   ├── streaming.py    #   MJPEG streaming, ByteTrack, ViolationTracker
│   │   ├── history.py      #   JSONL event log + image persistence
│   │   ├── llm_reporter.py #   Gemini 2.0 Flash report generator (rate-limited)
│   │   ├── pdf_builder.py  #   reportlab PDF builder (instant + scheduled)
│   │   └── scheduler.py    #   APScheduler auto-report every 4h
│   ├── config/             #   Environment-driven settings, dotenv loader
│   └── utils/              #   Image decode, detection helpers
├── frontend/               # React + Vite frontend (TypeScript)
│   └── src/
│       ├── components/
│       │   ├── ImageDetection.tsx
│       │   ├── VideoTracking.tsx   # + "Xem báo cáo" + "Tải PDF" buttons
│       │   ├── LiveStream.tsx      # + "Xem báo cáo" + "Tải PDF" buttons
│       │   ├── AlertDetailModal.tsx
│       │   └── ReportModal.tsx     # AI report display (risk badge + recommendations)
│       ├── content/
│       │   └── app-text.vi.json   # Vietnamese i18n strings
│       └── services/api.ts
├── notebooks/
│   ├── Model_trainer.ipynb # Kaggle training notebook (dataset merge → YOLOv8s-AP train)
│   └── report.pdf          # Thesis report with dataset, training, and evaluation details
├── models/                 # Model weights (yolov8s_ap.onnx) — gitignored
├── demo/                   # Real-world validation samples
├── Dockerfile              # Multi-stage build: Node (React) → Python (FastAPI)
├── docker-compose.yml
├── export_onnx.py
└── requirements.txt
```

---

## Installation & Running

### Recommended — Docker

```bash
git clone https://huggingface.co/spaces/tr-th-khoi/safety-helmet-detection
cd safety-helmet-detection
cp .env.example .env   # add GEMINI_API_KEY for AI reports (optional)
docker compose up --build
```

Open `http://localhost:8000` in your browser.

### Alternative — Dev Servers (hot-reload)

**Terminal 1 — Backend:**

```bash
python -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # edit MODEL_PATH, GEMINI_API_KEY, etc.
uvicorn src.api.main:app --reload --port 8000
```

**Terminal 2 — Frontend:**

```bash
cd frontend
npm install
npm run dev
```

Create `frontend/.env.local` to point the frontend at the local backend:

```env
VITE_API_BASE=http://localhost:8000
```

---

## API Reference

Rate limits are enforced per IP address (no API key required):

| Endpoint | Limit |
|----------|-------|
| `POST /predict` | 30 req/min |
| `POST /api/detect_video` | 5 req/min |
| `POST /api/live/webcam/frame` | 600 req/min |

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Liveness check — reports model load status |
| `POST` | `/predict` | Single-image inference → JSON detections |
| `POST` | `/api/detect/image` | Detect + persist event to history |
| `GET` | `/api/history` | List recent violation events |
| `POST` | `/api/upload-video` | Upload video, returns `video_id` |
| `GET` | `/api/stream/video` | MJPEG stream with ByteTrack overlays |
| `GET` | `/api/stream/video/alerts` | Poll confirmed violation alerts for a video |
| `POST` | `/api/live/webcam/frame` | Process single webcam frame → detections + alerts |
| `GET` | `/api/live/alerts/{live_id}` | Poll alerts for an IP camera stream |
| `POST` | `/api/report/from-alert` | Generate AI report for a live/video alert (cached) |
| `GET` | `/api/report/event/{event_id}` | Generate AI report for a history event |
| `POST` | `/api/report/simple-pdf` | **Instant PDF** from current session alerts (no LLM) |
| `GET` | `/api/report/download` | Full PDF report from history (last N hours) |

---

## Configuration

All runtime behaviour is controlled by environment variables (`.env` file):

| Variable | Default | Description |
|----------|---------|-------------|
| `MODEL_PATH` | `models/yolov8s_ap.onnx` | Path to helmet model weights |
| `CONFIDENCE_THRESHOLD` | `0.25` | Minimum detection confidence |
| `PORT` | `7860` | Server port (Hugging Face Spaces requires 7860) |
| `LOG_LEVEL` | `INFO` | Python logging level |
| `CORS_ORIGINS` | `http://localhost:5173` | Comma-separated list of allowed origins |
| `ONNX_EXECUTION_PROVIDER` | _(auto)_ | ONNX Runtime provider: empty = auto-detect; set to `CPUExecutionProvider` to force CPU or `CUDAExecutionProvider` to force GPU (requires `onnxruntime-gpu`) |
| `GEMINI_API_KEY` | _(empty)_ | Google Gemini API key for AI reports — free at [aistudio.google.com](https://aistudio.google.com/apikey) |
| `REPORT_INTERVAL_HOURS` | `4` | Auto PDF generation interval (hours) |

> If `GEMINI_API_KEY` is not set, the system still works — "Xem báo cáo" returns a pre-built fallback report instead of calling Gemini.

### Person-First Pipeline Variables

Only relevant when `PERSON_FIRST_ENABLED=true`:

| Variable | Default | Description |
|----------|---------|-------------|
| `PERSON_FIRST_ENABLED` | `false` | Enable two-stage person → helmet pipeline |
| `PERSON_MODEL_PATH` | _(auto)_ | Path to person detector weights; supports `.pt` and `.onnx`; omit to auto-download `yolov8n.pt` |
| `PERSON_CONFIDENCE_THRESHOLD` | `0.40` | Minimum confidence to accept a person detection |
| `PERSON_DETECTION_INTERVAL` | `1` | Run person detector every N processed frames (default=1, full accuracy; increase to 2–4 on weak CPU to reduce load with minor temporal lag) |
| `HELMET_RECHECK_INTERVAL` | `5` | Re-run helmet model every N frames per tracked person |
| `PERSON_CROP_MARGIN` | `0.05` | Fractional padding added around each person crop |
| `UPPER_BODY_CROP_RATIO` | `0.60` | Top fraction of person bbox used as helmet crop |

**Using an ONNX person model** (faster CPU inference):
```bash
python -c "from ultralytics import YOLO; YOLO('yolov8n.pt').export(format='onnx', imgsz=640)"
mv yolov8n.onnx models/
```
Then set `PERSON_MODEL_PATH=models/yolov8n.onnx`. The backend auto-detects the format and selects the appropriate device.

---

## Engineering Notes
- **Frontend demo refresh (2025-04)** — UI shifted to a light `#9fe7f5`-led visual system with DM Sans throughout. Hero shows a single-line title + 4 stat cards; a 5-step pipeline diagram accurately describes the default single-stage flow with a note that ByteTrack/rolling-window (step 4) is Video/Live only; a dedicated callout card inside the pipeline section explains the person-first 2-stage variant (YOLOv8n → crop → YOLOv8s) and clarifies it is opt-in via `PERSON_FIRST_ENABLED=true` and applies to Video, Live Stream, and Webcam — not image-only requests; real demo images are embedded as a showcase section; three mode-selector cards explain each detection mode. Existing component functionality unchanged.
- **Singleton model loader** — the ONNX model is loaded once at startup and reused across all requests; no per-request overhead.
- **Same-origin deployment** — on Hugging Face Spaces, React static files are served by FastAPI itself, so all API calls use relative paths with no hardcoded URLs or CORS issues.
- **ByteTrack confirmation window** — violations are only alerted after a rolling 3-of-5 frame window, eliminating single-frame false positives.
- **Deferred PIL conversion** — `cv2.cvtColor + Image.fromarray` in the streaming loop runs only when a threshold crossing actually triggers `persist_window_event`, not on every processed frame.
- **ONNX provider auto-detection** — both the helmet predictor and the person detector check `onnxruntime.get_available_providers()` at model load and select CUDAExecutionProvider automatically if available; no manual config required for GPU machines.
- **Gemini rate limiter** — a global threading lock enforces ≤12 calls/minute (safely under the free-tier 15 RPM limit); in-flight deduplication prevents redundant calls for the same alert.
- **Instant PDF** — `POST /api/report/simple-pdf` accepts raw alert data from the frontend and builds a PDF immediately without history lookup or LLM calls; image aspect ratios are preserved.
- **Runtime data isolation** — detection history and uploaded videos are written to `/app/data` inside the container (gitignored); only the model weights and source code are version-controlled.
- **Filter-aware violation alerts** — the `classes` URL parameter passed at stream start is applied to both the visual overlay and the alert store; only classes matching the user's filter selection are saved in `VIDEO_ALERTS` / `LIVE_ALERTS`, so polled alerts are already pre-filtered. For webcam, the frontend filters `data.alerts` client-side using the active class set before adding to state.
- **Per-session person-first (webcam)** — `person_first` is sent as a form field with every webcam frame POST. A new session initialises a `PersonFirstPipeline` when `person_first=true` regardless of the global `PERSON_FIRST_ENABLED` env var, so the UI toggle works independently of server configuration.
- **Active filter indicator** — after clicking "Bắt đầu nhận dạng" on Video or Webcam, a badge strip appears beneath the stream header listing every class currently being filtered, so the active filter is always visible during inference.
