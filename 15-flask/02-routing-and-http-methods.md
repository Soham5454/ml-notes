# 02 — Routing & HTTP Methods

Recap file 01's closing note — going deeper into URL patterns and the GET/POST distinction, genuinely essential once data needs to flow INTO the app, not just out.

## Dynamic routes — capturing values from the URL itself

```python
@app.route('/user/<username>')
def show_user(username):
    return f"Hello, {username}!"

@app.route('/predict/<int:model_id>')      # type converters -- genuinely important for validation
def predict_with_model(model_id):
    return f"Using model #{model_id}"
```

Genuinely important, practical detail: `<username>` captures ANY string by default, while `<int:model_id>` restricts it to integers ONLY — visiting `/predict/abc` would genuinely return a 404 automatically, since `abc` doesn't match the `int` converter. Other converters exist too: `<float:value>`, `<path:subpath>` (captures slashes too, unlike the default string converter).

## HTTP methods — GET vs POST, the genuinely fundamental distinction

```python
from flask import request

@app.route('/submit', methods=['GET', 'POST'])
def submit():
    if request.method == 'POST':
        return "Data was submitted via POST"
    else:
        return "This is a GET request -- just viewing the page"
```

Genuinely the core distinction worth understanding precisely:
- **GET** — requesting/retrieving data. Parameters go in the URL itself (`?key=value`). Should NOT change server-side state (a GET request is expected to be safe to repeat/cache).
- **POST** — sending/submitting data. Parameters go in the request BODY, not the URL. Used for creating/changing something (form submissions, file uploads — file 04, model prediction requests — file 06).

Genuinely important, real convention worth internalizing: browsers/tools may CACHE or RE-SEND GET requests without warning (bookmark it, hit back button), but should always explicitly confirm before re-sending a POST — this is precisely WHY actions with real side effects (submitting a prediction request, saving data) should genuinely use POST, not GET.

## Reading GET parameters — query strings

```python
@app.route('/search')
def search():
    query = request.args.get('q', default='', type=str)      # /search?q=hello -> query='hello'
    return f"Searching for: {query}"
```

`request.args` genuinely holds the URL's query string parameters — `.get('q', default='')` safely retrieves a value with a fallback if it's missing, avoiding a `KeyError` that a direct `request.args['q']` access would raise.

## Reading POST data — form submissions

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')
    return f"Login attempt for {username}"
```

Genuinely important distinction from `request.args`: `request.form` reads data sent in a POST request's BODY (typically from an HTML form, full treatment in file 04), while `request.args` reads URL query parameters — these are genuinely DIFFERENT data sources, easy to confuse when first learning Flask.

## Reading JSON data — genuinely essential for APIs (full treatment file 07)

```python
@app.route('/api/predict', methods=['POST'])
def api_predict():
    data = request.get_json()      # parses a JSON request body into a Python dict
    feature_value = data.get('feature')
    return {"received": feature_value, "prediction": "placeholder"}
```

Genuinely the pattern used constantly once building an actual ML prediction API (file 06/07) — a client sends JSON data (e.g. `{"feature": 5.2}`), Flask parses it directly into a usable Python dict, no manual string parsing required.

## Status codes — genuinely important, often-overlooked practice

```python
@app.route('/resource/<int:id>')
def get_resource(id):
    if id > 100:
        return {"error": "Not found"}, 404      # returning a tuple: (response, status_code)
    return {"id": id, "data": "some value"}, 200
```

Genuinely important habit worth building — HTTP status codes communicate SEMANTIC meaning beyond just the response content: `200` (OK), `201` (Created), `400` (Bad Request — client sent something wrong), `404` (Not Found), `500` (Internal Server Error). A well-built API returns the CORRECT status code alongside its content, genuinely essential for any client code (or automated testing) that needs to programmatically check whether a request succeeded, rather than parsing response text to guess.

## Redirects — sending the client to a different URL

```python
from flask import redirect, url_for

@app.route('/old-page')
def old_page():
    return redirect(url_for('home'))      # url_for looks up the URL for a route BY FUNCTION NAME, not hardcoded string

@app.route('/')
def home():
    return "This is the home page"
```

`url_for('home')` is genuinely a real, important best practice — referencing routes by their FUNCTION NAME rather than hardcoding the URL string means if the route's URL path ever changes (`/` becomes `/homepage`), every `url_for('home')` call updates automatically, while hardcoded `'/'` strings scattered throughout the code would all need manual updating.

## Error handling — custom error pages

```python
@app.errorhandler(404)
def page_not_found(e):
    return {"error": "This resource doesn't exist"}, 404

@app.errorhandler(500)
def internal_error(e):
    return {"error": "Something went wrong on our end"}, 500
```

Genuinely important for a real API (fully relevant to file 06's model-serving endpoint) — without custom error handlers, Flask returns a generic, unhelpful HTML error page even for a JSON API, which is genuinely confusing for API clients expecting JSON responses consistently.

## Try this

```python
# Build a small Flask app with:
# 1. A dynamic route /square/<int:number> that returns {"input": number, "result": number**2}
# 2. A route /greet accepting BOTH GET (reading a 'name' query parameter) and POST
#    (reading a 'name' form field) that returns a greeting either way
# 3. A custom 404 error handler that returns a JSON error message instead of
#    Flask's default HTML error page
# Test all of these using both a browser (for GET requests) and a tool like
# curl or Postman (for POST requests) -- confirm the correct data flows through
# request.args vs request.form correctly for each method
```

Testing with both a browser AND a POST-capable tool (curl/Postman) is genuinely important — GET routes are easy to test by just visiting a URL, but POST routes genuinely need a proper tool since browsers don't send POST requests by simply navigating to a URL.

## What's next
File 03 — Templates with Jinja2, moving from returning raw strings/dicts into properly rendering dynamic HTML pages — the foundation for building an actual user-facing interface around a model.
