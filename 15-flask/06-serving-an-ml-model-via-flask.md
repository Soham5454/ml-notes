# 06 — Serving an ML Model via Flask

Recap file 05's closing note — the genuine payoff of this entire folder so far: loading a real trained model and connecting it to everything built in files 01-05. This is precisely the moment a notebook project becomes a real, demoable web application.

## Loading a scikit-learn model — recap that folder's `.fit()`/`.predict()` pattern

```python
import joblib      # genuinely the standard tool for saving/loading scikit-learn models

# Saving (done once, typically in a training script, NOT in the Flask app itself)
# joblib.dump(trained_model, 'model.pkl')

# Loading (done ONCE at app startup, NOT inside every request -- genuinely important, covered below)
model = joblib.load('model.pkl')
vectorizer = joblib.load('vectorizer.pkl')      # recap Classical NLP file 04's TfidfVectorizer, if relevant
```

Genuinely the single most important architectural decision in this whole file, worth flagging explicitly: the model is loaded ONCE, at APP STARTUP (module-level code, outside any route function) — NOT reloaded inside every single request. Loading a model from disk is genuinely slow relative to a web request's expected response time; loading it once and reusing it across all requests is essential for reasonable performance.

## A complete scikit-learn prediction endpoint

```python
from flask import Flask, request, render_template

app = Flask(__name__)
model = joblib.load('model.pkl')
vectorizer = joblib.load('vectorizer.pkl')

@app.route('/predict', methods=['GET', 'POST'])
def predict():
    if request.method == 'POST':
        text = request.form.get('text_input')
        features = vectorizer.transform([text])      # recap Classical NLP file 04's train/test .transform() discipline --
                                                          # NEVER .fit_transform() on new inference data!
        prediction = model.predict(features)[0]
        probability = model.predict_proba(features)[0].max()      # recap scikit-learn's probability output

        return render_template('predict.html', prediction=prediction, confidence=round(probability*100, 1))

    return render_template('predict.html', prediction=None)
```

Genuinely worth flagging explicitly, recapping Classical NLP file 04's exact caution: using `.transform()` (not `.fit_transform()`) on the incoming request's text is CRITICAL — the vectorizer must use the SAME vocabulary/weighting learned during training, otherwise the feature representation genuinely wouldn't match what the model was trained on, producing meaningless predictions.

## Loading and serving a PyTorch model — recap PyTorch file 14 directly

```python
import torch
from transformers import AutoTokenizer, AutoModelForSequenceClassification      # recap Hugging Face folder

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')      # recap PyTorch file 13

tokenizer = AutoTokenizer.from_pretrained('./my_finetuned_model')      # recap Hugging Face file 05's saved model
model = AutoModelForSequenceClassification.from_pretrained('./my_finetuned_model')
model.to(device)
model.eval()      # recap PyTorch file 04/09 -- CRITICAL, disables dropout/batchnorm training behavior

@app.route('/api/predict', methods=['POST'])
def api_predict():
    data = request.get_json()
    text = data.get('text', '')

    inputs = tokenizer(text, return_tensors='pt', truncation=True, padding=True).to(device)

    with torch.no_grad():      # recap PyTorch file 03 -- no gradients needed for inference
        outputs = model(**inputs)

    probs = torch.softmax(outputs.logits, dim=-1)      # recap PyTorch file 05
    predicted_class = torch.argmax(probs, dim=-1).item()
    confidence = probs[0][predicted_class].item()

    label = model.config.id2label[predicted_class]      # recap Hugging Face file 03

    return {"prediction": label, "confidence": round(confidence * 100, 2)}
```

Genuinely satisfying to see this — EVERY piece of this endpoint directly recaps a specific concept already covered: `model.eval()` (PyTorch files 04/09), `torch.no_grad()` (PyTorch file 03), softmax/argmax (PyTorch file 05), `id2label` (Hugging Face file 03). This route is genuinely just "the manual inference code from Hugging Face file 03, wrapped in a Flask endpoint."

## Loading a PyTorch CNN for image classification — recap PyTorch file 10, OpenCV folder

```python
from torchvision import transforms
from PIL import Image

cnn_model = torch.load('cnn_model.pt', map_location=device)      # recap PyTorch file 14's map_location note
cnn_model.eval()

transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])      # recap PyTorch file 12
])

@app.route('/api/classify-image', methods=['POST'])
def classify_image():
    file = request.files['image_file']      # recap file 04's file upload handling
    img = Image.open(file.stream).convert('RGB')

    tensor_img = transform(img).unsqueeze(0).to(device)      # recap PyTorch file 02's unsqueeze

    with torch.no_grad():
        output = cnn_model(tensor_img)
        probs = torch.softmax(output, dim=1)
        predicted_class = torch.argmax(probs, dim=1).item()

    return {"class": predicted_class, "confidence": round(probs[0][predicted_class].item() * 100, 2)}
```

Genuinely the direct continuation of OpenCV file 12's bridge discussion — an uploaded image flows through PIL/torchvision preprocessing (the SAME normalization/resizing already covered in PyTorch file 12), then through a trained CNN, exactly the "OpenCV/PIL for I/O, PyTorch for inference" pattern from that bridge file, now wired into a real, working web endpoint.

## Error handling for model inference — genuinely important, real-world necessary

```python
@app.route('/predict', methods=['POST'])
def predict():
    text = request.form.get('text_input')

    if not text or not text.strip():
        return render_template('predict.html', error="Please enter some text")

    try:
        features = vectorizer.transform([text])
        prediction = model.predict(features)[0]
        return render_template('predict.html', prediction=prediction)
    except Exception as e:
        # Genuinely important: log the REAL error server-side for debugging,
        # but DON'T expose raw exception details to the end user (a real security concern)
        app.logger.error(f"Prediction failed: {e}")
        return render_template('predict.html', error="Something went wrong processing your request")
```

Genuinely important practice worth internalizing — a model can fail in real, unexpected ways (malformed input, unexpected data types, genuinely unpredictable edge cases) — a production-quality endpoint should NEVER let an unhandled exception crash the app or leak internal error details to users, both a reliability concern and a real security consideration.

## Model warm-up and health checks — genuinely practical production concerns

```python
@app.route('/health')
def health_check():
    return {"status": "healthy", "model_loaded": model is not None}, 200
```

Genuinely a standard, real practice worth knowing — a `/health` endpoint lets monitoring systems (or a load balancer, relevant once file 11 covers deployment) verify the app AND its model are actually working correctly, distinct from just checking that the Flask server process is running at all.

## Try this

```python
# Take a model already trained somewhere in this repo (a scikit-learn classifier
# from Classical NLP file 07, or a fine-tuned Hugging Face model from that folder's
# file 05) and save it properly (joblib.dump or save_pretrained)
# Build a complete Flask app: load the model ONCE at startup, create a form
# (recap file 04) for text input, and a route that runs real inference and
# displays the actual prediction with confidence score using a template (file 03)
# with basic styling (file 05)
# Test it with genuinely varied inputs, including an empty submission and a
# very long piece of text, confirming your error handling catches edge cases
# gracefully rather than crashing
```

Actually connecting a REAL, previously-trained model (not a toy placeholder) is genuinely the milestone this entire Flask folder has been building toward — a working, presentable, portfolio-ready ML demo application.

## What's next
File 07 — Building a REST API with Flask, formalizing the JSON API pattern already previewed in this file into a properly structured, well-documented API — genuinely essential once the "frontend" and "model backend" need to be built/deployed as separate, independent pieces.
