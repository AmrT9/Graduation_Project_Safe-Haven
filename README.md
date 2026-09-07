# SafeHaven 🛡️ — Privacy-Preserving Edge AI Safety Monitoring

An edge-AI safety monitoring system that detects **falls** and **silent help-gesture requests** in real time — combining Computer Vision, IoT, a cloud backend, and a cross-platform mobile app — without relying on wearables or continuous cloud video surveillance.

## Problem

Elderly individuals, children, and people living alone often need fall/emergency detection but reject continuous camera surveillance or wearables for privacy and comfort reasons. SafeHaven's goal: detect a fall or a silent request for help in real time, on-device, without ever streaming raw video to the cloud unless an incident is confirmed.

## Architecture

A three-layer architecture balancing real-time performance, scalability, and privacy:

```
                ┌─────────────────────┐
                │   Raspberry Pi 4    │   Edge Layer
                │   (camera + AI)     │   - pose/hand landmark extraction
                └──────────┬──────────┘   - fall + gesture inference
                           │                - local alert generation & queuing
                    Camera Input
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
       MediaPipe Pose            MediaPipe Hands
              │                         │
              ▼                         ▼
        LSTM Model              Gesture Classifier
              │                         │
              └────────────┬────────────┘
                           ▼
                 Alert Decision Module
                           │
                           ▼
               FastAPI + WebSockets (real-time)   Communication Layer
                           │
                       Supabase (auth, DB, storage, realtime)
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
      Firebase Cloud Messaging   Mobile Application       Application Layer
                  │                 │       (React Native + Expo + TypeScript)
                  └───────► Alerts ◄┘       Care Receiver / Care Giver / Admin roles
```

**Edge layer:** Raspberry Pi 4 captures video, extracts pose/hand landmarks, runs both AI models locally, and only ever sends confirmed-incident footage upstream — raw video never leaves the device continuously.

## Dataset

**Fall detection:** a custom dataset covering walking, sitting, lying down, and forward/backward/lateral falls across multiple camera angles, lighting conditions, and room layouts. Final MediaPipe Pose + LSTM evaluation used **5,939 labeled sequences** (3,451 ADL / 2,488 fall).

**Help gesture:** 300 positive + 300 negative samples of a predefined Y-shaped ASL help gesture, represented as 21 hand landmarks × 3 coordinates (63-dim feature vector), normalized relative to the wrist.

## Tech Stack

| Layer | Tools |
|---|---|
| AI / CV | Python, PyTorch, TensorFlow Lite, MediaPipe, Ultralytics YOLO, OpenCV, LSTM |
| Edge | Raspberry Pi 4 (4GB), Raspberry Pi Camera Module 3, Raspberry Pi OS 64-bit |
| Mobile | React Native, Expo, TypeScript |
| Backend / Cloud | Supabase (PostgreSQL, auth, storage, realtime), FastAPI, WebSockets, Firebase Cloud Messaging |

## How to Run
1. **Edge node:** deploy the AI pipeline (pose/gesture models + alert logic) to a Raspberry Pi 4 with the camera module attached.
2. **Backend:** provision a Supabase project (DB schema + auth) and a FastAPI service for real-time WebSocket communication.
3. **Mobile app:** `expo start` from the `app/` directory, pointed at your Supabase/FastAPI endpoints.

## Results

**Fall detection** (MediaPipe Pose + LSTM, final model):

| Metric | Result |
|---|---|
| Accuracy | 90.35% |
| Precision | 86.26% |
| Recall | 91.56% |
| F1-Score | 88.83% |

**Help gesture recognition:**

| Metric | Result |
|---|---|
| Accuracy | 98.81% |
| Precision | 98.84% |
| Recall | 98.81% |
| F1-Score | 98.81% |

**Model comparison during development:**

| Approach | Accuracy | Recall | Status |
|---|---|---|---|
| ROI + YOLOv10 | 85.0% | 87.0% | Abandoned |
| YOLOv8 + LSTM | 81.84% | 76% | Baseline |
| **MediaPipe Pose + LSTM** | **90.35%** | **91.56%** | **Final model** |

**Edge performance (Raspberry Pi 4, 4GB RAM):**

| Pipeline | Latency | FPS |
|---|---|---|
| Help gesture recognition | ~25 ms | ~40 |
| Fall detection | ~60 ms | ~16–17 |
| Combined pipeline | ~75 ms | ~13–14 |

> ⚠️ **Metric discrepancy resolved:** the graduation report's abstract states a fall-detection recall of 96.7%, which conflicts with the 91.56% reported in the results table above. The 91.56% figure is the one to use — it's internally consistent with the report's own claim of a "20.5% relative improvement over the YOLOv8+LSTM baseline (76% recall)": 91.56/76 = 1.205 (a 20.5% relative gain), while 96.7/76 would be a ~27% gain. The 96.7% in the abstract appears to be a stale number from an earlier draft that was never updated — fix it in the `.docx` report before submitting/sharing it further.

## Privacy by Design

AI processing happens locally on the Raspberry Pi. The system works primarily with skeletal keypoints, hand landmarks, and event metadata — not raw video — and only transmits footage when a confirmed incident requires it.

## Current Limitations

- Assumes a single monitored person (no multi-person tracking yet)
- Severe occlusion can affect pose landmark detection
- Very low-light environments may need IR-assisted cameras
- Some rapid non-fall movements may trigger false alerts
- Remote notification still depends on network availability

## What I Personally Built

I was responsible for the AI models at the core of the system:

- **Fall detection model:** the MediaPipe Pose + LSTM pipeline — keypoint extraction, temporal sequence construction, and the two-layer LSTM classifier that reached 91.56% recall.
- **Help gesture recognition model:** the MediaPipe Hands pipeline and the neural network classifier over the 63-dimensional hand-landmark feature vector, reaching 98.81% accuracy.
- **Model iteration:** evaluated and compared the earlier ROI+YOLOv10 and YOLOv8+LSTM approaches against the final MediaPipe+LSTM model to justify the final architecture choice on the accuracy/recall/edge-efficiency trade-off.

---

**Team:** Shaden Mohamed Abdelsalam, Merna Khaled Mahmoud, Nouran Adel Ahmed, Amr Tarek Ismail, Gamal Mohamed, Omar Ashraf
**Supervisor:** Dr. Ahmed Fathy Alnokrashy
**Academic Project:** Senior Graduation Project, Nile University — School of Information Technology and Computer Science

## Disclaimer

SafeHaven is an academic research and engineering prototype. It is not intended to replace professional medical monitoring, emergency services, or clinical safety systems.
