# 02 — Image Transformations

Recap file 01's closing note — the basic geometric operations used constantly in preprocessing pipelines, genuinely the same category of operations as PyTorch file 08's `torchvision.transforms`, just performed directly with OpenCV instead.

## Resizing — genuinely one of the most common preprocessing steps

```python
import cv2

img = cv2.imread('photo.jpg')

resized = cv2.resize(img, (300, 200))      # (width, height) -- note the ORDER, opposite of img.shape!
```

Genuinely important, easy-to-miss gotcha directly connecting to file 01's shape discussion: `img.shape` gives `(height, width, channels)`, but `cv2.resize()` takes `(width, height)` as its target size argument — a real, common source of accidentally-swapped dimensions.

```python
# Resizing while PRESERVING aspect ratio -- genuinely important to avoid distortion
def resize_with_aspect_ratio(img, target_width):
    h, w = img.shape[:2]
    aspect_ratio = target_width / w
    target_height = int(h * aspect_ratio)
    return cv2.resize(img, (target_width, target_height))
```

Genuinely a real, practical habit — resizing to an ARBITRARY fixed size (e.g. always 224x224 for a CNN input, recap PyTorch file 12's transfer learning preprocessing) without preserving aspect ratio can distort the image, which sometimes matters and sometimes doesn't depending on the downstream task — worth being a deliberate choice, not an accident.

## Interpolation methods — genuinely matters for quality

```python
cv2.resize(img, (300, 200), interpolation=cv2.INTER_LINEAR)      # good default, balanced speed/quality
cv2.resize(img, (300, 200), interpolation=cv2.INTER_CUBIC)         # higher quality, slower -- good for UPSCALING
cv2.resize(img, (300, 200), interpolation=cv2.INTER_AREA)            # genuinely best for DOWNSCALING
cv2.resize(img, (300, 200), interpolation=cv2.INTER_NEAREST)           # fastest, blockiest -- rarely the right choice
```

Genuinely worth knowing this isn't just a technicality — `INTER_AREA` specifically avoids aliasing artifacts when shrinking an image (by properly averaging pixel regions), while `INTER_CUBIC`/`INTER_LINEAR` interpolate smoothly when enlarging. Using the wrong one doesn't error, just silently produces lower-quality results.

## Cropping — recap file 01's ROI slicing directly

```python
cropped = img[50:250, 100:400]      # genuinely just NumPy slicing, nothing OpenCV-specific needed
```

## Flipping

```python
flipped_horizontal = cv2.flip(img, 1)      # 1 = horizontal flip (mirror left-right)
flipped_vertical = cv2.flip(img, 0)          # 0 = vertical flip (mirror top-bottom)
flipped_both = cv2.flip(img, -1)               # -1 = both at once
```

Recap PyTorch file 09's `RandomHorizontalFlip` data augmentation directly — genuinely the SAME operation, just performed manually here instead of inside a `torchvision.transforms.Compose` pipeline.

## Rotation — genuinely more involved than flipping, needs a transformation matrix

```python
import numpy as np

def rotate_image(img, angle):
    h, w = img.shape[:2]
    center = (w // 2, h // 2)
    rotation_matrix = cv2.getRotationMatrix2D(center, angle, scale=1.0)
    rotated = cv2.warpAffine(img, rotation_matrix, (w, h))
    return rotated

rotated_45 = rotate_image(img, 45)
```

Genuinely worth connecting DIRECTLY to Math Foundations file 04's rotation matrix example — `cv2.getRotationMatrix2D` builds EXACTLY the 2D rotation matrix covered there (`[[cos θ, -sin θ], [sin θ, cos θ]]`, with additional translation terms to rotate around a specific center point rather than the origin). `warpAffine` then APPLIES that matrix to every pixel coordinate — genuinely the same "matrix as a function that transforms space" idea from that file, now operating on actual image pixel coordinates instead of abstract 2D points.

## Affine transformations — a genuinely more general category

```python
# An affine transform preserves parallel lines but can rotate, scale, translate, AND shear
# (recap Math Foundations file 04's shear matrix example directly)

pts1 = np.float32([[50,50], [200,50], [50,200]])      # 3 points in the ORIGINAL image
pts2 = np.float32([[10,100], [200,50], [100,250]])       # where those 3 points should MAP TO

matrix = cv2.getAffineTransform(pts1, pts2)
transformed = cv2.warpAffine(img, matrix, (img.shape[1], img.shape[0]))
```

Genuinely satisfying to recognize — rotation, scaling, translation, and shearing (recap Math Foundations file 04's exact examples of each) are ALL special cases of this one general affine transformation, computed here from just 3 corresponding point pairs.

## Perspective transformations — a genuinely different, more powerful category

```python
# Perspective transforms do NOT preserve parallel lines -- used for "correcting"
# a skewed/angled view of something into a flat, front-on view (e.g. scanning a
# document photographed at an angle)

pts1 = np.float32([[56,65], [368,52], [28,387], [389,390]])      # 4 corners of the skewed region
pts2 = np.float32([[0,0], [300,0], [0,300], [300,300]])             # where they should map to (a perfect square)

matrix = cv2.getPerspectiveTransform(pts1, pts2)
warped = cv2.warpPerspective(img, matrix, (300, 300))
```

Genuinely a real, practical application worth knowing — this exact technique (often called a "bird's eye view" transform) is used in real document-scanning apps and in some autonomous-vehicle lane-detection pipelines to correct a camera's angled perspective into a straightened, top-down view.

## Translation — shifting an image

```python
def translate_image(img, x_shift, y_shift):
    h, w = img.shape[:2]
    matrix = np.float32([[1, 0, x_shift], [0, 1, y_shift]])
    return cv2.warpAffine(img, matrix, (w, h))
```

Genuinely worth recognizing this translation matrix's structure directly connects to Math Foundations file 04's affine-vs-linear distinction — the `+b` translation component that technically makes a transform "affine" rather than purely linear, now seen as literal, concrete numbers shifting pixel positions.

## Try this

```python
# Load an image and perform, in sequence: resize it to half its original size
# (using INTER_AREA), rotate the resized version by 30 degrees, then flip it horizontally
# Save each intermediate step as a separate file so you can visually confirm each
# transformation worked as expected
# Then take 4 corner points of a rectangular object photographed at a slight angle
# (or simulate this with 4 hand-picked points on any image) and use
# getPerspectiveTransform + warpPerspective to "straighten" it into a top-down view
```

Chaining multiple transformations and saving intermediate results is genuinely the best way to build confidence that each operation does exactly what's expected before combining them into more complex preprocessing pipelines later in this folder.

## What's next
File 03 — Color Spaces & Thresholding, moving beyond geometric operations into manipulating PIXEL VALUES themselves — color representations and the foundational technique behind separating objects from backgrounds.
