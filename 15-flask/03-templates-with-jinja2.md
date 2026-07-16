# 03 — Templates with Jinja2

Recap file 02's closing note — moving from returning raw strings/dicts into properly rendering dynamic HTML pages, the foundation for a real user-facing interface around a model.

## Why hardcoded HTML strings genuinely don't scale

```python
# Recap file 01's html_example -- genuinely fine for a one-line demo, but
# real pages have hundreds of lines of HTML, and mixing that directly into
# Python strings gets messy and hard to maintain FAST
```

Templates solve this by separating PRESENTATION (HTML) from LOGIC (Python) — genuinely the same separation-of-concerns principle that shows up throughout good software design.

## Setting up templates

```
my_flask_app/
├── app.py
└── templates/
    └── index.html      -- Flask automatically looks in a 'templates' folder
```

```python
from flask import render_template

@app.route('/')
def home():
    return render_template('index.html')
```

## Passing data into a template

```python
@app.route('/user/<username>')
def user_profile(username):
    return render_template('profile.html', name=username, joined_year=2026)
```

```html
<!-- templates/profile.html -->
<!DOCTYPE html>
<html>
<body>
    <h1>Welcome, {{ name }}!</h1>
    <p>Member since {{ joined_year }}</p>
</body>
</html>
```

`{{ variable_name }}` is genuinely Jinja2's core syntax for inserting a Python value into HTML — directly analogous to an f-string's `{expression}` syntax (recap the Python basics folder), just used within an HTML file instead of a Python string.

## Conditionals and loops in templates — genuinely powerful, real logic in HTML

```html
{% if prediction == 'positive' %}
    <p style="color: green;">Positive sentiment detected!</p>
{% else %}
    <p style="color: red;">Negative sentiment detected.</p>
{% endif %}

<ul>
{% for item in results %}
    <li>{{ item.name }}: {{ item.score }}</li>
{% endfor %}
</ul>
```

Genuinely the exact same `if`/`for` logic from Python (recap the Python basics folder), just using `{% %}` tags instead of colons/indentation, since HTML doesn't have Python's indentation-based syntax. This is precisely the mechanism that will display a model's prediction results (recap Classical NLP file 08's sentiment analysis, PyTorch's classification outputs) dynamically on a web page in file 06.

## Passing complex data — lists, dicts, model predictions

```python
@app.route('/results')
def show_results():
    predictions = [
        {"text": "I loved this movie!", "sentiment": "positive", "confidence": 0.95},
        {"text": "Terrible experience", "sentiment": "negative", "confidence": 0.88}
    ]
    return render_template('results.html', predictions=predictions)
```

```html
{% for pred in predictions %}
    <div class="result-card">
        <p><strong>Text:</strong> {{ pred.text }}</p>
        <p><strong>Sentiment:</strong> {{ pred.sentiment }}</p>
        <p><strong>Confidence:</strong> {{ (pred.confidence * 100) | round(1) }}%</p>
    </div>
{% endfor %}
```

Genuinely worth noticing `pred.text` (dot notation) works for dict access in Jinja2, even though it would need `pred['text']` in plain Python — a deliberate Jinja2 convenience. The `| round(1)` syntax is a **filter** — genuinely a small transformation applied to a value before display, covered more below.

## Template inheritance — the genuinely essential technique for avoiding repetition

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My ML App{% endblock %}</title>
</head>
<body>
    <nav>My App Navigation</nav>
    {% block content %}{% endblock %}
    <footer>© 2026</footer>
</body>
</html>
```

```html
<!-- templates/predict.html -->
{% extends "base.html" %}

{% block title %}Prediction Page{% endblock %}

{% block content %}
    <h1>Make a Prediction</h1>
    <p>Enter your data below</p>
{% endblock %}
```

Genuinely the template-world equivalent of Python class inheritance (recap the general OOP concept) — `base.html` defines the shared LAYOUT (navigation, footer, overall structure), and child templates only need to fill in the SPECIFIC blocks that differ per page. This is genuinely essential for any real multi-page app — without inheritance, the navigation/footer HTML would need to be copy-pasted into every single template, a real maintenance nightmare when it needs to change.

## Jinja2 filters — genuinely useful built-in transformations

```html
{{ name | upper }}                    <!-- uppercase -->
{{ price | round(2) }}                  <!-- round to 2 decimal places -->
{{ items | length }}                      <!-- count of a list -->
{{ description | truncate(100) }}           <!-- cut off long text -->
{{ user_input | escape }}                     <!-- HTML-escape user input -- genuinely a SECURITY concern -->
```

That last filter genuinely matters — recap the general security-awareness themes present throughout this roadmap. Jinja2 auto-escapes variables by DEFAULT (genuinely a real, important safety feature preventing a class of attack called Cross-Site Scripting/XSS, where malicious HTML/JavaScript submitted by a user could otherwise get rendered and executed on other users' browsers) — worth knowing this protection exists automatically, and worth NEVER disabling it (`| safe` filter) for genuinely untrusted user input.

## Linking between pages — `url_for` in templates

```html
<a href="{{ url_for('home') }}">Home</a>
<a href="{{ url_for('user_profile', username='soham') }}">My Profile</a>
```

Recap file 02's `url_for` explanation directly — the SAME "reference by function name, not hardcoded URL" best practice, now used within templates too, for the identical reason: if a route's URL changes, every link referencing it via `url_for` updates automatically.

## Static files in templates — a preview, full treatment file 05

```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
```

## Try this

```python
# Build a base.html template with a shared navigation bar and footer
# Create 2 child templates extending it: a home page and a "results" page
# The results page should receive a list of dicts (like the predictions example
# above) from Flask and display them using a Jinja2 for loop, showing each
# item's data in a styled way
# Add a Jinja2 conditional that shows a different message/color depending on
# whether a "confidence" value is above or below 0.5
# Deliberately pass a string containing an HTML tag (e.g. "<script>alert('test')</script>")
# as a variable and confirm Jinja2 auto-escapes it in the rendered output rather
# than executing it as real HTML
```

Confirming the auto-escaping behavior firsthand (seeing the tag displayed as literal text rather than executed) is genuinely a good, concrete way to understand WHY this default protection matters, rather than just being told it's a "security feature."

## What's next
File 04 — Forms & Request Data, building actual HTML forms that let users SUBMIT data into the Flask app — completing the input side of the request/response cycle that templates handle for output.
