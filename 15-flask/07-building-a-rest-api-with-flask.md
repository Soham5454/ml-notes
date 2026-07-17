# 07 — Building a REST API with Flask

Recap file 06's closing note — formalizing the JSON API pattern already previewed into a properly structured, well-documented API. Genuinely essential once the "frontend" and "model backend" need to be built/deployed as separate, independent pieces — the standard architecture for real ML products.

## What REST actually means, briefly

```
REST (Representational State Transfer) -- a set of CONVENTIONS for designing APIs:
- Resources are identified by URLs (/users, /predictions, /models)
- HTTP methods indicate the ACTION (GET=read, POST=create, PUT/PATCH=update, DELETE=remove)
- Responses are typically JSON, using appropriate status codes (recap file 02)
```

Genuinely worth knowing this is a set of CONVENTIONS, not a strict protocol — "RESTful" APIs vary in exactly how strictly they follow these ideas, but the core principles (resource-based URLs, meaningful HTTP methods, JSON responses) are genuinely near-universal in modern API design.

## Structuring a proper prediction API

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/v1/predict', methods=['POST'])
def predict():
    data = request.get_json()

    if not data or 'text' not in data:
        return jsonify({"error": "Missing 'text' field in request body"}), 400

    text = data['text']
    if not isinstance(text, str) or not text.strip():
        return jsonify({"error": "'text' must be a non-empty string"}), 400

    features = vectorizer.transform([text])
    prediction = model.predict(features)[0]
    confidence = model.predict_proba(features)[0].max()

    return jsonify({
        "prediction": str(prediction),
        "confidence": round(float(confidence), 4),
        "model_version": "1.0"
    }), 200
```

Genuinely worth noticing several deliberate, real practices here: `/api/v1/` in the URL (API VERSIONING, covered more below), explicit input VALIDATION with clear error messages (recap file 04's server-side validation discipline), `jsonify()` instead of returning a raw dict (genuinely more explicit and handles some edge cases dict auto-conversion doesn't), and CONSISTENT response structure across success/error cases.

## API versioning — genuinely important for real, evolving APIs

```python
@app.route('/api/v1/predict', methods=['POST'])
def predict_v1():
    # original behavior, genuinely never changes once clients depend on it
    ...

@app.route('/api/v2/predict', methods=['POST'])
def predict_v2():
    # improved behavior -- e.g. returns additional fields, different response structure
    ...
```

Genuinely important, real-world practice: once an API is being used by ANY external client (a frontend, a mobile app, another service), changing its response structure can BREAK that client unexpectedly. Versioning (`/v1/`, `/v2/`) lets improvements happen WITHOUT breaking existing consumers — old clients keep using `/v1/`, new ones adopt `/v2/`, and `/v1/` can eventually be deprecated on a planned timeline rather than breaking things without warning.

## Consistent error response structure — genuinely important for API usability

```python
def error_response(message, status_code):
    return jsonify({"error": message, "status": status_code}), status_code

@app.route('/api/v1/predict', methods=['POST'])
def predict():
    data = request.get_json()
    if not data:
        return error_response("Request body must be valid JSON", 400)
    if 'text' not in data:
        return error_response("Missing required field: 'text'", 400)
    # ...
```

Genuinely worth building a reusable helper like this — API clients (recap file 05's `fetch()` JavaScript example) benefit enormously from a CONSISTENT error shape, since they can write ONE piece of error-handling code that works across every endpoint, rather than needing to handle each endpoint's errors differently.

## Batch prediction endpoints — genuinely practical for real usage patterns

```python
@app.route('/api/v1/predict/batch', methods=['POST'])
def predict_batch():
    data = request.get_json()
    texts = data.get('texts', [])

    if not texts or not isinstance(texts, list):
        return error_response("'texts' must be a non-empty list", 400)

    if len(texts) > 100:      # genuinely important -- prevent unbounded resource usage
        return error_response("Maximum 100 texts per batch request", 400)

    features = vectorizer.transform(texts)      # recap scikit-learn -- genuinely more efficient
                                                     # than transforming one at a time in a loop
    predictions = model.predict(features)
    probabilities = model.predict_proba(features).max(axis=1)

    results = [
        {"text": t, "prediction": str(p), "confidence": round(float(c), 4)}
        for t, p, c in zip(texts, predictions, probabilities)
    ]
    return jsonify({"results": results}), 200
