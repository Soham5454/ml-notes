# 10 — GPU Acceleration

Genuinely relevant to my actual hardware setup — RTX 3050 Laptop GPU, 4GB VRAM, CUDA Toolkit already installed and verified working (recap from earlier PyTorch/CUDA verification work). XGBoost can use the GPU to significantly speed up training, though the considerations are somewhat different from neural network GPU usage.

## Enabling GPU training
```python
model = XGBClassifier(tree_method="hist", device="cuda")     # modern XGBoost syntax (post ~2.0) -- 'device' explicitly set to use the GPU
```
Older XGBoost versions used a slightly different parameter (`tree_method="gpu_hist"`) — worth checking the installed version's documentation, since this API genuinely changed between versions and mismatched syntax is a common source of confusion when following older tutorials/StackOverflow answers.

## Checking XGBoost's version and GPU availability
```python
import xgboost as xgb
print(xgb.__version__)

# Quick sanity check -- train something tiny and confirm no errors, rather than assuming GPU support is correctly configured
model = XGBClassifier(device="cuda")
model.fit(X_train, y_train)     # if CUDA/GPU isn't properly set up, this will error clearly rather than silently falling back
```

## Why GPU acceleration helps XGBoost specifically
Unlike some algorithms where GPU benefits are marginal, tree-building in XGBoost involves genuinely large amounts of parallel computation (evaluating potential splits across many features and data points simultaneously) — exactly the kind of workload GPUs excel at. For large datasets (many rows/columns) and many boosting rounds, GPU training can be significantly faster than CPU.

## Honest assessment for MY specific hardware (RTX 3050, 4GB VRAM)
Genuinely important to be realistic here: XGBoost's GPU benefit scales with DATASET SIZE. For smaller/medium tabular datasets (the kind typical of most practice projects and even many real business datasets — thousands to low hundreds-of-thousands of rows), CPU training is often ALREADY fast enough that GPU acceleration provides a smaller relative speedup than it would for genuinely massive datasets (millions of rows) or for neural network training specifically. The 4GB VRAM constraint that matters SO much for PyTorch/deep learning (batch sizes, model sizes) is generally LESS of a bottleneck for XGBoost, since XGBoost's memory usage patterns are different from a neural network's — this is worth remembering as an honest, realistic expectation rather than assuming GPU is always dramatically faster.

## When GPU acceleration is genuinely worth using for XGBoost
- Large datasets (hundreds of thousands+ rows, especially with many features)
- Extensive hyperparameter tuning (file 08) where MANY models get trained repeatedly — the cumulative time savings across many training runs adds up meaningfully even if each individual run's speedup is modest
- When CPU training is genuinely becoming a bottleneck in an iterative workflow

## When CPU is genuinely fine (don't over-engineer this)
- Smaller datasets (the practice/learning-stage datasets I'm currently working with)
- Quick prototyping/experimentation where iteration speed on small data matters more than raw training throughput

## Monitoring GPU usage during training (a practical habit, connects to system awareness)
```bash
nvidia-smi     # run this in a separate terminal WHILE training to see actual GPU utilization/memory usage in real time
```
Genuinely useful for confirming the GPU is actually being utilized (and to what extent) rather than just assuming the `device="cuda"` parameter worked as intended — a good habit to verify rather than assume, especially given how much attention I've already put into understanding my exact VRAM constraints for the DL work ahead.

## GPU parameters that matter for memory (connects to the same VRAM-awareness theme as the PyTorch prep work)
```python
model = XGBClassifier(device="cuda", max_bin=256)     # max_bin controls how features get discretized for GPU histogram building -- lower values use less GPU memory but can slightly reduce split-finding precision
```
Worth knowing this exists specifically because of my hardware constraints — if a genuinely large dataset ever runs into GPU memory issues with XGBoost specifically, `max_bin` is one of the first parameters worth reducing, similar in spirit to how batch size gets reduced for VRAM-constrained neural network training.

## Honest summary for this file
GPU acceleration for XGBoost is genuinely useful to know about and enable when it matters, but I'm not expecting it to be the dramatic game-changer for my current tabular-data practice work that it will likely be once I reach actual neural network training in PyTorch — different algorithm, different memory/compute profile, different realistic expectations.
