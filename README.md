# SafeHaven 🛡️

### Privacy-Preserving Edge AI System for Real-Time Safety Monitoring

SafeHaven is an intelligent, privacy-preserving safety monitoring system designed to detect critical events such as **falls** and **silent requests for help** in real time.

The system combines **Edge AI, Computer Vision, Deep Learning, IoT, Cloud Backend Services, and a Cross-Platform Mobile Application** to provide continuous safety monitoring without relying on wearable devices or continuous cloud video surveillance.

SafeHaven is designed for elderly individuals, children, and people who may require immediate assistance in unsupervised environments.

---

## 🚀 Key Features

- 🧠 Real-time AI-based fall detection
- ✋ Silent emergency help gesture recognition
- 🔒 Privacy-preserving edge processing
- ⚡ Real-time emergency alerts
- 📱 Cross-platform mobile application
- 👥 Role-based access control
- 🎥 Secure incident video capture and playback
- 📡 Real-time communication between edge devices and mobile clients
- ☁️ Cloud backend and persistent alert management
- 🔄 Local alert queuing during temporary network outages
- 🖥️ Edge AI deployment on Raspberry Pi

---

## 🏗️ System Architecture

SafeHaven follows a three-layer architecture designed to balance **real-time performance, scalability, and privacy**.

### 1. Edge Layer

A Raspberry Pi acts as the intelligent edge node.

The edge device is responsible for:

- Capturing live video using the Raspberry Pi Camera Module
- Extracting pose and hand landmarks
- Running AI-based fall detection
- Running emergency gesture recognition
- Performing local event validation
- Generating alerts
- Temporarily queuing alerts during network outages
- Processing sensitive visual data locally

Raw video is not continuously transmitted to the cloud.

---

### 2. Communication Layer

The system uses multiple communication technologies:

- **FastAPI + WebSockets** for real-time bidirectional communication
- **HTTPS** for secure backend communication
- **Supabase** for authentication, database services, storage, and real-time synchronization
- **Firebase Cloud Messaging (FCM)** for push notifications

---

### 3. Application Layer

The end-user interacts with SafeHaven through a cross-platform mobile application.

The application provides:

- Real-time alerts
- Incident history
- Secure incident video playback
- User management
- Monitoring zone configuration
- Device health monitoring
- Role-based access control
- Care Receiver and Care Giver interfaces

---

## 🤖 AI Pipeline

SafeHaven contains two independent AI pipelines:

### Fall Detection Pipeline

