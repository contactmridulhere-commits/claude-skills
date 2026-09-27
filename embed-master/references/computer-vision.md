# Computer Vision & OpenCV Reference

## Table of Contents
1. OpenCV Installation
2. Camera Capture (Pi, USB, ESP32-CAM)
3. Motion Detection
4. Object Tracking
5. Object Detection (DNN / YOLO)
6. Face & Pose Detection (MediaPipe)
7. Color/HSV Tracking
8. ArUco Marker Tracking
9. Optical Flow
10. Performance Optimization

---

## 1. OpenCV Installation

```bash
# Raspberry Pi (recommended — headless for servers, contrib for extras)
pip install opencv-contrib-python   # Full + ArUco, tracking, SIFT
# OR lightweight:
pip install opencv-python-headless  # No GUI, smaller

# Verify
python3 -c "import cv2; print(cv2.__version__)"
```

## 2. Camera Capture

### Raspberry Pi Camera (picamera2 + OpenCV)
```python
from picamera2 import Picamera2
import cv2

picam = Picamera2()
picam.configure(picam.create_preview_configuration(
    main={"size": (640, 480), "format": "RGB888"}
))
picam.start()

while True:
    frame = picam.capture_array()  # numpy array, RGB
    frame_bgr = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)  # OpenCV uses BGR
    cv2.imshow("Camera", frame_bgr)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
picam.stop()
```

### USB Webcam
```python
import cv2
cap = cv2.VideoCapture(0)  # 0 = first camera
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret: break
    cv2.imshow("Webcam", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
```

### ESP32-CAM (stream JPEG to Pi for processing)
```cpp
// ESP32-CAM: Stream MJPEG over HTTP
#include "esp_camera.h"
#include <WiFi.h>
#include <WebServer.h>

// AI-Thinker pin definitions
#define PWDN_GPIO   32
#define RESET_GPIO  -1
#define XCLK_GPIO    0
#define SIOD_GPIO   26
#define SIOC_GPIO   27
#define Y9_GPIO     35
#define Y8_GPIO     34
#define Y7_GPIO     39
#define Y6_GPIO     36
#define Y5_GPIO     21
#define Y4_GPIO     19
#define Y3_GPIO     18
#define Y2_GPIO      5
#define VSYNC_GPIO  25
#define HREF_GPIO   23
#define PCLK_GPIO   22

// Then capture and serve via HTTP stream
// Pi-side: cv2.VideoCapture("http://<esp32-ip>:81/stream")
```

```python
# Pi-side: Receive ESP32-CAM stream
cap = cv2.VideoCapture("http://192.168.1.100:81/stream")
```

## 3. Motion Detection

### Background Subtraction (MOG2) — Most Robust
```python
import cv2

cap = cv2.VideoCapture(0)
bg_sub = cv2.createBackgroundSubtractorMOG2(
    history=500, varThreshold=50, detectShadows=True
)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret: break

    # Apply background subtraction
    fg_mask = bg_sub.apply(frame)

    # Remove shadows (gray=127) and noise
    _, thresh = cv2.threshold(fg_mask, 200, 255, cv2.THRESH_BINARY)
    thresh = cv2.morphologyEx(thresh, cv2.MORPH_OPEN,
        cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5)))

    # Find contours of moving objects
    contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area > 1000:  # Filter small noise
            x, y, w, h = cv2.boundingRect(cnt)
            cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
            cv2.putText(frame, f"Motion ({area})", (x, y-10),
                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)

    cv2.imshow("Motion Detection", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

### Frame Differencing (Simpler, Faster)
```python
prev_gray = None
while cap.isOpened():
    ret, frame = cap.read()
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.GaussianBlur(gray, (21, 21), 0)
    if prev_gray is None:
        prev_gray = gray; continue
    diff = cv2.absdiff(prev_gray, gray)
    _, thresh = cv2.threshold(diff, 30, 255, cv2.THRESH_BINARY)
    prev_gray = gray
    # ... find contours on thresh
```

## 4. Object Tracking

### Single Object Tracking (OpenCV Trackers)
```python
import cv2

tracker_types = {
    'CSRT': cv2.TrackerCSRT_create,      # Accurate, ~15fps
    'KCF': cv2.TrackerKCF_create,        # Balanced, ~25fps
    'MOSSE': cv2.legacy.TrackerMOSSE_create,  # Fastest, ~50fps, less accurate
}

cap = cv2.VideoCapture(0)
ret, frame = cap.read()

# Select ROI (region of interest)
bbox = cv2.selectROI("Select Object", frame, fromCenter=False)
tracker = tracker_types['CSRT']()
tracker.init(frame, bbox)

