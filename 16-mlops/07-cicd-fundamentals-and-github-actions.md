# 07 — CI/CD Fundamentals & GitHub Actions

Recap file 06's closing note — shifting into AUTOMATION: automating the testing and deployment processes that have been done manually throughout this entire roadmap so far (recap this whole repo's `git add`/`commit`/`push` workflow, and Flask file 11's manual deployment steps).

## CI vs CD — genuinely two related but distinct concepts

```
CI (Continuous Integration): automatically TEST code every time it changes
  (recap Flask file 08's pytest tests directly) -- catches bugs EARLY, before
  they reach production

CD (Continuous Deployment/Delivery): automatically DEPLOY code that passes
  those tests -- recap Flask file 11's manual deployment steps, now automated
```

Genuinely worth being precise about the distinction — CI is about VERIFYING code is correct; CD is about SHIPPING verified code. Many teams do CI without full CD (automated testing, but a human still clicks "deploy"), and that's a genuinely reasonable, common middle ground.

## Why this matters MORE for ML projects — recap file 01's three-way risk directly

```
Recap file 01: code, DATA, and MODEL can all independently change and break things
CI/CD for ML genuinely needs to test:
1. Does the CODE still work? (standard software testing, recap Flask file 08's pytest)
2. Does the MODEL still meet a MINIMUM performance bar? (genuinely ML-specific)
3. Does the DATA pipeline still produce valid, expected output? (genuinely ML-specific)
```

## GitHub Actions — genuinely the most accessible CI/CD tool for a GitHub-based workflow

```yaml
# .github/workflows/test.yml
name: Run Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # genuinely checks out the repo's code first

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt      # recap Flask file 01's requirements.txt discipline

      - name: Run tests
        run: pytest tests/      # recap Flask file 08's pytest test suite directly
```

Genuinely worth understanding this file's structure explicitly: `on:` defines WHAT TRIGGERS this workflow (here: every push or pull request to `main`), `jobs:` defines WHAT ACTUALLY RUNS. `uses: actions/checkout@v4` and `actions/setup-python@v5` are genuinely pre-built, reusable ACTIONS (recap the general "don't reinvent the wheel" principle) — community-maintained building blocks for common tasks, rather than writing raw shell scripting for basic setup steps.

## Where this file lives — genuinely simple, convention-based

```
my_project/
├── .github/
│   └── workflows/
│       └── test.yml        -- GitHub automatically detects and runs ANY .yml file here
├── app.py
└── requirements.txt
```

Genuinely worth knowing this is purely CONVENTION-based — GitHub automatically discovers and runs any workflow file placed in `.github/workflows/`, no additional registration/configuration needed beyond committing the file to the repo.

## Running the SAME tests already built — recap Flask file 08's pytest suite

```python
# tests/test_api.py -- recap Flask file 08 directly, GENUINELY unchanged
def test_predict_valid_input(client):
    response = client.post('/api/v1/predict', json={'text': 'This is great!'})
    assert response.status_code == 200
```

Genuinely worth recognizing this is precisely the SAME test file already written in Flask file 08 — CI doesn't require writing NEW tests specifically for automation, it just runs the EXISTING test suite AUTOMATICALLY, on every code change, instead of relying on a human remembering to run `pytest` manually before pushing.

## Multiple jobs — linting, testing, and building in parallel

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run linter
        run: pip install flake8 && flake8 . --max-line-length=100

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest tests/

  build-docker:
    runs-on: ubuntu-latest
    needs: [lint, test]      # genuinely important -- only builds if BOTH prior jobs succeed
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t my-ml-app:${{ github.sha }} .      # recap MLOps file 02's build command directly
```

Genuinely worth understanding `needs:` explicitly — it creates a DEPENDENCY chain between jobs, directly mirroring DSA file 16's topological sort/dependency-ordering concept: `build-docker` genuinely won't run at all if linting or testing FAILS first, preventing a broken image from ever being built. `${{ github.sha }}` is genuinely a useful convention — tagging the Docker image with the exact GIT COMMIT hash that produced it creates a direct, traceable link between a specific code version and its corresponding image, directly connecting to file 06's reproducibility discipline.

## Secrets in GitHub Actions — recap the general credential-safety habit directly

```yaml
steps:
  - name: Deploy
    env:
      SECRET_KEY: ${{ secrets.SECRET_KEY }}      # genuinely pulled from GitHub's encrypted secrets storage,
                                                       # NEVER hardcoded in the workflow file itself
    run: ./deploy.sh
```

Genuinely important, real practice — recap Flask files 08/10/11's "never hardcode secrets" principle directly: GitHub Actions has its OWN encrypted secrets storage (configured in the repo's Settings), and `${{ secrets.NAME }}` references them safely — the actual secret VALUE never appears in the workflow file or logs.

## Caching dependencies — genuinely important for faster CI runs

```yaml
- name: Cache pip dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
```

Genuinely the SAME underlying idea as MLOps file 02's Docker layer caching, and file 06's `dvc repro` change-detection — avoid REDOING expensive work (here, reinstalling all Python packages) when the relevant inputs (`requirements.txt`) haven't actually changed. This exact "skip unchanged work" optimization pattern has now appeared in Docker, DVC, and GitHub Actions — genuinely a recurring, fundamental idea across nearly every tool in this whole folder.

## Status badges — a genuinely nice, real practice for a portfolio repo

```markdown
![Tests](https://github.com/username/repo/actions/workflows/test.yml/badge.svg)
```

Genuinely worth adding to this whole ml-notes repo's main README, or any project repo — a visible, live badge showing whether the latest code passes its automated tests is a small but genuinely real signal of engineering discipline, exactly the kind of detail that stands out favorably in a portfolio being reviewed by an interviewer.

## Try this

```yaml
# Take the Flask app with its pytest test suite (recap Flask file 08's exercise)
# and add a GitHub Actions workflow that:
# 1. Checks out the code and sets up Python
# 2. Installs dependencies (with caching)
# 3. Runs the full pytest suite
# 4. Only if tests pass, builds a Docker image (recap MLOps file 02) tagged
#    with the commit SHA
# Push this workflow file to a real GitHub repo and confirm it runs automatically,
# visible in the repo's "Actions" tab
# Then DELIBERATELY break one test (change an assertion to something false),
# push again, and confirm the workflow correctly FAILS and the Docker build
# job is correctly SKIPPED due to the needs: dependency
```

Actually watching a deliberately broken test correctly fail the CI pipeline (and block the downstream build job) is genuinely the clearest, most concrete proof that this automation is working as a real safety net, not just theoretical infrastructure.

## What's next
File 08 — CI/CD for ML Pipelines, extending this file's general CI/CD foundation with genuinely ML-SPECIFIC automation: automated model evaluation gates, data validation checks, and triggering retraining pipelines.