```text
Camera
   ↓
Pose Landmark Extraction
   ↓
Keypoint Normalization
   ↓
30-Frame Temporal Sequence
   ↓
LSTM Classifier
   ↓
Fall / Normal Activity
   ↓
Alert Decision

The final deployed fall detection approach uses MediaPipe Pose + LSTM.

Instead of relying directly on raw images, the system analyzes human skeletal landmarks to reduce computational requirements and improve privacy.

The model processes 33 skeletal landmarks per frame, represented by:

(x, y, z, visibility)

A two-layer LSTM processes 30-frame temporal sequences.

Final Fall Detection Results
Metric	Result
Accuracy	90.35%
Precision	86.26%
Recall	91.56%
F1-Score	88.83%

The high recall is particularly important for safety monitoring because missed fall events can have serious consequences.

✋ Help Gesture Recognition

SafeHaven provides a silent emergency mechanism through a predefined Y-shaped ASL help gesture.

The pipeline is:

Camera
   ↓
MediaPipe Hands
   ↓
21 Hand Landmarks
   ↓
63-Dimensional Feature Vector
   ↓
Neural Network Classifier
   ↓
Help / Normal Gesture
   ↓
Alert Decision

The model uses the coordinates of the 21 hand landmarks rather than storing or processing raw images for classification.

Help Gesture Results
Metric	Result
Accuracy	98.81%
Precision	98.84%
Recall	98.81%
F1-Score	98.81%
🔐 Privacy by Design

Privacy is one of the core principles of SafeHaven.

Instead of continuously uploading camera footage to cloud servers, AI processing is performed locally on the Raspberry Pi.

The system primarily works with:

Skeletal keypoints
Hand landmarks
Event metadata
Confirmed incident media

This significantly reduces the exposure of sensitive visual information.

Raw video remains at the edge and is only transmitted when a confirmed incident requires associated footage.

📱 Mobile Application

The SafeHaven mobile application was developed using:

React Native
Expo
TypeScript

The application supports role-based access control and provides different capabilities for different users.

Care Receiver
Receive system interaction and monitoring services
Trigger silent help requests using the emergency gesture
Access relevant application features based on assigned permissions
Care Giver / Security Personnel
Receive real-time emergency alerts
Review incident history
View incident footage
Monitor connected devices
Respond to detected events
Administrator
Manage users
Configure monitoring zones
Manage edge devices
Monitor device health
Manage system configuration
☁️ Backend Infrastructure

SafeHaven uses Supabase as its Backend-as-a-Service platform.

The backend provides:

PostgreSQL database
User authentication
Role-based access control
Real-time subscriptions
Secure media storage
Alert persistence
Device management

Firebase Cloud Messaging is used to deliver push notifications to registered mobile devices.

The database follows a normalized relational design based on Third Normal Form (3NF).

Main entities include:

Users
Devices
Alerts
Zones
Media
⚡ Edge Performance

SafeHaven was evaluated on a Raspberry Pi 4 with 4 GB RAM.

Hardware
Raspberry Pi 4
4 GB RAM
Raspberry Pi Camera Module 3
32 GB microSD Card
Wi-Fi 802.11ac
Raspberry Pi OS 64-bit
Software
Python
PyTorch
OpenCV
MediaPipe
Ultralytics
TensorFlow Lite
React Native
TypeScript
Supabase
FastAPI
Firebase Cloud Messaging
Inference Performance
Pipeline	Latency	FPS
Help Gesture Recognition	~25 ms	~40 FPS
Fall Detection	~60 ms	~16–17 FPS
Combined Pipeline	~75 ms	~13–14 FPS

The combined pipeline maintains practical real-time performance while running both detection systems simultaneously on the Raspberry Pi.

🧪 Dataset
Fall Detection Dataset

A custom fall detection dataset was created to represent different human activities and fall scenarios.

The dataset includes:

Walking
Sitting
Lying down
Forward falls
Backward falls
Lateral falls
Different camera angles
Different lighting conditions
Different room layouts

The final MediaPipe Pose + LSTM evaluation used 5,939 labeled sequences, including:

3,451 ADL sequences
2,488 Fall sequences
Help Gesture Dataset

The gesture recognition dataset contains:

300 positive samples
300 negative samples

The system uses MediaPipe hand landmarks instead of raw image data.

Each hand is represented using:

21 landmarks × 3 coordinates = 63 features

Coordinates are normalized relative to the wrist to improve translation invariance.

🔄 System Workflow
                ┌─────────────────────┐
                │   Raspberry Pi      │
                │     Edge Node       │
                └──────────┬──────────┘
                           │
                    Camera Input
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       MediaPipe Pose            MediaPipe Hands
              │                         │
              ▼                         ▼
        LSTM Model              Gesture Model
              │                         │
              └────────────┬────────────┘
                           ▼
                 Alert Decision Module
                           │
                           ▼
                  FastAPI / WebSocket
                           │
                           ▼
                       Supabase
                           │
                  ┌────────┴────────┐
                  │                 │
                  ▼                 ▼
                 FCM          Mobile Application
                  │                 │
                  └───────► Alerts ◄┘
🧠 Model Development

Multiple approaches were evaluated during development.

Approach	Accuracy	Recall	Status
ROI + YOLOv10	85.0%	87.0%	Abandoned
YOLOv8 + LSTM	81.84%	76%	Baseline
MediaPipe Pose + LSTM	90.35%	91.56%	Final Model
Why the Final Model?

The MediaPipe Pose + LSTM approach provided the best balance between:

Detection accuracy
Fall recall
Computational efficiency
Privacy
Raspberry Pi deployment feasibility

The earlier YOLO-based approaches were more computationally demanding and were not suitable for continuous edge deployment.

📂 Project Structure
SafeHaven/
│
├── app/                    # React Native mobile application
│
├── src/                    # Application components,
│                           # providers, themes and utilities
│
├── integration/            # Edge/backend integration
│
├── supabase/               # Database migrations and backend functions
│
├── assets/                 # Application assets
│
├── package.json            # Project dependencies
├── package-lock.json
├── app.json
├── tsconfig.json
│
└── README.md
🛠️ Technology Stack
Artificial Intelligence & Computer Vision
Python
PyTorch
TensorFlow Lite
MediaPipe
Ultralytics YOLO
OpenCV
LSTM
Deep Neural Networks
Edge Computing
Raspberry Pi 4
Raspberry Pi Camera Module 3
Raspberry Pi OS
Mobile Development
React Native
Expo
TypeScript
Backend & Cloud
Supabase
PostgreSQL
FastAPI
WebSockets
Firebase Cloud Messaging
REST APIs
HTTPS
Development Tools
Git
GitHub
Git LFS
🎯 Project Objectives

SafeHaven was developed to:

Detect falls automatically without wearable devices.
Provide a silent mechanism for users to request help.
Perform AI processing locally to protect privacy.
Deliver real-time alerts to authorized users.
Provide a centralized mobile interface for monitoring and response.
Deploy AI models on affordable edge hardware.
Maintain system operation during temporary network failures.
📊 Key Results
Fall Detection

90.35% Accuracy

91.56% Recall

88.83% F1-Score

Help Gesture Recognition

98.81% Accuracy

98.81% Recall

Edge Performance

~75 ms combined inference latency

~13–14 FPS combined pipeline

Architecture

Edge AI + Cloud Backend + Cross-Platform Mobile Application

🔮 Future Work

Future improvements include:

Multi-person tracking using DeepSORT or ByteTrack
Audio-visual fall validation
Improved nighttime monitoring using IR cameras
Blood and injury detection
Context-aware alert prioritization
Improved occlusion handling
Real-time video communication
Text and voice communication
Enhanced reliability and robustness
Advanced remote monitoring capabilities
⚠️ Current Limitations

The current prototype has several limitations:

The system primarily assumes a single monitored person.
Severe occlusion can affect pose landmark detection.
Very low-light environments may require IR-assisted cameras.
Some rapid non-fall movements may produce false alerts.
Remote notification still depends on network availability.
Multi-person tracking is not currently implemented.
🎓 Academic Project

SafeHaven was developed as a Senior Graduation Project at:

Nile University
School of Information Technology and Computer Science
Computer Science Program

👥 Team

SafeHaven Development Team

Shaden Mohamed Abdelsalam
Merna Khaled Mahmoud
Nouran Adel Ahmed
Amr Tarek Ismail
Gamal Mohamed
Omar Ashraf

Supervisor: Dr. Ahmed Fathy Alnokrashy

📄 Documentation

The complete technical documentation contains detailed information about:

Problem definition
Literature review
System requirements
System architecture
Use case design
Database design
AI methodology
Dataset preparation
Model development
Experimental evaluation
Performance analysis
Limitations
Future work

Please refer to the project documentation included in this repository for the complete technical details.

⚠️ Disclaimer

SafeHaven is an academic research and engineering prototype.

It is not intended to replace professional medical monitoring, emergency services, or clinical safety systems.

⭐ Project Highlights

SafeHaven combines Edge AI, Computer Vision, Deep Learning, IoT, Cloud Computing, and Mobile Development into a privacy-preserving real-time safety monitoring ecosystem.

Built with ❤️ as a Senior Graduation Project at Nile University.
