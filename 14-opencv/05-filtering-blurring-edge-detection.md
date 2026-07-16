# 05 — Filtering, Blurring & Edge Detection

Recap file 04's closing note and PyTorch file 10's CNN convolution introduction directly — this file covers CONVOLUTION-based operations, genuinely the classical, HAND-CRAFTED ancestor of what a CNN's LEARNED filters do automatically.

## Convolution, recapped from PyTorch file 10, now with hand-designed filters

```python
import cv2
import numpy as np

# Recap PyTorch file 10's Conv2d explanation directly:
# a convolution slides a small filter/kernel across the image, computing a weighted
# sum at each position. In a CNN, the filter WEIGHTS are LEARNED during training.
# In classical image processing, the filter weights are HAND-DESIGNED by humans
# for a SPECIFIC known purpose (blurring, sharpening, edge detection)

kernel = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]])      # a hand-designed "sharpen" kernel
sharpened = cv2.filter2D(img, -1, kernel)
```

Genuinely worth pausing on this connection — this whole file is essentially "what filters looked like before CNNs learned them automatically." Understanding these hand-crafted filters deeply makes it much more intuitive WHY a CNN's learned filters work the way they do, and what kinds of patterns (edges, blobs, textures) early CNN layers tend to discover on their own.

## Blurring — averaging out pixel values

```python
blurred_avg = cv2.blur(img, (5, 5))      # simple average of each 5x5 neighborhood

gaussian_blurred = cv2.GaussianBlur(img, (5, 5), sigmaX=0)      # weighted average, center pixels matter more
```

Recap Math Foundations file 10's Normal distribution directly — Gaussian blur genuinely uses a 2D Gaussian (bell curve, recap that file's formula) as the kernel's weighting pattern, giving pixels CLOSER to the center of the neighborhood more influence than pixels farther away. This produces a genuinely smoother, more natural-looking blur than simple averaging, and is the standard choice for noise reduction before edge detection (covered later in this file).

```python
median_blurred = cv2.medianBlur(img, 5)      # takes the MEDIAN value in each neighborhood, not the mean
```

Recap Math Foundations file 13's mean-vs-median outlier-robustness discussion directly — median blur is genuinely EXCELLENT at removing "salt and pepper" noise (random isolated bright/dark pixels) specifically BECAUSE the median (unlike the mean) is robust to those extreme outlier values, exactly the same statistical property discussed in that earlier file, now applied to image denoising.

```python
bilateral = cv2.bilateralFilter(img, d=9, sigmaColor=75, sigmaSpace=75)
```

Genuinely a more sophisticated option — bilateral filtering blurs SMOOTH regions while PRESERVING EDGES (unlike Gaussian blur, which blurs edges too) by additionally weighting neighboring pixels by how SIMILAR their color/intensity is, not just their spatial distance. Slower than Gaussian blur, but genuinely valuable when edge preservation matters (e.g. as a preprocessing step before edge detection, ironically to reduce noise WITHOUT destroying the real edges being searched for).

## Sharpening — the opposite operation

```python
sharpen_kernel = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]])
sharpened = cv2.filter2D(img, -1, sharpen_kernel)
```

Genuinely worth understanding the kernel's structure intuitively: the CENTER weight (5) amplifies the current pixel, while the surrounding NEGATIVE weights subtract out the neighborhood average — the net effect emphasizes DIFFERENCES between a pixel and its surroundings, which is precisely what makes edges/details appear more pronounced.

## Edge detection — Sobel operator, recap Math Foundations' derivatives directly

```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

sobel_x = cv2.Sobel(gray, cv2.CV_64F, dx=1, dy=0, ksize=3)      # detects VERTICAL edges (horizontal gradient)
sobel_y = cv2.Sobel(gray, cv2.CV_64F, dx=0, dy=1, ksize=3)        # detects HORIZONTAL edges (vertical gradient)

sobel_combined = cv2.magnitude(sobel_x, sobel_y)      # recap Math Foundations file 01's vector magnitude!
```

Genuinely one of the most satisfying cross-folder connections in this whole repo: an edge is, mathematically, a region where pixel intensity CHANGES RAPIDLY — precisely a large DERIVATIVE (recap Math Foundations file 05's derivative definition directly: "the derivative measures how much a function's output changes when its input changes"). The Sobel operator approximates this derivative in the x and y directions using a small convolution kernel, and `cv2.magnitude` combines the two directional gradients using the EXACT vector magnitude formula from Math Foundations file 01 (`sqrt(x² + y²)`) — genuinely the same math, now computing "how strong is the edge at this pixel" instead of "how long is this vector."

## Canny edge detection — the genuinely standard, more complete algorithm

```python
edges = cv2.Canny(gray, threshold1=100, threshold2=200)
```

Genuinely worth knowing Canny is a MULTI-STEP algorithm built on top of the Sobel gradient idea just covered: (1) Gaussian blur to reduce noise first (recap this file's blurring section), (2) compute gradients via Sobel-like operators, (3) "non-maximum suppression" — thin out edges to single-pixel width by keeping only local gradient-magnitude maxima, (4) "hysteresis thresholding" — the two threshold parameters: pixels above `threshold2` are definitely edges, pixels below `threshold1` are definitely not, and pixels in between are only kept if CONNECTED to a definite edge. Genuinely the standard, most commonly used edge detector in practice — the two-threshold hysteresis approach makes it more robust than a single Sobel-magnitude cutoff.

```python
# Threshold tuning matters genuinely a lot -- too low picks up noise as false edges,
# too high misses genuine but subtle edges
edges_sensitive = cv2.Canny(gray, 30, 100)      # more edges detected, more noise too
edges_strict = cv2.Canny(gray, 150, 250)          # fewer, more confident edges only
```

## Laplacian — a genuinely different edge-detection approach, using the SECOND derivative

```python
laplacian = cv2.Laplacian(gray, cv2.CV_64F)
```

Recap Math Foundations file 08's second-derivative curvature discussion directly — where Sobel approximates the FIRST derivative (rate of change), the Laplacian approximates the SECOND derivative (rate of change OF the rate of change). Genuinely detects edges as ZERO-CROSSINGS in the second derivative (recap that file's critical-point discussion) rather than peaks in the first derivative — a conceptually related but mathematically distinct edge-detection philosophy, generally MORE sensitive to noise than Sobel/Canny as a direct consequence of using a higher-order derivative.

## Sharpening vs edge detection — the honest, important distinction

```
Blurring (this file)   -> REDUCES high-frequency detail/noise
Sharpening (this file)   -> AMPLIFIES existing edges/detail
Edge detection (this file) -> EXTRACTS just the edge locations as a new image,
  discarding everything else (genuinely a very different OUTPUT, not just a variation
  on sharpening)
```

## Try this

```python
# Load an image with some genuine noise (or add artificial noise using
# np.random.normal, recap Math Foundations file 10) and compare Gaussian blur,
# median blur, and bilateral filter side by side -- which preserves edges best
# while still reducing the noise?
# Then apply Sobel (both x and y, combined via magnitude) and Canny edge detection
# to the SAME image, and compare the results -- which produces cleaner, more usable
# edges? Try tuning Canny's two threshold values and observe how the detected
# edges change
```

Directly comparing all three blur types and both edge-detection methods on the SAME image is genuinely the clearest way to build real intuition about when each technique is the right choice, rather than defaulting to whichever one happens to be most familiar.

## What's next
File 06 — Morphological Operations, a specialized category of operations for cleaning up BINARY images (recap file 03's thresholding output) — genuinely essential preprocessing before file 07's contour detection.
