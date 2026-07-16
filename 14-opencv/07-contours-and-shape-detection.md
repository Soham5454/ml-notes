# 07 — Contours & Shape Detection

Recap file 06's closing note — applying the cleaned-up binary image pipeline to an actual TASK: identifying and analyzing distinct objects as CONTOURS, genuinely the classical technique for "find and describe the shapes in this image."

## What a contour actually is

A contour is genuinely just a curve joining all the continuous points along a boundary that share the same color/intensity — in practice, the OUTLINE of a connected white region in a binary image (recap files 03 and 06's thresholding/morphology pipeline).

```python
import cv2
import numpy as np

img = cv2.imread('shapes.jpg')
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
_, binary = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)      # recap file 03

contours, hierarchy = cv2.findContours(binary, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
print(f"Found {len(contours)} contours")
```

Genuinely important prerequisite worth restating: `findContours` works on a BINARY image — this is precisely WHY files 03 (thresholding) and 06 (morphological cleanup) mattered so much as preprocessing before this file could work reliably.

## Retrieval modes and approximation methods — genuinely important parameters

```python
# Retrieval modes -- WHICH contours to return
cv2.RETR_EXTERNAL      # only the OUTERMOST contours (ignores holes/nested shapes)
cv2.RETR_LIST            # ALL contours, no hierarchy information
cv2.RETR_TREE              # ALL contours, WITH full parent/child hierarchy info

# Approximation methods -- HOW MANY points to store per contour
cv2.CHAIN_APPROX_NONE       # stores EVERY single boundary point -- genuinely wasteful
cv2.CHAIN_APPROX_SIMPLE       # compresses straight-line segments to just their ENDPOINTS
```

Genuinely worth recognizing `CHAIN_APPROX_SIMPLE`'s compression idea connects to Math Foundations file 03's rank/redundancy discussion — a straight line's intermediate points are genuinely REDUNDANT information (fully determined by the two endpoints), so storing only endpoints is a lossless, efficient compression, not an approximation that loses real information.

## Drawing contours — recap file 04's drawing tools directly

```python
img_with_contours = img.copy()
cv2.drawContours(img_with_contours, contours, -1, (0, 255, 0), thickness=2)
# -1 for the contour INDEX means draw ALL of them; a specific index draws just ONE
```

## Contour properties — extracting meaningful measurements

```python
for contour in contours:
    area = cv2.contourArea(contour)          # genuinely just the enclosed region's pixel area
    perimeter = cv2.arcLength(contour, closed=True)

    x, y, w, h = cv2.boundingRect(contour)      # recap file 04's bounding box drawing helper directly
    cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)

    print(f"Area: {area}, Perimeter: {perimeter}")
```

Genuinely worth filtering by area as a real, practical noise-removal step at this stage too — even after file 06's morphological cleanup, tiny leftover contours sometimes remain, and `if area > 100: ...` is a genuinely common, simple additional filter.

## Shape approximation — simplifying a contour into a polygon

```python
for contour in contours:
    epsilon = 0.02 * cv2.arcLength(contour, True)
    approx = cv2.approxPolyDP(contour, epsilon, True)
    print(f"Number of vertices: {len(approx)}")
```

Genuinely the key technique for CLASSIFYING shapes — `approxPolyDP` simplifies a noisy, many-point contour down to its essential POLYGON VERTICES (using the Douglas-Peucker algorithm, which recursively keeps only points that deviate significantly from a straight-line approximation — conceptually related to DSA's divide-and-conquer recursion pattern, recap that folder's merge sort). The resulting VERTEX COUNT directly reveals the shape:

```python
def classify_shape(approx):
    vertices = len(approx)
    if vertices == 3:
        return "Triangle"
    elif vertices == 4:
        x, y, w, h = cv2.boundingRect(approx)
        aspect_ratio = w / float(h)
        return "Square" if 0.95 <= aspect_ratio <= 1.05 else "Rectangle"
    elif vertices > 8:
        return "Circle"          # genuinely: a circle approximates as MANY small polygon edges
    else:
        return f"Polygon ({vertices} sides)"
```

Genuinely a real, complete classical shape-classification technique — no machine learning needed at all, just geometric reasoning about vertex counts and aspect ratios, directly usable for tasks like sorting geometric shapes or detecting simple icons/symbols.

## Contour hierarchy — understanding nested shapes

```python
contours, hierarchy = cv2.findContours(binary, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
print(hierarchy)
# Each entry: [Next, Previous, First_Child, Parent] -- genuinely a TREE structure!
```

Recap DSA file 07's tree structure directly — `RETR_TREE` genuinely builds a REAL tree (parent/child relationships between contours), exactly the same conceptual structure as that folder's binary trees, just representing "this shape contains that shape" (e.g. a donut shape's outer boundary is the PARENT of its inner hole's boundary) rather than a general binary tree's left/right children.

## Convex hull — the "smallest enclosing convex shape"

```python
for contour in contours:
    hull = cv2.convexHull(contour)
    cv2.drawContours(img, [hull], -1, (0, 0, 255), 2)
```

Genuinely useful for hand-gesture recognition and similar tasks — the convex hull "wraps" a shape tightly, and comparing the ORIGINAL contour against its convex hull reveals CONCAVITY DEFECTS (points where the real shape "caves inward" relative to the hull) — a real, classical technique for counting fingers in hand-tracking applications, for instance.

## Moments and centroid — finding a shape's "center of mass"

```python
for contour in contours:
    M = cv2.moments(contour)
    if M["m00"] != 0:
        cx = int(M["m10"] / M["m00"])
        cy = int(M["m01"] / M["m00"])
        cv2.circle(img, (cx, cy), 5, (255, 0, 255), -1)
```

Genuinely connects to Math Foundations file 12's expectation/mean formula — image moments are conceptually a weighted average of pixel COORDINATES (weighted by whether they're part of the shape), and the centroid formula here is genuinely the exact same "sum of values divided by count" mean calculation, just applied to (x, y) pixel positions instead of a plain list of numbers.

