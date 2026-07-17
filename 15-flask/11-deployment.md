# 11 — Deployment

Recap file 10's closing note — taking the app out of local development and onto a real, publicly-accessible server. Genuinely the final practical step turning this entire folder's work into something actually shareable in a portfolio or with real users.

## Why the development server genuinely isn't enough — recap file 01's warning

```python
# app.run(debug=True) -- recap file 01's explicit warning:
# 1. Single-threaded by default -- genuinely can't handle multiple simultaneous
#    requests efficiently (a real problem the moment more than one person visits at once)
# 2. debug=True exposes an INTERACTIVE DEBUGGER in error pages -- genuinely a
#    severe security risk in production, since it can allow arbitrary code execution
#    if an attacker can trigger an error
# 3. Not hardened against the kinds of traffic/edge cases a real public server faces
```

Genuinely worth taking this warning seriously — deploying with `debug=True` and the built-in dev server is a real, common, and genuinely dangerous mistake for a public-facing app.

## Gunicorn — the genuinely standard production WSGI server for Flask

```bash
pip install gunicorn

gunicorn --workers 4 --bind 0.0.0.0:8000 run:app
```

Genuinely worth understanding what this does: `run:app` tells Gunicorn to import the `app` object from `run.py` (recap file 08's application factory pattern — genuinely this is precisely why that pattern matters, Gunicorn needs a real, importable app object). `--workers 4` runs FOUR separate worker processes, letting Gunicorn genuinely handle multiple simultaneous requests in parallel — recap the general concurrency concept, directly solving the single-threaded limitation of the development server.

## Environment variables — recap file 08's config management, now genuinely essential

```bash
export FLASK_ENV=production
export SECRET_KEY="a-genuinely-long-random-production-secret"
export DATABASE_URL="postgresql://user:pass@host:5432/dbname"
```

Genuinely important, real practice: production secrets/configuration should NEVER live in code — recap file 08 and file 10's `SECRET_KEY` warnings directly, now in their actual real-world context. Most hosting platforms provide a way to set environment variables through their dashboard/CLI, keeping secrets out of the codebase entirely.

## Docker — a genuinely important, standard packaging approach

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000
CMD ["gunicorn", "--workers", "4", "--bind", "0.0.0.0:8000", "run:app"]
```

```bash
docker build -t my-ml-app .
docker run -p 8000:8000 my-ml-app
```

Genuinely worth understanding WHY Docker matters, not just how to write a Dockerfile: it packages the app, its EXACT Python version, and ALL its dependencies (recap file 01's `requirements.txt` discipline) into one PORTABLE, reproducible unit — "works on my machine" problems (recap the general dependency-conflict concerns from file 01's virtual environment discussion) genuinely disappear, since the container runs identically regardless of the underlying host machine. Genuinely worth knowing this is a real, standard practice covered in much more depth in the upcoming MLOps folder — this file introduces just enough to actually deploy.

## Choosing a hosting platform — the honest, practical landscape

```
Render / Railway -- genuinely the easiest, most beginner-friendly options,
  often with free tiers, connect a GitHub repo and deploy with minimal config
Heroku -- historically the standard beginner choice, still usable though the
  free tier has genuinely changed significantly over the years
AWS / GCP / Azure -- genuinely more powerful and flexible, but a real, steeper
  learning curve -- covered properly in the upcoming Cloud folder
PythonAnywhere -- a genuinely simple, Flask-friendly option for smaller projects
```

Genuinely the practical, honest recommendation for a portfolio project at this stage: Render or Railway — both let a GitHub repo (recap this entire roadmap's Git workflow) be connected directly, automatically detecting a `requirements.txt` and Flask app, deploying with genuinely minimal manual configuration. AWS/GCP genuinely matter more once the Cloud folder is reached, and represent real, industry-standard skills, but add real complexity that isn't necessary just to get a demo online.

## A genuinely complete deployment checklist

```
[ ] debug=False in production (or genuinely never set at all -- Gunicorn doesn't use it)
[ ] SECRET_KEY set via environment variable, genuinely random and never committed to git
[ ] Database connection uses environment variables (file 08/09), not hardcoded credentials
[ ] requirements.txt is complete and up to date (pip freeze > requirements.txt)
[ ] .gitignore excludes venv/, __pycache__/, .env files, and any local database files
[ ] CORS configured correctly if a separate frontend needs to reach the API (file 07)
[ ] Error handlers return clean, non-revealing error messages (recap file 06's
    "don't leak internal error details" caution)
[ ] A /health endpoint exists (recap file 06/07) for monitoring
```

## Logging in production — genuinely important for debugging issues after deployment

```python
import logging

logging.basicConfig(
    filename='app.log',
    level=logging.INFO,
    format='%(asctime)s %(levelname)s: %(message)s'
)

@app.route('/api/v1/predict', methods=['POST'])
def predict():
    app.logger.info(f"Prediction request received")
    try:
        # ... prediction logic ...
        app.logger.info(f"Prediction successful")
    except Exception as e:
        app.logger.error(f"Prediction failed: {e}")
        return jsonify({"error": "Internal error"}), 500
```

Genuinely important, real practical necessity — once an app is deployed, there's genuinely no `debug=True` traceback to look at directly anymore (correctly, for security reasons per this file's earlier warning) — LOGS become the primary way to understand what's actually happening/failing in production, a genuinely essential habit for any real deployed application.

## Environment-specific settings — recap file 08's config classes, now fully relevant

```python
class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')      # recap file 09, now a REAL production database,
                                                                       # not the local SQLite file used in development
```

Genuinely worth knowing that production deployments typically use a genuinely more robust database than SQLite (recap SQL file 01's SQLite-for-practice framing) — PostgreSQL is genuinely the standard choice for production Flask apps, and most hosting platforms provide it as a managed add-on.

## A minimal, complete production-ready `Procfile`/startup config

```
# Procfile (used by many hosting platforms)
web: gunicorn run:app
```

Genuinely worth knowing this tiny file is often ALL that's needed for platforms like Render/Railway to know how to actually start the app in production — connecting directly to the Gunicorn command covered earlier in this file.

## Try this

```python
# Take the complete Flask app built across files 08-10's exercises (blueprints,
# database, authentication) and prepare it for real deployment:
# 1. Move SECRET_KEY and database URL to environment variables
# 2. Write a complete requirements.txt
# 3. Write a Dockerfile and successfully build + run the container locally
#    (docker build, docker run, confirm the app works at localhost:8000)
# 4. Push the project to a GitHub repo (recap this whole roadmap's git workflow)
#    and deploy it to Render or Railway's free tier, connecting the repo directly
# Confirm the deployed, PUBLIC url actually works -- make a real prediction
# request against it from a completely different device/network to confirm
# it's genuinely publicly accessible, not just working locally
```

Actually getting a real, public URL working end-to-end is genuinely the single most rewarding milestone in this entire Flask folder — a live, demoable ML application, deployable in any interview or portfolio conversation from here forward.

## What's next
File 12 — the final Flask → Production ML Serving Bridge, pulling every concept from files 01-11 together and connecting them to the more specialized, production-grade ML serving tools (FastAPI, dedicated model-serving frameworks) covered properly in the upcoming MLOps folder.
