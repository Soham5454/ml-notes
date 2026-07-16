# 11 — Video Processing & Real-time Analysis

Recap file 10's closing note — extending every single-image technique from files 01-10 into working with SEQUENCES of frames: motion detection, object tracking, and real-time webcam processing.

## A video is genuinely just a sequence of images

```python
import cv2

cap = cv2.VideoCapture('video.mp4')      # or cv2.VideoCapture(0) for the default webcam

while cap.isOpened():
    ret, frame = cap.read()      # ret: whether a frame was successfully read; frame: the actual image (recap file 01!)
    if not ret:
        break

    cv2.imshow('Video', frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):      # recap file 01's waitKey note -- 1ms wait for real-time playback
        break

cap.release()
cv2.destroyAllWindows()
```

Genuinely the single most important realization for this whole file: EVERY technique from files 01-10 (thresholding, edge detection, contours, feature matching, object detection) applies DIRECTLY to `frame` inside this loop — a video is genuinely just "call the same image-processing function repeatedly, once per frame," nothing conceptually new required.

## Video properties and writing output

```python
fps = cap.get(cv2.CAP_PROP_FPS)
width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))

fourcc = cv2.VideoWriter_fourcc(*'mp4v')
out = cv2.VideoWriter('output.mp4', fourcc, fps, (width, height))

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break
    processed_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)      # any processing, e.g. recap file 01
    processed_frame = cv2.cvtColor(processed_frame, cv2.COLOR_GRAY2BGR)      # VideoWriter needs 3 channels
    out.write(processed_frame)

cap.release()
out.release()
```

Genuinely important gotcha worth flagging: `VideoWriter` expects frames with the SAME channel count it was configured for — converting to grayscale (dropping to 2D, recap file 01) and forgetting to convert back to a 3-channel format before writing is a real, common source of a silently broken/corrupted output video.

## Frame differencing — the simplest possible motion detection

```python
cap = cv2.VideoCapture(0)
ret, prev_frame = cap.read()
prev_gray = cv2.cvtColor(prev_frame, cv2.COLOR_BGR2GRAY)

while True:
    ret, frame = cap.read()
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    diff = cv2.absdiff(prev_gray, gray)      # recap file 04's arithmetic operations directly
    _, motion_mask = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)      # recap file 03

    cv2.imshow('Motion', motion_mask)
    prev_gray = gray

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

Genuinely elegant in its simplicity: motion is, definitionally, CHANGE between consecutive frames — subtracting one frame from the next (recap file 04's `cv2.add`/subtract saturation note — `absdiff` similarly handles this correctly) and thresholding the result (recap file 03) directly reveals which pixels CHANGED, which is precisely where motion occurred.

## Background subtraction — a genuinely more robust approach

```python
back_sub = cv2.createBackgroundSubtractorMOG2(detectShadows=True)

cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    fg_mask = back_sub.apply(frame)      # genuinely LEARNS and continuously UPDATES a background model

    cv2.imshow('Foreground Mask', fg_mask)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

Genuinely more sophisticated than simple frame differencing — MOG2 (Mixture of Gaussians) builds a STATISTICAL MODEL of the background (recap Math Foundations file 10's Gaussian distributions directly, and file 09's Gaussian Mixture Model mention) for EVERY pixel over time, learning what "normal, unchanging background" looks like at each location, and flagging pixels that deviate significantly from their learned model as foreground. Genuinely more robust to gradual lighting changes and repetitive background motion (like waving tree branches) than simple frame differencing, which would incorrectly flag those as motion every single frame.

## Object tracking — following a detected object across frames

```python
tracker = cv2.TrackerCSRT_create()      # genuinely one of several available tracker algorithms

cap = cv2.VideoCapture('video.mp4')
ret, frame = cap.read()

bbox = cv2.selectROI('Select Object', frame, False)      # manually draw a box around the object once
tracker.init(frame, bbox)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    success, bbox = tracker.update(frame)
    if success:
        x, y, w, h = [int(v) for v in bbox]
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)      # recap file 04's bounding box pattern
    cv2.imshow('Tracking', frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

Genuinely important distinction worth being precise about, connecting back to file 10's detection discussion: DETECTION finds an object FRESH in every single frame (recap file 10's Haar cascade running independently each time), while TRACKING is initialized ONCE with a starting position, then uses the object's visual appearance and motion patterns to FOLLOW it efficiently across subsequent frames WITHOUT needing to re-detect it from scratch every time — genuinely faster per-frame, and can track objects that a detector might not even recognize as a specific "class" (tracking works on ANY selected region, not just pretrained object types).

## Combining detection and tracking — the genuinely realistic real-world pattern

```python
# Genuinely common real-world pipeline:
# 1. Run a (possibly expensive) DETECTOR periodically (e.g. every 30 frames, or file 12's
#    deep-learning detector) to find objects and confirm/refresh tracking
# 2. Run a (cheap, fast) TRACKER on every frame IN BETWEEN detections, to maintain
#    smooth, real-time tracking without the cost of running full detection every frame
```

Genuinely the standard, practical architecture for real-time video analysis systems — balancing detection ACCURACY (periodic, more expensive) against tracking SPEED (continuous, cheap), directly analogous to the general efficiency-vs-accuracy engineering tradeoffs seen throughout this entire roadmap (recap the XGBoost bridge file's founding framework, echoed constantly since).

## Optical flow — a genuinely deeper technique for understanding motion

```python
# Lucas-Kanade optical flow -- tracks how SPECIFIC POINTS move between consecutive frames
prev_pts = cv2.goodFeaturesToTrack(prev_gray, maxCorners=100, qualityLevel=0.3, minDistance=7)      # recap file 08's corner detection philosophy

next_pts, status, error = cv2.calcOpticalFlowPyrLK(prev_gray, gray, prev_pts, None)
```

Genuinely connects directly back to file 08's feature-point philosophy — `goodFeaturesToTrack` finds CORNERS (recap Harris corner detection's eigenvalue-based logic), and optical flow then tracks how each of those specific points MOVES from one frame to the next, genuinely giving a dense understanding of MOTION VECTORS across the scene, useful for tasks like camera-shake stabilization or estimating an object's velocity/direction.

## Try this

```python
# Using your webcam (or a downloaded short video), implement simple motion detection
# using BOTH frame differencing and MOG2 background subtraction
# Compare their behavior when: (1) something genuinely moves through the scene,
# (2) the lighting gradually changes (e.g. a cloud passing, or dimming a light),
# (3) something repetitive but not truly "new motion" happens (like a fan spinning)
# Honestly note which method handles each scenario better, directly connecting to
# this file's discussion of MOG2's statistical robustness advantage
```

Testing both motion-detection approaches under genuinely different real conditions (not just simple, clean motion) is the clearest way to build honest intuition about when the simpler frame-differencing approach is sufficient versus when the more sophisticated background-subtraction model genuinely earns its extra complexity.

## What's next
File 12 — the final OpenCV → Deep Learning CV Bridge, pulling every classical technique from files 01-11 together and connecting them explicitly to the CNN-based methods (recap PyTorch file 10, and modern detectors like YOLO) that represent the current state of the art.
