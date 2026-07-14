# PyTorch — A to Z

Notes on PyTorch, learned right after XGBoost as the transition from classical ML into Deep Learning. Covers tensors, autograd, building networks, training loops, CNNs, RNNs, transfer learning, transformers, generative models, and deployment — theory, code, and worked examples in every file.

## Index

| # | Topic | File |
|---|---|---|
| 01 | Introduction & Why PyTorch | [01-introduction-and-why-pytorch.md](01-introduction-and-why-pytorch.md) |
| 02 | Tensors Fundamentals | [02-tensors-fundamentals.md](02-tensors-fundamentals.md) |
| 03 | Autograd & Computational Graphs | [03-autograd-and-computational-graphs.md](03-autograd-and-computational-graphs.md) |
| 04 | Building Blocks: nn.Module | [04-building-blocks-nn-module.md](04-building-blocks-nn-module.md) |
| 05 | Activation Functions | [05-activation-functions.md](05-activation-functions.md) |
| 06 | Loss Functions | [06-loss-functions.md](06-loss-functions.md) |
| 07 | Optimizers & Training Loop | [07-optimizers-and-training-loop.md](07-optimizers-and-training-loop.md) |
| 08 | Datasets & DataLoaders | [08-datasets-and-dataloaders.md](08-datasets-and-dataloaders.md) |
| 09 | Regularization & Overfitting Control | [09-regularization-and-overfitting-control.md](09-regularization-and-overfitting-control.md) |
| 10 | CNNs — Computer Vision | [10-cnns-computer-vision.md](10-cnns-computer-vision.md) |
| 11 | RNNs, LSTMs & Sequence Models | [11-rnns-lstm-sequence-models.md](11-rnns-lstm-sequence-models.md) |
| 12 | Transfer Learning | [12-transfer-learning.md](12-transfer-learning.md) |
| 13 | GPU/CUDA & Mixed Precision | [13-gpu-cuda-mixed-precision.md](13-gpu-cuda-mixed-precision.md) |
| 14 | Saving & Loading Models | [14-saving-loading-models.md](14-saving-loading-models.md) |
| 15 | Transformers & Attention | [15-transformers-attention-mechanism.md](15-transformers-attention-mechanism.md) |
| 16 | Autoencoders | [16-autoencoders.md](16-autoencoders.md) |
| 17 | GANs | [17-gans-generative-adversarial-networks.md](17-gans-generative-adversarial-networks.md) |
| 18 | Model Deployment | [18-model-deployment.md](18-model-deployment.md) |
| 19 | PyTorch → Hugging Face Bridge | [19-ml-dl-huggingface-bridge.md](19-ml-dl-huggingface-bridge.md) |

## How to read these
Each file builds directly on the ones before it — e.g. file 07's training loop assumes autograd (03), nn.Module (04), and loss functions (06) are already understood. Every file ends with a "Try this" exercise meant to actually be run, not just read.

## Roadmap position
Python Basics → NumPy → pandas → matplotlib → Seaborn → scikit-learn → XGBoost → PyTorch (done) → Hugging Face (next)
