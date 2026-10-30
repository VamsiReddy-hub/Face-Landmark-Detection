# 🧠 Face Landmark Detection & Drowsiness Monitoring

A real-time **face landmark detection** and **drowsiness monitoring** system built with [MediaPipe](https://mediapipe.dev/), [OpenCV](https://opencv.org/), and [Flask](https://flask.palletsprojects.com/). The application detects **468 facial landmarks** and uses them to compute physiological signals for driver/user alertness monitoring — supporting both **static image uploads** and **live webcam streaming**.

---

## ✨ Features

- 🎯 **468-point facial landmark detection** using MediaPipe FaceLandmarker
- 👁️ **Eye Aspect Ratio (EAR)** — detects closed/drowsy eyes
- 👄 **Mouth Aspect Ratio (MAR)** — detects yawning
- 📐 **Head Pitch Ratio** — detects head nodding / drooping
- 📸 **Static image upload** mode with annotated output
- 🎥 **Live webcam streaming** with real-time MJPEG feed
- 📊 **Live metrics dashboard** updated in real time
- 💾 **Event logging** — saves snapshots when drowsiness is detected
- 🌐 **Web UI** accessible from any browser

---

## 🏗️ Project Structure

```
Face_Landmark_Detection/
├── app.py                     # Flask application entry point
├── requirements.txt           # Python dependencies
│
├── src/
│   ├── landmark_detection.py  # MediaPipe FaceLandmarker wrapper
│   ├── drowsiness_detection.py# EAR / MAR / Head Pitch logic
│   ├── event_logger.py        # Drowsiness event snapshot logger
│   └── utils.py               # Image I/O, validation, logging helpers
│
├── data/
│   ├── models/
│   │   └── face_landmarker.task  # MediaPipe model file (required)
│   └── input_images/          # Uploaded images (auto-created)
│
├── outputs/
│   ├── images/                # Annotated output images (auto-created)
│   └── events/                # Drowsiness event snapshots (auto-created)
│
├── templates/
│   └── index.html             # Main web UI
│
├── static/                    # CSS, JS, and frontend assets
├── logs/                      # Application logs
└── .vscode/
    ├── launch.json            # VS Code debug configuration
    └── settings.json          # VS Code Python interpreter settings
```

---

## ⚙️ How It Works

### Landmark Detection
The `FaceLandmarkDetector` class loads the **MediaPipe FaceLandmarker** model and detects up to 468 facial landmarks from a single face in an image or video frame. Landmarks are converted from normalized coordinates to pixel coordinates.

### Drowsiness Detection
The `DrowsinessDetector` class computes three metrics per frame:

| Metric | Formula | Trigger Threshold |
|--------|---------|------------------|
| **EAR** (Eye Aspect Ratio) | `(A + B) / (2 × C)` using 6 eye landmarks | `< 0.22` for 15+ frames |
| **MAR** (Mouth Aspect Ratio) | `vertical / horizontal` using 4 lip landmarks | `> 0.60` for 7+ frames |
| **Head Pitch** | `forehead-to-nose / nose-to-chin` distance ratio | `> 1.15` for 15+ frames |

A **DROWSY** alert is triggered when any condition persists beyond its consecutive-frame threshold.

---

## 🚀 Getting Started

### Prerequisites

- Python 3.9 or higher
- A webcam (for live monitoring mode)
- MediaPipe model file (`face_landmarker.task`)

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd Face_Landmark_Detection
```

### 2. Create a Virtual Environment

```bash
python -m venv .venv
```

Activate it:
- **Windows**: `.venv\Scripts\activate`
- **macOS/Linux**: `source .venv/bin/activate`

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Download the MediaPipe Model

Download the `face_landmarker.task` model from the [MediaPipe Models page](https://developers.google.com/mediapipe/solutions/vision/face_landmarker#models) and place it at:

```
data/models/face_landmarker.task
```

### 5. Run the Application

```bash
python app.py
```

Open your browser and navigate to: **[http://127.0.0.1:5000](http://127.0.0.1:5000)**

---

## 🖥️ Running in VS Code

A `.vscode/` configuration is included for one-click debugging:

1. Open the `Face_Landmark_Detection` folder in VS Code.
2. Press **`F5`** (or go to **Run & Debug** → **Python: Flask App**).
3. The app will launch at [http://127.0.0.1:5000](http://127.0.0.1:5000).

VS Code will automatically use the `.venv` Python interpreter.

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Serves the main web UI |
| `POST` | `/upload` | Upload an image for face & drowsiness analysis |
| `POST` | `/webcam/start` | Start the webcam capture |
| `POST` | `/webcam/stop` | Stop the webcam capture |
| `GET` | `/webcam/frame` | MJPEG stream of the annotated webcam feed |
| `GET` | `/webcam/stats` | Latest real-time drowsiness metrics (JSON) |
| `GET` | `/outputs/<filename>` | Retrieve a processed output image |
| `GET` | `/health` | Server health check |

### Example: Upload Image

```bash
curl -X POST http://127.0.0.1:5000/upload \
  -F "image=@your_photo.jpg"
```

**Response:**
```json
{
  "status": "success",
  "faces_detected": 1,
  "landmarks_count": 468,
  "ear": 0.31,
  "mar": 0.12,
  "head_pitch": 0.98,
  "drowsy": false,
  "yawning": false,
  "head_dropped": false,
  "drowsiness_message": "NORMAL",
  "processing_time": 0.12,
  "output_path": "/outputs/annotated_your_photo.jpg"
}
```

---

## 📦 Dependencies

| Package | Purpose |
|---------|---------|
| `flask` | Web framework & REST API |
| `opencv-python` | Image processing & webcam capture |
| `mediapipe >= 0.10.13` | Face landmark detection model |
| `numpy` | Numerical computations (EAR, MAR, distances) |

---

## 🔧 Configuration

Key thresholds can be adjusted in `src/drowsiness_detection.py`:

```python
DrowsinessDetector(
    ear_threshold=0.22,        # Eye closure threshold (lower = less sensitive)
    mar_threshold=0.60,        # Yawn threshold (higher = less sensitive)
    head_pitch_threshold=1.15, # Head droop threshold
    consecutive_frames=15      # Frames needed to trigger alert
)
```

Image upload size limit is set in `app.py`:
```python
app.config["MAX_CONTENT_LENGTH"] = 10 * 1024 * 1024  # 10MB
```

---

## 📝 Logging & Events

- **Application logs** are written to the `logs/` directory via Python's logging module.
- **Drowsiness events** (snapshots + metadata) are saved to `outputs/events/` whenever a drowsy state is detected.
  - In **webcam mode**, events are throttled to once every **2 seconds** to avoid flooding.
  - In **image upload mode**, a single event is logged per drowsy image.

---

## 🙏 Acknowledgements

- [MediaPipe](https://mediapipe.dev/) by Google — face landmark model
- [OpenCV](https://opencv.org/) — image and video processing
- [Flask](https://flask.palletsprojects.com/) — lightweight Python web framework
- EAR algorithm based on the paper: *"Real-Time Eye Blink Detection using Facial Landmarks"* by Soukupová & Čech (2016)
