# 01 — Introduction to Flask & Web Basics

Starting the Flask phase — genuinely the practical payoff moment flagged way back when this roadmap was first being planned: this is what turns a trained model sitting in a notebook into something that can actually be demoed, shared, and shown in a portfolio.

## What Flask actually is

A genuinely lightweight, minimal Python web framework — unlike Django (a full "batteries-included" framework), Flask gives you the essentials (routing, request/response handling) and lets you add exactly what's needed, nothing more. Genuinely the standard choice for small-to-medium projects, APIs, and ML model demos specifically because of that minimalism.

## The absolute minimum Flask app

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(debug=True)
```

Genuinely worth understanding every piece of this:
- `Flask(__name__)` — creates the application object; `__name__` tells Flask where to look for resources (templates, static files, covered in files 03/05).
- `@app.route('/')` — a **decorator** (recap the general Python decorator pattern) that maps a URL path to the function below it.
- `debug=True` — enables auto-reload on code changes and detailed error pages — genuinely essential during development, and genuinely a security risk to leave on in production (flagged properly in file 11).

## How the web actually works — the request/response cycle, briefly

```
Browser sends a REQUEST (a URL, a method like GET/POST, maybe some data)
    -> Flask receives it, matches it to a route
    -> The matched Python function runs, returns some content
    -> Flask sends that content back as a RESPONSE
    -> Browser renders it
```

Genuinely worth internalizing this cycle explicitly — everything in this whole folder is, at its core, just "write a Python function that takes a request and returns a response," repeated for different URLs and purposes.

## Running the development server

```bash
python app.py
# or, the more standard/flexible way:
export FLASK_APP=app.py        # on Windows (Git Bash): set FLASK_APP=app.py
flask run
```

Genuinely important, practical note: `app.run()` starts Flask's built-in DEVELOPMENT server — fine for local testing, but explicitly NOT designed for production traffic (no real concurrency handling, not hardened for security). File 11 covers the proper production alternative (Gunicorn) once deployment is actually needed.

## WSGI — briefly, the standard underneath Flask

```python
# WSGI (Web Server Gateway Interface) is the standard Python web servers and
# frameworks use to communicate. Flask's `app` object IS a WSGI application --
# genuinely just a callable that takes a request environment and returns a response
# Not something to implement by hand, but worth knowing the name -- it's the
# reason Flask apps can be served by many different underlying servers (development
# server, Gunicorn, uWSGI) without changing the application code itself
```

## Multiple routes

```python
@app.route('/')
def home():
    return "Welcome to my app!"

@app.route('/about')
def about():
    return "This is a Flask app demonstrating ML model deployment."

@app.route('/predict')
def predict():
    return "Prediction endpoint coming in file 06!"
```

Genuinely the whole shape of a Flask app at this stage — a collection of URL-to-function mappings, each returning some content.

## Returning different content types

```python
@app.route('/json-example')
def json_example():
    return {"message": "Flask automatically converts a dict to JSON!"}      # genuinely automatic since Flask 1.1+

@app.route('/html-example')
def html_example():
    return "<h1>This is rendered as HTML</h1>"      # raw strings starting with HTML tags render as HTML
```

Genuinely worth knowing Flask's automatic dict-to-JSON conversion upfront — this becomes the foundation of file 07's REST API work, where returning structured data (not just HTML pages) is the whole point.

## The Flask project structure — a genuinely sensible starting layout

```
my_flask_app/
├── app.py              -- main application file
├── templates/           -- HTML templates (file 03)
├── static/                -- CSS, JS, images (file 05)
├── requirements.txt        -- genuinely important, recap the general dependency-management habit
└── venv/                     -- virtual environment (not committed to git)
```

Genuinely worth setting up `requirements.txt` from day one:

```bash
pip freeze > requirements.txt
```

This is precisely the SAME dependency-tracking discipline that matters for any real project — anyone (including future-you) cloning this project needs to know exactly which package versions to install.

## Virtual environments — genuinely non-negotiable practice

```bash
python -m venv venv
source venv/bin/activate          # on Windows Git Bash: source venv/Scripts/activate
pip install flask
```

Genuinely important habit, worth carrying forward from general Python practice — isolating each project's dependencies prevents version conflicts between different projects (one project needing Flask 2.0, another needing Flask 3.0, genuinely a real, common problem without virtual environments).

## Try this

```python
# Create a new Flask app with 3 routes: '/' returning a welcome message,
# '/about' returning a short bio/project description, and '/status' returning
# a JSON dict like {"status": "running", "version": "1.0"}
# Run it locally with debug=True, visit each route in a browser, and confirm
# the JSON route displays as proper JSON (not a Python dict's string representation)
# Then deliberately introduce a syntax error in one route function while debug=True
# is active, and observe Flask's detailed error page -- this is genuinely useful
# to have seen once, so it's recognizable during real debugging later
```

Seeing Flask's debug-mode error page firsthand (with the full traceback and interactive debugger) is genuinely worth doing once deliberately — it becomes a familiar, useful tool rather than a scary surprise the first time a real bug triggers it.

## What's next
File 02 — Routing & HTTP Methods, going deeper into URL patterns, dynamic routes, and the GET/POST distinction that becomes essential once actual data needs to flow INTO the app, not just out of it.
