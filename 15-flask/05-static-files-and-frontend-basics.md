# 05 — Static Files & Frontend Basics

Recap file 04's closing note — adding CSS and JavaScript to make the app look and feel like a real, presentable application, genuinely important for a portfolio-quality ML demo project.

## The static folder — where CSS, JS, and images live

```
my_flask_app/
├── app.py
├── templates/
│   └── index.html
└── static/
    ├── style.css
    ├── script.js
    └── images/
        └── logo.png
```

Flask automatically serves anything in the `static/` folder — recap file 03's `url_for('static', filename='style.css')` preview, now fully explained: this generates the correct URL path (`/static/style.css`) without hardcoding it, the SAME `url_for` best practice from files 02-03.

## Linking CSS

```html
<!-- templates/base.html -->
<head>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
```

```css
/* static/style.css */
body {
    font-family: 'Segoe UI', sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.result-card {
    background: white;
    border-radius: 8px;
    padding: 16px;
    margin: 10px 0;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.positive { color: #2ecc71; }
.negative { color: #e74c3c; }
```

Genuinely worth building at least basic CSS competence for this phase — a working ML demo with zero styling looks genuinely unpolished compared to one with even simple, clean CSS, and this matters real for a portfolio piece meant to impress in interviews.

## Applying dynamic classes from Jinja2 — connecting file 03 and CSS directly

```html
{% for pred in predictions %}
    <div class="result-card {{ 'positive' if pred.sentiment == 'positive' else 'negative' }}">
        <p>{{ pred.text }}</p>
        <p>Sentiment: {{ pred.sentiment }}</p>
    </div>
{% endfor %}
```

Genuinely a clean, real pattern — Jinja2's inline conditional expression (`'positive' if condition else 'negative'`) dynamically applies a CSS class based on the prediction's actual value, directly connecting the model's output (recap PyTorch/scikit-learn's classification predictions) to visual styling.

## Basic JavaScript — making the page interactive without a full reload

```html
<!-- templates/predict.html -->
<button onclick="showLoading()">Predict</button>
<div id="loading" style="display:none;">Analyzing...</div>

<script src="{{ url_for('static', filename='script.js') }}"></script>
```

```javascript
// static/script.js
function showLoading() {
    document.getElementById('loading').style.display = 'block';
}
```

Genuinely a small but real usability improvement — showing a loading indicator while a prediction request processes (especially relevant once file 06's model inference takes a noticeable moment) makes the app feel more responsive and professional.

## AJAX — submitting a form WITHOUT a full page reload

```javascript
// static/script.js
async function submitPrediction() {
    const text = document.getElementById('text_input').value;

    const response = await fetch('/api/predict', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: text })
    });

    const data = await response.json();
    document.getElementById('result').innerText =
        `Prediction: ${data.prediction} (${data.confidence}% confident)`;
}
```

```html
<input type="text" id="text_input">
<button onclick="submitPrediction()">Predict</button>
<div id="result"></div>
```

Genuinely the modern, standard way to build a responsive single-page-feeling experience — `fetch()` sends a request to Flask's JSON API endpoint (recap file 02's JSON handling, fully built out in file 07) and updates just the relevant part of the page with the result, WITHOUT reloading the whole page like a traditional form submission would. This is genuinely the JavaScript-side counterpart to the REST API Flask will serve in file 07.

## Bootstrap — a genuinely fast way to get professional-looking styling

```html
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1 class="text-primary">My ML Demo</h1>
        <button class="btn btn-primary">Predict</button>
        <div class="alert alert-success">Prediction successful!</div>
    </div>
</body>
```

Genuinely worth knowing this exists as a real, practical shortcut — Bootstrap (or similar CSS frameworks like Tailwind) provides pre-built, polished CSS classes, letting a genuinely presentable UI get built quickly WITHOUT needing to write extensive custom CSS from scratch — a real, practical time-saver for a portfolio project where the ML logic matters more than hand-crafted design.

## Serving user-uploaded files back — recap file 04's upload handling

```python
from flask import send_from_directory

@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(UPLOAD_FOLDER, filename)
```

```html
<img src="{{ url_for('uploaded_file', filename=uploaded_filename) }}" alt="Uploaded image">
```

Genuinely important pattern connecting directly to file 04's image upload example — after saving an uploaded image, this is how it gets DISPLAYED back to the user (e.g. showing the image alongside its CV/model analysis results).

## A complete, realistic page — pulling files 03-05 together

```html
{% extends "base.html" %}
{% block content %}
<div class="container">
    <h1>Sentiment Analysis Demo</h1>
    <form id="predict-form">
        <textarea id="text_input" placeholder="Enter text to analyze..."></textarea>
        <button type="button" onclick="submitPrediction()">Analyze</button>
    </form>
    <div id="loading" style="display:none;">Analyzing...</div>
    <div id="result" class="result-card"></div>
</div>
{% endblock %}
```

Genuinely the realistic shape of a complete, working ML demo page — template inheritance (file 03), a form (file 04), CSS styling and JavaScript-driven AJAX interactivity (this file), all working together, ready for file 06 to connect the actual model.

## Try this

```python
# Take the prediction form/template built in file 04's exercise and add:
# 1. Basic CSS styling (a clean container, styled buttons, a styled result card)
#    either hand-written or using Bootstrap's CDN link
# 2. A loading indicator that shows while "processing" (simulate a delay with
#    JavaScript's setTimeout if no real model is connected yet)
# 3. Convert the form submission to use fetch()/AJAX instead of a traditional
#    full-page form POST, updating just the result div with the response
# Compare the user experience before and after adding AJAX -- does the page
# feel noticeably more responsive without the full reload?
```

Actually feeling the difference between a traditional full-page-reload form and an AJAX-powered one is genuinely the best way to understand why AJAX became the standard pattern for modern, responsive web apps.

## What's next
File 06 — Serving an ML Model via Flask, the genuine payoff of this entire folder so far: loading a real trained model (recap PyTorch/scikit-learn/Hugging Face) and connecting it to everything built in files 01-05.
