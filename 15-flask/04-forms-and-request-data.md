# 04 — Forms & Request Data

Recap file 03's closing note — building actual HTML forms for users to submit data, completing the input side of the request/response cycle that templates (file 03) handle for output.

## A basic HTML form

```html
<!-- templates/predict_form.html -->
<form action="/predict" method="POST">
    <label for="text_input">Enter text to analyze:</label>
    <input type="text" name="text_input" id="text_input">
    <button type="submit">Analyze</button>
</form>
```

Genuinely important detail worth being precise about: the `name` attribute (NOT `id`) is what Flask reads on the server side via `request.form` (recap file 02) — a genuinely common beginner mistake is forgetting the `name` attribute or confusing it with `id`, which is purely for CSS/JavaScript targeting, not form data submission.

## Handling the form submission

```python
from flask import Flask, request, render_template

@app.route('/predict', methods=['GET', 'POST'])
def predict():
    if request.method == 'POST':
        text = request.form.get('text_input')
        # ... recap file 06's actual model prediction logic, coming next file ...
        result = f"You submitted: {text}"
        return render_template('predict_form.html', result=result)
    return render_template('predict_form.html', result=None)
```

Genuinely the standard pattern: the SAME route handles both displaying the empty form (GET) and processing the submitted data (POST) — checking `request.method` to decide which behavior to run, directly recapping file 02's GET/POST distinction in a real, complete example.

## Different input types — genuinely useful HTML form elements

```html
<form action="/submit" method="POST">
    <input type="text" name="username" placeholder="Username">
    <input type="number" name="age" min="0" max="120">
    <input type="email" name="email">
    <input type="password" name="password">

    <select name="category">
        <option value="sports">Sports</option>
        <option value="tech">Technology</option>
    </select>

    <input type="checkbox" name="subscribe" value="yes"> Subscribe to updates

    <input type="radio" name="plan" value="free"> Free
    <input type="radio" name="plan" value="pro"> Pro

    <textarea name="comments" rows="4"></textarea>

    <button type="submit">Submit</button>
</form>
```

Genuinely worth knowing `type="number"`/`type="email"` provide BASIC client-side validation (browsers reject obviously malformed input before even submitting) — but this is genuinely NOT a substitute for server-side validation, covered next.

## Server-side validation — genuinely non-negotiable, never trust client input alone

```python
@app.route('/submit', methods=['POST'])
def submit():
    age = request.form.get('age')

    if not age or not age.isdigit():
        return render_template('form.html', error="Please enter a valid age")

    age = int(age)
    if age < 0 or age > 120:
        return render_template('form.html', error="Age must be between 0 and 120")

    return render_template('form.html', success=f"Age {age} recorded")
```

Genuinely a real, important security/reliability principle worth internalizing permanently: CLIENT-SIDE validation (HTML `type="number"`, JavaScript checks) can always be BYPASSED — a malicious or just unusual client can send whatever data it wants directly to the server, skipping the browser's form entirely (recap file 02's curl/Postman testing note — this is EXACTLY how someone could bypass form validation). SERVER-SIDE validation is the only validation that can genuinely be trusted, since it's the last line of defense before data actually gets used.

## File uploads — genuinely essential for image-based ML demos

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
    <input type="file" name="image_file" accept="image/*">
    <button type="submit">Upload</button>
</form>
```

Genuinely important, easy-to-forget detail: `enctype="multipart/form-data"` is REQUIRED on the form tag for file uploads to work at all — without it, the file data genuinely won't be transmitted correctly.

```python
import os
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = 'uploads'
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg'}

def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'image_file' not in request.files:
        return "No file part", 400

    file = request.files['image_file']
    if file.filename == '':
        return "No file selected", 400

    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)      # genuinely important -- sanitizes the filename
        filepath = os.path.join(UPLOAD_FOLDER, filename)
        file.save(filepath)
        return f"File uploaded: {filename}"

    return "Invalid file type", 400
```

`secure_filename` is genuinely a real, important security function — recap the general security-awareness theme from this whole roadmap — without it, a malicious filename like `"../../etc/passwd"` could potentially be used to write files OUTSIDE the intended upload directory (a "path traversal" attack). `secure_filename` strips out dangerous characters, making the filename safe to use directly in a filesystem path. Similarly, `ALLOWED_EXTENSIONS` checking prevents uploading arbitrary file types that could genuinely pose a security risk (e.g. uploading executable scripts disguised as images).

## Feeding an uploaded image into a CV pipeline — recap OpenCV/PyTorch directly

```python
import cv2

@app.route('/analyze-image', methods=['POST'])
def analyze_image():
    file = request.files['image_file']
    filename = secure_filename(file.filename)
    filepath = os.path.join(UPLOAD_FOLDER, filename)
    file.save(filepath)

    img = cv2.imread(filepath)      # recap OpenCV file 01 directly
    # ... run classical CV processing (OpenCV folder) or a PyTorch model (file 06) ...

    return render_template('result.html', filename=filename)
```

Genuinely the exact real-world connective tissue this whole roadmap has been building toward — a file uploaded through an HTML form, saved to disk, then processed using EXACTLY the OpenCV techniques from that entire folder, or fed into a PyTorch model (file 06) for prediction.

## CSRF protection — a genuinely important, often-overlooked concern

```python
# Genuinely worth knowing this exists: Cross-Site Request Forgery (CSRF) protection
# prevents a malicious site from tricking a logged-in user's browser into submitting
# a form to YOUR app without their knowledge
# Flask-WTF (a genuinely common extension) adds CSRF tokens to forms automatically
# -- worth knowing the name and concept now, full implementation detail is a
# "look up when building a real production form" topic rather than core to this file
```

## Try this

```python
# Build a form that accepts a piece of text (a textarea) and submits it via POST
# Add SERVER-SIDE validation: reject empty submissions, and reject submissions
# over 500 characters, returning a clear error message rendered back in the template
# Then build a SEPARATE file upload form accepting only image files, using
# secure_filename and extension checking
# Deliberately try to upload a non-image file (like a .txt or .exe) and confirm
# your validation correctly rejects it with an appropriate error message
```

Deliberately testing the validation with genuinely invalid input (empty text, oversized text, wrong file type) is the real test of whether the server-side validation is actually doing its job, not just looking correct on the happy path.

## What's next
File 05 — Static Files & Frontend Basics, adding CSS styling and JavaScript to make the app actually look and feel like a real, presentable web application rather than plain unstyled HTML.
