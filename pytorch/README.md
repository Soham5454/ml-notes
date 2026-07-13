# PyTorch — A to Z

My self-taught deep learning notes, continuing straight on from the XGBoost folder in this repo. Same format as before: theory + code + notes + examples in every file, written as I actually learn it, not copy-pasted docs.

## Roadmap position
Python Basics → NumPy → pandas → matplotlib → Seaborn → scikit-learn → XGBoost → **PyTorch (in progress)** → Hugging Face

## Files (Part 1 — foundations, files 01-10)

1. `01-introduction-and-why-pytorch.md` — what PyTorch is, why not just NumPy, installation
2. `02-tensors-fundamentals.md` — tensor creation, indexing, reshaping, broadcasting, GPU tensors
3. `03-autograd-and-computational-graphs.md` — how automatic differentiation actually works
4. `04-building-blocks-nn-module.md` — nn.Module, nn.Linear, nn.Sequential, parameters
5. `05-activation-functions.md` — ReLU, Sigmoid, Tanh, Softmax, LeakyReLU, GELU
6. `06-loss-functions.md` — MSE, BCE, CrossEntropy, custom loss functions
7. `07-optimizers-and-training-loop.md` — SGD, Adam, the 5-step training loop pattern
8. `08-datasets-and-dataloaders.md` — Dataset class, DataLoader, transforms, batching
9. `09-regularization-and-overfitting-control.md` — dropout, batchnorm, weight decay, early stopping, augmentation
10. `10-cnns-computer-vision.md` — Conv2d, pooling, full CNN, MNIST training loop

## Coming in Part 2 (files 11-19 + this README gets updated)

11. RNNs, LSTMs, GRUs — sequence models
12. Transfer learning — pretrained models, fine-tuning
13. GPU/CUDA training — mixed precision, performance
14. Saving & loading models — state_dict, checkpoints
15. Transformers & attention — the mechanism behind modern NLP
16. Autoencoders — representation learning, dimensionality reduction
17. GANs — generative adversarial networks
18. Model deployment — TorchScript, ONNX, serving basics
19. ML→DL→Hugging Face bridge — final connective-tissue file for the roadmap, same style as the XGBoost bridge file

## How to read these
Each file assumes the previous ones in the folder are already read — concepts build directly on each other (e.g. file 07's training loop assumes files 03/04/06 are understood first). Every file ends with a "Try this" exercise — actually doing these is where the concepts stick, not just reading the theory.
