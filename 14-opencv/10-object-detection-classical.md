# 10 — Object Detection (Classical)

Recap file 09's closing note — moving from segmenting regions into detecting and LOCALIZING specific, KNOWN object types. The classical, pre-deep-learning approach to a task now dominated by CNN-based methods (properly bridged in file 12).

## Haar Cascades — the genuinely classic, historically important technique

```python
import cv2

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

img = cv2.imread('photo.jpg')
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)      # recap file 04's bounding box pattern
```

Genuinely the technique behind early digital camera "face detection" boxes, and worth understanding conceptually even though modern deep-learning detectors have largely superseded it for serious applications.

## How Haar cascades actually work — genuinely worth understanding, not just using

```python
# Haar features are simple rectangular patterns (recap file 05's convolution kernel idea,
# but much simpler) that compute the DIFFERENCE in average brightness between adjacent
# rectangular regions -- e.g. "the eye region is genuinely DARKER than the cheek region below it"

# A "cascade" is a SEQUENCE of increasingly complex Haar-feature checks (recap DSA file 02's
# backtracking "prune early" philosophy) -- most candidate windows get REJECTED
# after just the first few, cheap checks, so the expensive full check only runs on
# genuinely promising candidates. This CASCADE structure is precisely what makes
# it fast enough for real-time detection on hardware from the early 2000s
```

Genuinely a beautiful, efficiency-focused design worth appreciating on its own — most of the image is CLEARLY not a face, and the cascade structure lets those regions get discarded almost instantly, spending real computation only where it might actually matter. Directly analogous to DSA file 14's greedy "eliminate clearly-bad options early" philosophy, applied here to detection windows instead of algorithmic choices.

## `detectMultiScale` parameters — genuinely important to tune correctly

```python
faces = face_cascade.detectMultiScale(
    gray,
    scaleFactor=1.1,      # how much the image size is REDUCED at each scale step
    minNeighbors=5,          # how many overlapping detections are needed to confirm a real match
    minSize=(30, 30)           # smallest object size to even consider
)
```

`scaleFactor` genuinely matters because Haar cascades are trained on a FIXED window size — to detect faces of different SIZES in the image (someone close to the camera vs far away), the algorithm repeatedly SHRINKS the image and re-scans, effectively searching at multiple scales. `minNeighbors` is a genuinely important noise-reduction parameter — a real face typically gets detected multiple times at slightly different positions/scales; requiring several overlapping detections to agree filters out spurious single false-positives, conceptually similar to file 08's Lowe's ratio test philosophy of requiring stronger evidence before accepting a match.

## Multiple cascade types — genuinely available for different objects

```python
eye_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_eye.xml')
smile_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_smile.xml')

# Genuinely a common, practical pattern: detect faces FIRST, then search for eyes
# ONLY within each detected face region -- recap file 01's ROI slicing directly
for (x, y, w, h) in faces:
    face_roi_gray = gray[y:y+h, x:x+w]
    eyes = eye_cascade.detectMultiScale(face_roi_gray)
```

Genuinely a practical, important pattern — detecting SUB-features WITHIN an already-detected region is both faster (smaller search area) and more RELIABLE (fewer false positives from unrelated image regions) than searching the whole image for eyes independently.

## HOG + SVM — the other genuinely classic pedestrian/object detector

```python
hog = cv2.HOGDescriptor()
hog.setSVMDetector(cv2.HOGDescriptor_getDefaultPeopleDetector())

boxes, weights = hog.detectMultiScale(img, winStride=(8,8))

for (x, y, w, h) in boxes:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 0, 255), 2)
```

**HOG (Histogram of Oriented Gradients)** — genuinely worth understanding the feature extraction idea, connecting directly to file 05's Sobel gradient discussion: HOG divides an image region into small cells, computes the GRADIENT direction (recap file 05's Sobel operator) at every pixel within each cell, and builds a HISTOGRAM (recap the matplotlib/Seaborn histogram notes) of gradient DIRECTIONS per cell. This captures the overall SHAPE/STRUCTURE of an object (e.g. a person's characteristic silhouette) in a way that's genuinely robust to small lighting/color variations, since it only cares about gradient DIRECTIONS, not raw pixel intensities.

**SVM (Support Vector Machine)** — recap scikit-learn's classification algorithms directly: once HOG extracts a feature VECTOR describing a candidate window, an SVM classifier (a genuinely standard scikit-learn algorithm) decides "person" or "not person" based on that feature vector — precisely the same classical ML classification pattern used throughout the scikit-learn folder, just with HOG features instead of tabular data as input.

## Template matching — the genuinely simplest possible detection technique

```python
template = cv2.imread('logo_template.jpg', 0)
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

result = cv2.matchTemplate(img_gray, template, cv2.TM_CCOEFF_NORMED)
min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)

if max_val > 0.8:      # genuinely a real, tunable confidence threshold
    h, w = template.shape
    top_left = max_loc
    bottom_right = (top_left[0] + w, top_left[1] + h)
    cv2.rectangle(img, top_left, bottom_right, (0, 255, 0), 2)
```

Genuinely worth knowing this exists as the SIMPLEST possible detection approach — literally slide the exact template image across the target image and compute a similarity score (via cross-correlation, genuinely a variant of the dot-product-based similarity from Math Foundations file 01) at every position. Honestly, genuinely LIMITED: it only works for finding EXACT or near-exact copies of the template (same scale, same rotation, same appearance) — completely fails if the object appears at a different size/angle/lighting than the template, unlike the more robust feature-matching approach from file 08.

## The honest, complete comparison of classical detection techniques

| Technique | Genuinely good for | Genuinely limited by |
|---|---|---|
| Haar Cascades | Fast, specific pretrained objects (faces) | Needs extensive pretraining per object type, sensitive to angle/lighting |
| HOG + SVM | Pedestrian/rigid-object detection | Slower than Haar, still needs a trained classifier per object type |
| Template Matching | Exact, known logos/icons at fixed scale | Fails on scale/rotation/appearance changes |
| Feature Matching (file 08) | Known objects under varying angle/lighting | Needs a reference image, struggles with genuinely NEW/unseen object categories |

Genuinely the honest, unifying limitation across ALL classical detection techniques in this file: every one of them requires EITHER a specifically pretrained classifier (Haar, HOG+SVM) OR an exact reference image/template (template matching, feature matching) — NONE of them can genuinely generalize to detect a brand-new OBJECT CATEGORY they've never explicitly been prepared for. This is PRECISELY the gap that CNN-based object detection (YOLO, Faster R-CNN — covered in file 12) was built to close, learning general, transferable visual features (recap PyTorch file 10's CNN feature-learning discussion) rather than relying on hand-crafted rules or exact templates.

## Try this

```python
# Using the built-in Haar face cascade, detect faces in a photo containing at least
# one person, then WITHIN each detected face region, detect eyes using the eye cascade
# (recap the ROI-based sub-detection pattern from this file)
# Compare detectMultiScale's results using different minNeighbors values (try 2, 5, 10)
# on the SAME photo -- observe how lower values produce more false positives while
# higher values might miss genuine faces, directly demonstrating the precision/recall
# tradeoff (recap scikit-learn's exact framing) in a classical detection context
```

Directly tuning `minNeighbors` and observing the precision/recall tradeoff play out visually (more boxes vs fewer, but more/less accurate) is genuinely a good concrete bridge back to the classification metrics already deeply covered in the scikit-learn folder, now seen in a detection context.

## What's next
File 11 — Video Processing & Real-time Analysis, extending every single-image technique covered so far (files 01-10) into working with SEQUENCES of frames — motion detection, object tracking, and real-time webcam processing.
