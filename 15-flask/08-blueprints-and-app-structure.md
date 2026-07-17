# 08 — Blueprints & App Structure

Recap file 07's closing note — organizing a growing Flask app into a properly structured, maintainable project rather than one increasingly large `app.py` file.

## The problem — a single-file app genuinely doesn't scale

```python
# After files 01-07, a real app.py would genuinely contain:
# - Model loading code
# - Web page routes (/,  /about, /predict)
# - API routes (/api/v1/predict, /api/v1/predict/batch, /api/v1/health)
# - Form handling, file upload handling
# - Error handlers
# Genuinely all in ONE file -- hundreds of lines, hard to navigate, hard to test
# pieces independently
```

## Blueprints — Flask's genuinely built-in solution for modular organization

```python
# app/routes/main.py
from flask import Blueprint, render_template

main_bp = Blueprint('main', __name__)

@main_bp.route('/')
def home():
    return render_template('index.html')

@main_bp.route('/about')
def about():
    return render_template('about.html')
```

```python
# app/routes/api.py
from flask import Blueprint, request, jsonify

api_bp = Blueprint('api', __name__, url_prefix='/api/v1')      # genuinely useful -- applies to ALL routes below

@api_bp.route('/predict', methods=['POST'])
def predict():
    # recap file 07's prediction logic
    ...

@api_bp.route('/health')
def health():
    return jsonify({"status": "healthy"}), 200
```

```python
# app/__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)

    from app.routes.main import main_bp
    from app.routes.api import api_bp

    app.register_blueprint(main_bp)
    app.register_blueprint(api_bp)      # genuinely, EVERY route in api_bp is now prefixed with /api/v1

    return app
```

Genuinely a blueprint is a REUSABLE, SELF-CONTAINED collection of routes — directly analogous to how a large codebase gets organized into separate modules/files (recap the general software organization principles from throughout this roadmap), just Flask's specific mechanism for doing so with routes. The `url_prefix='/api/v1'` parameter genuinely means every route registered on `api_bp` automatically gets that prefix, avoiding the need to repeat `/api/v1/` in every single route decorator.

## The application factory pattern — `create_app()`

```python
# app/__init__.py
def create_app(config_name='development'):
    app = Flask(__name__)
    app.config.from_object(config[config_name])      # covered next, config management

    # Register blueprints
    from app.routes.main import main_bp
    from app.routes.api import api_bp
    app.register_blueprint(main_bp)
    app.register_blueprint(api_bp)

    return app
```

```python
# run.py -- the actual entry point
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

Genuinely the standard, recommended Flask pattern for anything beyond a tiny toy app — wrapping app creation in a FUNCTION (rather than creating the `app` object directly at module level, as done in files 01-07) genuinely enables creating MULTIPLE app instances with different configurations, which matters enormously for TESTING (covered below) and for supporting different environments (development vs production, file 11).

## A complete, realistic project structure

```
my_ml_app/
├── app/
│   ├── __init__.py          -- create_app() factory
│   ├── routes/
│   │   ├── main.py            -- web page routes
│   │   └── api.py               -- API routes
│   ├── models/
│   │   └── ml_model.py            -- model loading/inference logic, SEPARATED from routes
│   ├── templates/
│   └── static/
├── config.py                  -- configuration classes
├── tests/                        -- covered below
│   └── test_api.py
├── run.py                          -- entry point
└── requirements.txt
```

Genuinely worth internalizing this separation of concerns explicitly — routes handle REQUEST/RESPONSE logic, `models/ml_model.py` handles MODEL logic, config handles ENVIRONMENT-specific settings. This mirrors the general software engineering principle (recap the general "separation of concerns" theme touched on in file 03's templates-vs-logic discussion) of keeping genuinely different responsibilities in genuinely different places.

## Separating model logic from routes — a genuinely important refactor

```python
# app/models/ml_model.py
import joblib

class SentimentModel:
    def __init__(self, model_path, vectorizer_path):
        self.model = joblib.load(model_path)
        self.vectorizer = joblib.load(vectorizer_path)

    def predict(self, text):
        features = self.vectorizer.transform([text])
        prediction = self.model.predict(features)[0]
        confidence = self.model.predict_proba(features)[0].max()
        return {"prediction": str(prediction), "confidence": round(float(confidence), 4)}

sentiment_model = SentimentModel('model.pkl', 'vectorizer.pkl')      # loaded ONCE, recap file 06
```

```python
# app/routes/api.py
from app.models.ml_model import sentiment_model

@api_bp.route('/predict', methods=['POST'])
def predict():
    data = request.get_json()
    result = sentiment_model.predict(data['text'])
    return jsonify(result), 200
```

Genuinely worth recognizing this as a real, meaningful improvement — the ROUTE function is now genuinely simple and readable (just request handling), while the MODEL class handles all the ML-specific logic independently, and could genuinely be TESTED on its own (covered next) without needing to spin up the whole Flask app.

## Configuration management — genuinely important for handling secrets and environments

```python
# config.py
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-key-change-in-production')
    MODEL_PATH = 'model.pkl'

class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    DEBUG = False
    SECRET_KEY = os.environ.get('SECRET_KEY')      # genuinely MUST come from environment in production

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig
}
```

Genuinely important, real security practice worth flagging directly — NEVER hardcode secrets (API keys, database passwords, `SECRET_KEY`) directly in code that might get committed to a public repo (recap the general credential-safety habit already flagged in the Hugging Face folder's Model Hub file). Using environment variables (`os.environ.get`) is the standard, correct approach — the actual secret VALUES live outside the codebase entirely.

## Basic testing — genuinely essential for a real, maintainable app

```python
# tests/test_api.py
import pytest
from app import create_app

@pytest.fixture
def client():
    app = create_app('testing')
    with app.test_client() as client:
        yield client

def test_health_check(client):
    response = client.get('/api/v1/health')
    assert response.status_code == 200
    assert response.get_json()['status'] == 'healthy'

def test_predict_missing_text(client):
    response = client.post('/api/v1/predict', json={})
    assert response.status_code == 400

def test_predict_valid_input(client):
    response = client.post('/api/v1/predict', json={'text': 'This is great!'})
    assert response.status_code == 200
    assert 'prediction' in response.get_json()
```

Genuinely worth building this habit — Flask's `test_client()` lets routes be tested WITHOUT running a real server, sending simulated requests directly to the app. This is precisely WHY the `create_app()` factory pattern matters — it lets a genuinely SEPARATE, isolated app instance be created specifically for testing, distinct from the one that would run in development or production.

## Try this

```python
# Refactor a single-file Flask app (from files 06-07's exercises) into the
# proper blueprint-based structure: separate main_bp and api_bp blueprints,
# a create_app() factory, a separate models/ file containing the ML logic
# as a class, and a config.py with at least Development and Testing configs
# Write 3 genuine tests using pytest and Flask's test_client(): one for a
# successful prediction, one for a missing-field error, and one for the
# health check endpoint
# Run the tests and confirm they all pass WITHOUT needing to manually start
# the Flask development server
```

Actually experiencing tests run against the app WITHOUT a manually-started server is genuinely the clearest demonstration of why the factory pattern and blueprint structure matter — this kind of automated testing becomes genuinely essential once the app grows past a few routes.

## What's next
File 09 — Database Integration with Flask, connecting SQLAlchemy (recap the entire SQL folder's concepts, now via Python ORM syntax) to persist data — predictions, user accounts, application state — beyond what a stateless API alone can hold.
