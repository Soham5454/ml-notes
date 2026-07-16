# 01 — Introduction to OpenCV & Image Fundamentals

Starting the OpenCV phase — genuinely the practical, classical computer vision toolkit that sits alongside the CNN theory already covered in PyTorch file 10. Where that file explained HOW a CNN learns features from raw pixels, this folder covers the CLASSICAL image-processing techniques that came before (and still work alongside) deep learning for vision tasks.

## What OpenCV actually is

Open Source Computer Vision Library — a genuinely huge, mature library (originally C++, with excellent Python bindings) covering everything from basic image loading to advanced deep-learning-based detection. Genuinely the standard tool for practical, non-research computer vision work.

## An image, as data — recap NumPy directly

```python
import cv2
import numpy as np

img = cv2.imread('photo.jpg')
print(type(img))    # <class 'numpy.ndarray'>
print(img.shape)     # (height, width, channels) -- e.g. (480, 640, 3)
print(img.dtype)      # uint8 -- values from 0 to 255
```

Genuinely the single most important realization to start with: **an image IS just a NumPy array.** Recap the NumPy folder directly — every single operation covered there (indexing, slicing, broadcasting, reshaping) applies DIRECTLY to images, since that's genuinely all an image is under the hood: a 3D array of pixel intensity values.

## The BGR vs RGB gotcha — genuinely the most common OpenCV beginner trap

```python
img_bgr = cv2.imread('photo.jpg')      # OpenCV loads images in BGR order, NOT RGB!

img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)      # convert for correct display elsewhere
```

Genuinely a real, historical quirk of OpenCV (dating back to early camera hardware conventions) that trips up nearly everyone at first — displaying a BGR image using a library that expects RGB (like `matplotlib`, recap that folder) produces visibly WRONG colors (blues and reds swapped). Worth internalizing this conversion as an automatic habit whenever OpenCV output needs to be displayed or handed to a non-OpenCV tool.

```python
import matplotlib.pyplot as plt      # recap the matplotlib folder directly

plt.imshow(cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB))
plt.show()
# Without the conversion, plt.imshow(img_bgr) would show wrong colors
```

## Reading, displaying, and writing images

```python
img = cv2.imread('photo.jpg')

cv2.imshow('My Image', img)      # opens a native window -- genuinely only works in a local script,
cv2.waitKey(0)                     # NOT in most notebook environments (use matplotlib there instead)
cv2.destroyAllWindows()

cv2.imwrite('output.jpg', img)       # save an image to disk
```

`cv2.waitKey(0)` genuinely means "wait indefinitely for a keypress" — a real, common gotcha for beginners running this in a script expecting it to just display and move on; without a keypress the window will hang. `cv2.waitKey(1)` (wait 1ms) is the standard choice for video/webcam loops, covered in file 11.

## Pixel access and manipulation — recap NumPy indexing directly

```python
pixel = img[100, 50]          # (row, col) -- recap NumPy's row-major indexing convention
print(pixel)                    # [B, G, R] values, e.g. [120, 80, 200]

img[100, 50] = [255, 255, 255]     # set a single pixel to white

roi = img[50:150, 100:200]           # Region of Interest -- recap NumPy slicing EXACTLY
```

Genuinely important, direct recap: image ROW/COLUMN indexing is `img[y, x]`, NOT `img[x, y]` — this trips people up because it's the OPPOSITE of how (x, y) coordinates are usually thought about intuitively, but matches NumPy's standard (row, column) array convention exactly.

## Image dimensions and channels — genuinely important to keep straight

```python
height, width, channels = img.shape
print(f"Height: {height}, Width: {width}, Channels: {channels}")

# Grayscale images have NO channel dimension at all
gray = cv2.imread('photo.jpg', cv2.IMREAD_GRAYSCALE)
print(gray.shape)   # (480, 640) -- genuinely just 2D, no third dimension
```

Recap PyTorch file 10's CNN input shape discussion (`[batch, channels, height, width]`) — genuinely worth noting OpenCV's convention is `(height, width, channels)`, channels LAST, which is the OPPOSITE ordering from PyTorch's `channels, height, width` convention. This is a real, practical detail that matters a lot once file 12 bridges OpenCV preprocessing into a PyTorch model — a transpose/permute step (recap PyTorch file 02) is genuinely required to convert between these two conventions.

## Splitting and merging channels

```python
b, g, r = cv2.split(img)      # separates into 3 individual 2D arrays
merged = cv2.merge([b, g, r])   # recombines them

# Genuinely often FASTER using NumPy slicing directly instead of cv2.split:
blue_channel = img[:, :, 0]
```

## Basic image properties — a quick, practical inspection habit

```python
print(f"Size: {img.size}")           # total number of pixel VALUES (height * width * channels)
print(f"Data type: {img.dtype}")       # uint8 -- important later for arithmetic (file 04's overflow note)
```

## Try this

```python
# Load any image, print its shape/dtype, and confirm channels=3
# Convert it to grayscale using cv2.cvtColor(img, cv2.COLOR_BGR2GRAY) and confirm the
# shape drops to 2D
# Extract a 100x100 pixel region from the CENTER of the image using NumPy slicing
# (recap the ROI example above) and save it as a separate file using cv2.imwrite
# Deliberately load the SAME image with matplotlib's plt.imshow() WITHOUT converting
# BGR to RGB first, and confirm the colors look visibly wrong -- then fix it and compare
```

Deliberately triggering the BGR/RGB color-swap bug once, then fixing it, is genuinely the fastest way to make sure this specific gotcha never causes confusion later in the folder.

## What's next
File 02 — Image Transformations: resizing, cropping, rotating, and flipping — the basic geometric operations used constantly in preprocessing pipelines before any further analysis.
