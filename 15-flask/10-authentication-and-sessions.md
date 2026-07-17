# 10 — Authentication & Sessions

Recap file 09's closing note — adding user accounts and access control, genuinely necessary once an app needs to distinguish between different users or protect certain endpoints from public access.

## Sessions — how Flask remembers a user across requests

```python
# HTTP is genuinely STATELESS -- each request is independent, the server has
# NO built-in memory of previous requests from the same browser
# Sessions solve this: the server sends the browser a SIGNED COOKIE containing
# a session identifier, and the browser sends it back on every subsequent
# request, letting the server "remember" that user across multiple requests
```

```python
from flask import Flask, session

app.config['SECRET_KEY'] = 'a-genuinely-random-secret-key'      # recap file 08's config management --
                                                                     # NEVER hardcode this in real production code

@app.route('/set-session')
def set_session():
    session['username'] = 'soham'
    return "Session set"

@app.route('/get-session')
def get_session():
    username = session.get('username', 'Guest')
    return f"Hello, {username}"
```

`SECRET_KEY` is genuinely what SIGNS the session cookie cryptographically — this prevents a user from tampering with their own session data (e.g. editing a cookie to claim `is_admin=True`) since any tampering would invalidate the cryptographic signature and Flask would reject it. Genuinely important, real security fundamental: a weak or exposed `SECRET_KEY` genuinely compromises this entire protection.

## Password hashing — genuinely non-negotiable, never store plain text passwords

```python
from werkzeug.security import generate_password_hash, check_password_hash

hashed = generate_password_hash('user_password_123')
print(hashed)   # genuinely a long, irreversible hash string, NOT the original password

is_correct = check_password_hash(hashed, 'user_password_123')      # True
is_correct_wrong = check_password_hash(hashed, 'wrong_password')     # False
```

Genuinely one of the most important security principles in this entire folder, worth stating with real emphasis: passwords must NEVER be stored in plain text, EVER. Hashing is a ONE-WAY transformation (recap the general "irreversible" property, conceptually related to hash functions from DSA file 06 — genuinely the same core cryptographic idea, applied here specifically for password security) — even if a database were compromised, hashed passwords genuinely can't be reversed back into the original password. `check_password_hash` works by hashing the ATTEMPTED password and comparing hashes, never by "decrypting" the stored hash.

## A complete user registration/login flow

```python
class User(db.Model):      # recap file 09's SQLAlchemy model pattern
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(200), nullable=False)

@app.route('/register', methods=['POST'])
def register():
    username = request.form.get('username')
    password = request.form.get('password')

    if User.query.filter_by(username=username).first():      # recap file 09's .query.filter_by()
        return "Username already taken", 400

    new_user = User(username=username, password_hash=generate_password_hash(password))
    db.session.add(new_user)
    db.session.commit()
    return "Registered successfully", 201

@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    password = request.form.get('password')

    user = User.query.filter_by(username=username).first()
    if user and check_password_hash(user.password_hash, password):
        session['user_id'] = user.id      # genuinely store the ID, not sensitive data, in the session
        return "Logged in successfully"

    return "Invalid username or password", 401
```

Genuinely important, deliberate practice worth flagging: the login error message is INTENTIONALLY the SAME whether the username doesn't exist OR the password is wrong — revealing WHICH one was incorrect would let an attacker confirm whether a specific username exists in the system, a real, small but genuine security consideration.

## Protecting routes — requiring login

```python
from functools import wraps
from flask import redirect, url_for

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/dashboard')
@login_required
def dashboard():
    return "Welcome to your dashboard!"
```

Genuinely worth recognizing this as a real, practical application of Python decorators (recap the general decorator pattern from file 01's `@app.route`) — `login_required` is a CUSTOM decorator, following the same underlying mechanism, that wraps any route to check for a valid session BEFORE letting the actual route logic run.

## Logout — clearing the session

```python
@app.route('/logout')
def logout():
    session.pop('user_id', None)
    return redirect(url_for('login'))
```

## Flask-Login — the genuinely standard extension for real applications

```python
from flask_login import LoginManager, UserMixin, login_user, login_required, current_user

login_manager = LoginManager(app)

class User(UserMixin, db.Model):      # UserMixin adds the methods Flask-Login expects
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    password_hash = db.Column(db.String(200))

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

@app.route('/login', methods=['POST'])
def login():
    user = User.query.filter_by(username=request.form.get('username')).first()
    if user and check_password_hash(user.password_hash, request.form.get('password')):
        login_user(user)      # handles session management internally
        return redirect(url_for('dashboard'))
    return "Invalid credentials", 401

@app.route('/dashboard')
@login_required      # Flask-Login's OWN decorator, genuinely handles the same thing manually built above
def dashboard():
    return f"Welcome, {current_user.username}!"
```

Genuinely worth knowing `flask-login` exists as the STANDARD, well-tested extension for real applications rather than hand-rolling session logic from scratch — it handles a genuinely large number of real, subtle edge cases (remember-me cookies, session refresh, "next page" redirects after login) correctly, that a manual implementation would likely miss.

## API authentication — API keys, a genuinely different pattern from session-based login

```python
API_KEYS = {"abc123xyz": "client_name_1"}      # in a real app, genuinely stored in a database, hashed

def require_api_key(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        api_key = request.headers.get('X-API-Key')
        if api_key not in API_KEYS:
            return jsonify({"error": "Invalid or missing API key"}), 401
        return f(*args, **kwargs)
    return decorated_function

@api_bp.route('/predict', methods=['POST'])
@require_api_key
def predict():
    ...
```

Genuinely an important, different authentication pattern from browser-based session login — API clients (recap file 07's REST API discussion) typically authenticate via an API KEY sent in a request HEADER, rather than cookies/sessions, since API clients are often NOT browsers and don't naturally handle cookies the same way. Worth knowing this is the standard pattern for machine-to-machine API access, distinct from human-facing login.

## Try this

```python
# Add a User model and complete registration/login/logout flow to the Flask app
# from file 09's exercise, using werkzeug's password hashing (or flask-login
# for the more complete, standard approach)
# Protect the /api/v1/history endpoint from file 09 with @login_required,
# and confirm accessing it without logging in redirects appropriately
# Then build a SEPARATE API-key-protected version of the /api/v1/predict
# endpoint, testing it with curl -H "X-API-Key: ..." (recap file 07's curl usage)
# and confirming requests WITHOUT a valid key are correctly rejected with 401
```

Testing both authentication styles (session-based for the human-facing dashboard, API-key-based for the machine-facing prediction endpoint) is genuinely realistic — most real ML products need BOTH patterns simultaneously, serving genuinely different kinds of clients.

## What's next
File 11 — Deployment, taking this entire app out of local development and onto a real, publicly-accessible server — the actual final step in turning this whole folder's work into something genuinely shareable.
