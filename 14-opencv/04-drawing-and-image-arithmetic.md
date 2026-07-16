# 04 — Drawing & Image Arithmetic

Recap file 03's closing note — drawing shapes/text (essential for visualizing results throughout this whole folder) and combining images together, including the bitwise masking technique previewed in file 03.

## Drawing basic shapes

```python
import cv2
import numpy as np

img = np.zeros((400, 400, 3), dtype=np.uint8)      # a blank black canvas, recap file 01's array/dtype note

cv2.line(img, (50, 50), (350, 50), (255, 0, 0), thickness=3)          # blue line (BGR! recap file 01)
cv2.rectangle(img, (50, 100), (200, 250), (0, 255, 0), thickness=2)     # green rectangle outline
cv2.rectangle(img, (250, 100), (350, 250), (0, 0, 255), thickness=-1)     # -1 thickness = FILLED, red
cv2.circle(img, (200, 320), 50, (255, 255, 0), thickness=2)                 # cyan circle outline
```

Genuinely worth remembering: `thickness=-1` is the standard OpenCV convention for a FILLED shape rather than an outline — easy to forget and a real, common small confusion.

## Drawing text

```python
cv2.putText(img, "Hello OpenCV", (50, 380), cv2.FONT_HERSHEY_SIMPLEX,
            fontScale=1, color=(255, 255, 255), thickness=2)
```

Genuinely the standard tool for ANNOTATING images — labeling detected objects with class names/confidence scores, a technique used constantly from file 07 (contour labeling) through file 12 (deep learning detection output visualization).

## Drawing bounding boxes — the genuinely most-used drawing pattern in this whole folder

```python
def draw_bounding_box(img, x, y, w, h, label, color=(0, 255, 0)):
    cv2.rectangle(img, (x, y), (x + w, y + h), color, thickness=2)
    cv2.putText(img, label, (x, y - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.6, color, thickness=2)
    return img
```

Genuinely worth building this as a reusable helper early — this EXACT pattern (rectangle + label above it) is what every object detection visualization looks like, from file 10's classical detectors through file 12's deep-learning-based ones, and directly parallels how detection results get visualized in real production computer vision systems.

## Image arithmetic — adding, subtracting, blending images

```python
img1 = cv2.imread('photo1.jpg')
img2 = cv2.imread('photo2.jpg')      # assume same dimensions

added = cv2.add(img1, img2)      # genuinely NOT the same as img1 + img2 with plain NumPy!
```

**Genuinely the single most important gotcha in this whole file:**

```python
# Recap file 01's dtype note -- images are uint8, range 0-255
a = np.array([200], dtype=np.uint8)
b = np.array([100], dtype=np.uint8)

print(a + b)          # plain NumPy: WRAPS AROUND -- 300 overflows to 44! (300 - 256)
print(cv2.add(a, b))   # cv2.add: SATURATES/CLIPS at 255 -- the mathematically sensible result
```

Genuinely a real, dangerous bug source — plain NumPy addition on `uint8` arrays silently WRAPS AROUND on overflow (200+100=300 becomes 44, a completely wrong, unexpected value), while `cv2.add()` correctly CLIPS/saturates at 255. This directly connects to the general "know your dtypes" caution from the NumPy/SQL folders — always use `cv2.add()`/`cv2.subtract()` for image arithmetic, never plain `+`/`-` on raw uint8 image arrays.

## Blending two images — weighted addition

```python
blended = cv2.addWeighted(img1, 0.7, img2, 0.3, gamma=0)
# result = img1*0.7 + img2*0.3 + 0
```

Genuinely useful for creating overlay effects — watermarking, fade transitions between video frames (file 11), or overlaying a heatmap/mask visualization on top of an original image with partial transparency.

## Bitwise operations — recap file 03's masking preview, now fully explained

```python
# Genuinely essential for working with MASKS -- binary images where 255=keep, 0=ignore
mask = np.zeros(img1.shape[:2], dtype=np.uint8)
cv2.circle(mask, (200, 200), 100, 255, thickness=-1)      # a circular mask region

masked_result = cv2.bitwise_and(img1, img1, mask=mask)      # keeps ONLY the circular region, blacks out the rest

# Combining two masks
combined_mask = cv2.bitwise_or(mask1, mask2)      # union -- either mask's region
intersection_mask = cv2.bitwise_and(mask1, mask2)    # intersection -- only where BOTH masks overlap
inverted_mask = cv2.bitwise_not(mask)                  # flips 0s and 255s
```

Recap Math Foundations file 09's Boolean logic / DSA file 17's bitwise operators directly — genuinely the SAME AND/OR/NOT operations, just applied pixel-by-pixel across an entire image array instead of individual bits in a single integer. `bitwise_and` with a mask argument is genuinely the standard OpenCV idiom for "apply this shape/region as a stencil."

## A complete, realistic masking pipeline — combining files 03 and 04

```python
img = cv2.imread('photo.jpg')
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)      # recap file 03

lower_green = np.array([40, 40, 40])
upper_green = np.array([80, 255, 255])
mask = cv2.inRange(hsv, lower_green, upper_green)      # recap file 03's color mask

result = cv2.bitwise_and(img, img, mask=mask)      # recap this file's masking technique

# Draw a bounding box around the masked region's overall extent (preview of file 07's contours)
ys, xs = np.where(mask == 255)      # recap NumPy's np.where -- find all matching pixel coordinates
if len(xs) > 0:
    x_min, x_max, y_min, y_max = xs.min(), xs.max(), ys.min(), ys.max()
    draw_bounding_box(img, x_min, y_min, x_max - x_min, y_max - y_min, "Green Object")
```

Genuinely a realistic, complete mini-pipeline — combining color-based masking (file 03), NumPy-based coordinate extraction, and the bounding-box drawing helper from earlier in this file — a genuinely simplified but real version of a classical color-based object detector, before file 07's proper contour-based approach and file 10's more robust detection methods.

## Try this

```python
# Create a blank 400x400 canvas and draw: a filled blue circle, an outlined red
# rectangle, a green line connecting two points, and a text label using putText
# Then load two real photos of the SAME size, blend them together with
# addWeighted at several different weight ratios (0.9/0.1, 0.5/0.5, 0.1/0.9) and
# save each result to compare the visual effect
# Finally, deliberately add two images using plain NumPy + instead of cv2.add(),
# and find/create a region where overflow visibly causes a wrong-looking color
# artifact -- then fix it with cv2.add() and confirm the artifact disappears
```

Deliberately triggering the NumPy-overflow bug and then fixing it is genuinely the best way to make sure this specific gotcha becomes permanent, hands-on knowledge rather than something read once and forgotten.

## What's next
File 05 — Filtering, Blurring & Edge Detection, moving into CONVOLUTION-based operations — genuinely the classical, hand-crafted ancestor of what a CNN's learned filters (recap PyTorch file 10) do automatically.