while cap.isOpened():
    ret, frame = cap.read()
    success, bbox = tracker.update(frame)
    if success:
        x, y, w, h = [int(v) for v in bbox]
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
        # Center point
        cx, cy = x + w//2, y + h//2
        cv2.circle(frame, (cx, cy), 5, (0, 0, 255), -1)
    else:
        cv2.putText(frame, "Lost!", (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
    cv2.imshow("Tracking", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

### Multi-Object Tracking
```python
trackers = cv2.legacy.MultiTracker_create()
# Add multiple ROIs
for bbox in bboxes:
    trackers.add(cv2.TrackerCSRT_create(), frame, bbox)

# Update all
success, boxes = trackers.update(frame)
for box in boxes:
    x, y, w, h = [int(v) for v in box]
    cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
```

## 5. Object Detection (YOLOv5/v8 with TFLite)

### Using OpenCV DNN (works without extra frameworks)
```python
import cv2
import numpy as np

# Load ONNX model
net = cv2.dnn.readNetFromONNX("yolov5n.onnx")

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    blob = cv2.dnn.blobFromImage(frame, 1/255.0, (640, 640), swapRB=True, crop=False)
    net.setInput(blob)
    outputs = net.forward(net.getUnconnectedOutLayersNames())
    # Parse detections (boxes, confidences, class IDs)
    # Apply NMS (non-maximum suppression)
    # Draw bounding boxes
```

### Using TFLite (faster on Pi)
```python
from tflite_runtime.interpreter import Interpreter
import cv2, numpy as np

interpreter = Interpreter(model_path="yolov5n_int8.tflite")
interpreter.allocate_tensors()
interpreter.set_num_threads(4)

input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()
input_size = input_details[0]['shape'][1:3]  # e.g., (320, 320)

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    resized = cv2.resize(frame, tuple(input_size))
    input_data = np.expand_dims(resized, axis=0).astype(np.uint8)

    interpreter.set_tensor(input_details[0]['index'], input_data)
    interpreter.invoke()
    output = interpreter.get_tensor(output_details[0]['index'])
    # Parse and draw detections
```

## 6. Face & Pose Detection (MediaPipe)

```bash
pip install mediapipe
```

### Hand Tracking
```python
import cv2
import mediapipe as mp

mp_hands = mp.solutions.hands
mp_draw = mp.solutions.drawing_utils
hands = mp_hands.Hands(max_num_hands=2, min_detection_confidence=0.7)

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(rgb)

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_draw.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)
            # Access specific landmarks:
            # hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]

    cv2.imshow("Hands", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

### Pose Estimation
```python
mp_pose = mp.solutions.pose
pose = mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5)
# results = pose.process(rgb_frame)
# results.pose_landmarks — 33 body landmarks
```

## 7. Color/HSV Tracking

```python
import cv2, numpy as np

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    # Track red objects (red wraps around H=0 and H=180)
    lower_red1 = np.array([0, 120, 70])
    upper_red1 = np.array([10, 255, 255])
    lower_red2 = np.array([170, 120, 70])
    upper_red2 = np.array([180, 255, 255])

    mask = cv2.inRange(hsv, lower_red1, upper_red1) | cv2.inRange(hsv, lower_red2, upper_red2)
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, np.ones((5,5), np.uint8))

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    for cnt in contours:
        if cv2.contourArea(cnt) > 500:
            x, y, w, h = cv2.boundingRect(cnt)
            cv2.rectangle(frame, (x,y), (x+w,y+h), (0,255,0), 2)

    cv2.imshow("Color Track", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

### Common HSV Ranges
| Color | H Low | H High | S Low | V Low |
|-------|-------|--------|-------|-------|
| Red | 0-10 + 170-180 | (wraps) | 120 | 70 |
| Blue | 100 | 130 | 120 | 70 |
| Green | 40 | 80 | 60 | 60 |
| Yellow | 20 | 35 | 100 | 100 |
| Orange | 10 | 20 | 120 | 70 |

## 8. ArUco Marker Tracking

```python
import cv2

aruco_dict = cv2.aruco.getPredefinedDictionary(cv2.aruco.DICT_4X4_50)
detector_params = cv2.aruco.DetectorParameters()
detector = cv2.aruco.ArucoDetector(aruco_dict, detector_params)

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    corners, ids, rejected = detector.detectMarkers(frame)

    if ids is not None:
        cv2.aruco.drawDetectedMarkers(frame, corners, ids)
        for i, corner in enumerate(corners):
            center = corner[0].mean(axis=0).astype(int)
            cv2.putText(frame, f"ID:{ids[i][0]}", tuple(center),
                cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
    cv2.imshow("ArUco", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

## 9. Optical Flow

### Dense Optical Flow (Farneback)
```python
prev_gray = cv2.cvtColor(prev_frame, cv2.COLOR_BGR2GRAY)
while True:
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    flow = cv2.calcOpticalFlowFarneback(prev_gray, gray, None,
        pyr_scale=0.5, levels=3, winsize=15, iterations=3, poly_n=5, poly_sigma=1.2, flags=0)
    magnitude, angle = cv2.cartToPolar(flow[..., 0], flow[..., 1])
    # Visualize: map angle to hue, magnitude to value
    prev_gray = gray
```

## 10. Performance Optimization Tips

1. **Resize early**: Process at 320×240 or 640×480, not full resolution
2. **Skip frames**: Process every 2nd or 3rd frame for non-critical tasks
3. **Use grayscale**: When color isn't needed (`cvtColor(frame, COLOR_BGR2GRAY)`)
4. **ROI processing**: Only process relevant region of frame
5. **Threading**: Capture in one thread, process in another
6. **INT8 models**: 2-4× faster than FP32 on Pi
7. **Pi 5**: Use `picamera2` with hardware-accelerated preview pipeline
8. **Headless**: Use `-headless` OpenCV build if no display needed (saves memory)

```python
# Threaded capture pattern
import threading

class VideoCapture:
    def __init__(self, src=0):
        self.cap = cv2.VideoCapture(src)
        self.ret, self.frame = self.cap.read()
        self.running = True
        threading.Thread(target=self._update, daemon=True).start()

    def _update(self):
        while self.running:
            self.ret, self.frame = self.cap.read()

    def read(self):
        return self.ret, self.frame.copy()
```