```

Genuinely important real-world efficiency point directly recapping PyTorch file 08's batching motivation — processing multiple inputs in ONE batched model call is genuinely far more efficient than making the client send 100 separate HTTP requests, each with its own overhead. The `len(texts) > 100` limit is a genuinely important, real production safeguard — without SOME limit, a client could send an enormous request that overwhelms the server's memory/compute.

## Rate limiting — a genuinely important production concern, briefly

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(app=app, key_func=get_remote_address, default_limits=["100 per hour"])

@app.route('/api/v1/predict', methods=['POST'])
@limiter.limit("10 per minute")      # stricter limit for this specific, expensive endpoint
def predict():
    ...
```

Genuinely worth knowing this exists as a real, standard concern — without rate limiting, a single client (malicious or just buggy) could overwhelm the server with requests, genuinely degrading service for everyone else. `flask-limiter` is a genuinely standard extension for this exact purpose.

## API documentation — genuinely important for anyone (including future-you) using the API

```python
@app.route('/api/v1/predict', methods=['POST'])
def predict():
    """
    Predict sentiment for a given text.

    Request body (JSON):
        text (str): The text to analyze

    Returns:
        200: {"prediction": str, "confidence": float, "model_version": str}
        400: {"error": str, "status": 400}
    """
    ...
```

Genuinely worth building the habit of documenting every endpoint's expected input/output clearly — recap the general docstring/documentation discipline that should apply throughout any real codebase. For more formal API documentation, tools like **Swagger/OpenAPI** (via `flask-smorest` or `flasgger`) can auto-generate interactive API docs from code — genuinely worth knowing these tools exist for larger, more serious API projects.

## CORS — a genuinely essential, often-confusing concern for frontend/backend separation

```python
from flask_cors import CORS

CORS(app, resources={r"/api/*": {"origins": "https://my-frontend-domain.com"}})
```

Genuinely important, real practical necessity once a frontend (hosted on one domain) needs to call a Flask API (hosted on a DIFFERENT domain) — browsers enforce a security policy (Cross-Origin Resource Sharing) that BLOCKS such cross-domain requests by default. `flask-cors` explicitly allows specified origins to access the API — a genuinely common, real "why isn't my frontend able to reach my API" debugging scenario worth knowing about upfront rather than discovering through confusion later.

## Testing an API with `curl` — genuinely essential practical skill

```bash
curl -X POST http://localhost:5000/api/v1/predict \
  -H "Content-Type: application/json" \
  -d '{"text": "This movie was fantastic!"}'
```

Genuinely worth building real comfort with `curl` (or Postman) for testing API endpoints directly — recap file 02's earlier note about needing a POST-capable tool, now the standard, real workflow for testing and debugging any API endpoint without needing a full frontend built first.

## Try this

```python
# Build a complete, versioned prediction API with:
# 1. POST /api/v1/predict -- single prediction, with proper input validation
#    and consistent error responses
# 2. POST /api/v1/predict/batch -- batch predictions with a reasonable size limit
# 3. GET /api/v1/health -- a health check endpoint (recap file 06)
# Test EVERY endpoint using curl, including deliberately malformed requests
# (missing fields, wrong data types, oversized batches) to confirm your error
# handling responds with clear, consistent JSON error messages and correct status codes
```

Testing with deliberately broken requests (not just the happy path) is genuinely the real test of whether this API would survive contact with a real, imperfect client — exactly the mindset a production API needs to be built with.

## What's next
File 08 — Blueprints & App Structure, organizing a growing Flask app (now with multiple routes, templates, and API endpoints) into a properly structured, maintainable project rather than one increasingly large `app.py` file.
