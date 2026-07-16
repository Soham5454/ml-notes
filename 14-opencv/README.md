# OpenCV — A to Z

Notes on classical, practical computer vision with OpenCV — the toolkit that sits alongside PyTorch's CNN theory (file 10 of that folder). Where PyTorch explained how a CNN learns features automatically, this folder covers the hand-crafted, classical techniques that came before deep learning for vision, and still matter today for preprocessing, lightweight tasks, and understanding what CNNs are really doing.

## Index

| # | Topic | File |
|---|---|---|
| 01 | Introduction to OpenCV & Image Fundamentals | [01-introduction-to-opencv-and-image-fundamentals.md](01-introduction-to-opencv-and-image-fundamentals.md) |
| 02 | Image Transformations | [02-image-transformations.md](02-image-transformations.md) |
| 03 | Color Spaces & Thresholding | [03-color-spaces-and-thresholding.md](03-color-spaces-and-thresholding.md) |
| 04 | Drawing & Image Arithmetic | [04-drawing-and-image-arithmetic.md](04-drawing-and-image-arithmetic.md) |
| 05 | Filtering, Blurring & Edge Detection | [05-filtering-blurring-edge-detection.md](05-filtering-blurring-edge-detection.md) |
| 06 | Morphological Operations | [06-morphological-operations.md](06-morphological-operations.md) |
| 07 | Contours & Shape Detection | *(coming soon)* |
| 08 | Feature Detection & Matching | *(coming soon)* |
| 09 | Image Segmentation | *(coming soon)* |
| 10 | Object Detection (Classical) | *(coming soon)* |
| 11 | Video Processing & Real-time Analysis | *(coming soon)* |
| 12 | OpenCV → Deep Learning CV Bridge | *(coming soon)* |

## How to read these
Files 01-04 build the fundamentals (images as arrays, transformations, color, drawing). Files 05-06 cover convolution-based filtering and binary-image cleanup — genuinely essential preprocessing before files 07+ can reliably detect and analyze objects. Every file cross-references back to NumPy, Math Foundations, and PyTorch's CNN chapter throughout.

## Roadmap position
Python Basics → NumPy → pandas → Matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch → Math Foundations → DSA → SQL → Classical NLP → Hugging Face → **OpenCV (in progress)** → Flask → MLOps → Cloud → Frontier GenAI → ML System Design → Interview Prep