## A complete shape-detection pipeline

```python
img = cv2.imread('shapes.jpg')
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5,5), 0)      # recap file 05
_, binary = cv2.threshold(blurred, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)      # recap file 03
kernel = np.ones((3,3), np.uint8)
cleaned = cv2.morphologyEx(binary, cv2.MORPH_OPEN, kernel)      # recap file 06

contours, _ = cv2.findContours(cleaned, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

for contour in contours:
    if cv2.contourArea(contour) < 100:      # filter out tiny noise contours
        continue
    epsilon = 0.02 * cv2.arcLength(contour, True)
    approx = cv2.approxPolyDP(contour, epsilon, True)
    shape_name = classify_shape(approx)
    x, y, w, h = cv2.boundingRect(contour)
    cv2.drawContours(img, [contour], -1, (0, 255, 0), 2)
    cv2.putText(img, shape_name, (x, y-10), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0,255,0), 2)      # recap file 04
```

Genuinely the realistic, complete shape this whole folder has been building toward — files 03, 05, 06, and 07 all combining into one working classical shape-detection system, no deep learning required.

## Try this

```python
# Draw several shapes on a blank canvas using cv2's shape-drawing functions
# (recap file 04): 2 triangles, 2 rectangles/squares of different aspect ratios,
# and 2 circles, with some genuine variation in size and rotation
# Run the complete pipeline above and confirm classify_shape() correctly labels
# every shape
# Then deliberately draw two overlapping/touching shapes and observe what
# findContours does -- does it correctly separate them, or does it merge them
# into one contour? This limitation is genuinely worth knowing honestly
```

Deliberately testing overlapping shapes and observing the honest limitation firsthand is genuinely valuable — contour-based detection assumes objects are reasonably SEPARATED, and this exercise makes that assumption's real, practical failure mode concrete rather than abstract.

## What's next
File 08 — Feature Detection & Matching, moving beyond simple shapes into identifying and matching DISTINCTIVE POINTS across different images — genuinely essential for tasks like image stitching, object recognition, and tracking.
