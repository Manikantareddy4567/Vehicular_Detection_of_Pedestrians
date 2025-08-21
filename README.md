# Vehicular Prediction of Pedestrians

A Flask-based real‑time application that uses **Ultralytics YOLO** and **OpenCV** to detect pedestrians and vehicles from a webcam or uploaded media, estimate proximity, track motion via optical flow, and trigger audible alerts when pedestrians are near.

> **Tech stack:** Python, Flask, OpenCV, Ultralytics YOLO, NumPy. Windows beep alerts via `winsound`.

---

## Table of Contents

* [Features](#features)
* [Project Media (screenshots/)](#project-media-screenshots)
* [Folder Structure](#folder-structure)
* [Setup](#setup)
* [Running the App](#running-the-app)
* [API Endpoints](#api-endpoints)
* [Configuration](#configuration)
* [Usage Tips](#usage-tips)
* [Troubleshooting](#troubleshooting)
* [License](#license)

---

## Features

* Real‑time detection with **YOLO** (`yolov9c.pt` by default).
* Tracks motion using **Lucas–Kanade optical flow** and overlays speed & direction.
* Estimates pedestrian distance (configurable constants) and triggers a **beep alert** when too close.
* Works with **webcam**, **video files**, and **photos**.
* Simple JSON **stats** endpoint for integration/dashboards.

---

## Project Media (screenshots/)

This project includes a top-level `screenshots/` folder with supporting media:

```
screenshots/
├─ screenshot/   # Sample screenshots of the application UI/overlays
├─ video/        # Demo videos showing detections and tracking in action
└─ details/      # Extra images or annotated frames explaining specific features
```

These assets help visualize functionality, expected outputs, and example scenarios. If you use Git, consider excluding large media in `.gitignore` or storing videos in releases/cloud storage.

---

## Folder Structure

```
project-root/
├─ app.py                # Flask application (server + detection logic)
├─ templates/
│  └─ index.html         # Frontend page to view streams
├─ uploads/              # Saved uploads (created at runtime)
├─ screenshots/          # Project media (see section above)
│  ├─ screenshot/
│  ├─ video/
│  └─ details/
├─ yolov9c.pt            # YOLO model weights (provide your own file)
├─ requirements.txt      # Python dependencies
└─ README.md             # This file
```

---

## Setup

1. **Create & activate a virtual environment** (Windows PowerShell):

   ```powershell
   python -m venv venv
   venv\Scripts\activate
   ```

2. **Install dependencies**:

   ```powershell
   pip install --upgrade pip setuptools wheel
   pip install -r requirements.txt
   ```

3. **Model weights**: Place your YOLO weights file at `./yolov9c.pt` (or update the path in `app.py`).

> **Note (Windows):** Audible alerts use `winsound.Beep`. On non‑Windows systems, replace with a cross‑platform alternative (e.g., `playsound`, `pyaudio`, or remove alerts).



## Running the App

From the project root with the virtual environment activated:

```powershell
python app.py
```

The server starts in debug mode at `http://127.0.0.1:5000/`.

* Open your browser to `/` to view the UI.
* Use the **Toggle Camera** control to start/stop the webcam feed (`/video_feed`).

---

## API Endpoints

### `GET /`

Renders the main UI (`templates/index.html`).

### `GET /video_feed`

Returns an MJPEG stream from the active webcam. Returns **204** if the camera is off.

### `POST /toggle_camera`

Toggles the webcam capture.

* **Response**: `"Camera turned on"` or `"Camera turned off"`.

### `POST /upload`

Uploads a file (any media) and stores it in `uploads/`.

* **Form field**: `file`
* **Response**: Saved filename.

### `POST /upload_video_feed`

Streams detection results for an uploaded video file as MJPEG.

* **Form field**: `file`
* **Response**: MJPEG multipart stream.

### `POST /upload_photo`

Runs detection on a single photo and returns the annotated image (`image/jpeg`).

* **Form field**: `file`
* **Response**: JPEG image with boxes/labels.

### `GET /stats`

Returns cumulative detection stats since server start.

* **Response (JSON)**:

  ```json
  {
    "total_detections": 0,
    "vehicles_detected": 0,
    "pedestrians_detected": 0
  }
  ```

---

## Configuration

Key constants are defined in `app.py`:

* **Model path**: `model = YOLO('./yolov9c.pt')`
* **Classes**:

  * `HUMAN_CLASS = 0`
  * `VEHICLE_CLASSES = [1, 2, 3, 5, 7]`
* **Distance estimation**:

  * `KNOWN_WIDTH = 0.5` (meters; approx. shoulder width)
  * `FOCAL_LENGTH = 700` (pixels; calibrate for your camera)
* **Optical flow (Lucas–Kanade)**: `lk_params` dict.

> Tune `KNOWN_WIDTH`, `FOCAL_LENGTH`, and the threshold in `is_near(distance, threshold=4.0)` to match your camera and safety criteria.

---

## Usage Tips

* For best results, use well‑lit scenes and a stable camera.
* Ensure `yolov9c.pt` matches the classes you expect (COCO‑like indexing is assumed in this app).
* If performance is low, consider:

  * Smaller input frames (resize before inference),
  * A lighter YOLO model variant,
  * Processing every Nth frame (already supported via `frame_count`).

---

## Troubleshooting

* **Matplotlib build error on Windows (Meson/Ninja/GCC)**: upgrade `pip`, use prebuilt wheels or pin `matplotlib==3.8.4` if needed.
* **Camera not opening**: another app may be using the webcam; check device permissions.
* **No model file**: make sure `yolov9c.pt` exists at the configured path.
* **High CPU usage**: reduce frame size or inference frequency.

---

