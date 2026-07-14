# 18 — Model Deployment

Every file so far ends with a trained model sitting in a Python variable inside a script/notebook. Deployment is about getting that model into a form that can actually serve real predictions to a real application.

## The core problem with using a raw PyTorch model in production

A live PyTorch model needs the full PyTorch library, the exact model class definition (recap file 14's `state_dict` gotcha), and a Python environment — genuinely heavy dependencies for something that might need to run on a phone, a browser, or a lightweight server. Deployment formats exist specifically to strip this down to just "run the trained computation," without needing the training framework at all.

## TorchScript — PyTorch's own serialization/deployment format

Two ways to convert a model to TorchScript, already touched on in file 14:

```python
import torch

model.eval()

# Option 1: Tracing — runs the model once with example input, records the operations
example_input = torch.rand(1, 3, 224, 224)
traced_model = torch.jit.trace(model, example_input)
traced_model.save('model_traced.pt')

# Option 2: Scripting — analyzes the actual Python code directly (handles control flow better)
scripted_model = torch.jit.script(model)
scripted_model.save('model_scripted.pt')
```

**Tracing vs scripting — genuinely important distinction:** tracing just records what happened during ONE example forward pass — if the model has data-dependent control flow (an `if` statement that behaves differently depending on input values, common in RNNs or custom architectures), tracing can silently bake in only the branch that happened to execute during that one example, producing wrong results for other inputs. Scripting actually parses the Python code and correctly preserves control flow, but requires the code to be written in a TorchScript-compatible subset of Python (no arbitrary Python objects/dynamic typing tricks). Rule of thumb: use tracing for simple feedforward architectures (most CNNs), use scripting when the model has real conditional logic.

Loading a TorchScript model needs no original class definition at all:

```python
loaded_model = torch.jit.load('model_traced.pt')
loaded_model.eval()
output = loaded_model(new_input)
```

This is genuinely the big win over plain `state_dict` — the file is fully self-contained, can even be loaded from C++ via LibTorch, no Python required at inference time at all.

## ONNX — cross-framework deployment format

```python
dummy_input = torch.rand(1, 3, 224, 224)
torch.onnx.export(
    model, dummy_input, "model.onnx",
    input_names=['input'], output_names=['output'],
    dynamic_axes={'input': {0: 'batch_size'}, 'output': {0: 'batch_size'}}   # allow variable batch size
)
```

ONNX (Open Neural Network Exchange) is genuinely framework-agnostic — a model exported from PyTorch can be loaded and run using ONNX Runtime, or even converted for use in TensorFlow-based systems, mobile runtimes, or specialized hardware accelerators. Useful specifically when the deployment target doesn't use PyTorch/LibTorch at all.

```python
import onnxruntime as ort
import numpy as np

session = ort.InferenceSession("model.onnx")
input_data = np.random.rand(1, 3, 224, 224).astype(np.float32)
outputs = session.run(None, {'input': input_data})
```

`dynamic_axes` matters genuinely in real deployment — without it, the exported model is locked to the exact batch size used during export (usually 1), and can't process a batch of 8 requests at once later without re-exporting.

## Serving a model behind a simple API — the actual "make it usable" step

A minimal FastAPI example — genuinely the most common lightweight pattern for serving a PyTorch model as a web API:

```python
from fastapi import FastAPI
import torch

app = FastAPI()
model = torch.jit.load('model_traced.pt')
model.eval()

@app.post("/predict")
def predict(data: list[float]):
    input_tensor = torch.tensor(data).unsqueeze(0)   # add batch dimension, recap file 02
    with torch.no_grad():                              # no gradients needed for inference, recap file 03
        output = model(input_tensor)
    return {"prediction": output.tolist()}
```

`torch.no_grad()` is genuinely non-optional in a serving context — every inference request that builds an unnecessary computation graph wastes memory and compute for gradients that will never be used. This gets forgotten surprisingly often even by people who remembered it correctly during training.

## Model optimization for deployment — quantization

```python
quantized_model = torch.quantization.quantize_dynamic(
    model, {torch.nn.Linear}, dtype=torch.qint8
)
```

Quantization converts model weights from 32-bit floats down to 8-bit integers — genuinely shrinks model size roughly 4x and speeds up inference on CPU, at a small, usually acceptable accuracy cost. Different from the mixed-precision training in file 13 — that was about speeding up TRAINING using 16-bit floats; quantization here is specifically about shrinking a model for INFERENCE using 8-bit integers, a distinct technique for a distinct stage of the model's life.

## The honest deployment checklist I'm taking forward

1. Train and validate the model normally (files 06-14).
2. Load the BEST checkpoint (recap file 14), not just the last one.
3. Set `model.eval()` — non-negotiable, dropout/batchnorm must behave in inference mode (recap files 04/09).
4. Convert to TorchScript or ONNX depending on the deployment target.
5. Wrap inference in `torch.no_grad()` always.
6. Consider quantization if size/speed on the deployment device is a real constraint.
7. Serve behind an API (FastAPI/Flask) or embed directly in a mobile/edge app via LibTorch/ONNX Runtime.

## Try this

```python
# Take the SimpleCNN from file 10, trained on MNIST
# Export it via torch.jit.trace, save it, then load it back in a FRESH script
# (one that never imports the SimpleCNN class at all) and confirm it can still run inference correctly
```

Confirming the traced model runs correctly WITHOUT the original class definition anywhere in scope is genuinely the clearest proof that TorchScript is doing what it claims — a fully self-contained deployable artifact.

## What's next
File 19 — the final bridge file, connecting everything learned across this PyTorch folder forward into Hugging Face, same style as the XGBoost→PyTorch bridge that started this whole folder.
