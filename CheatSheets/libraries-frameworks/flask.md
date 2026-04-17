# Flask: A Complete Progressive Tutorial

---

## 1. What & Why

Flask is a micro web framework for Python. "Micro" means Flask has a small, well-defined core and deliberately leaves decisions like database choice, authentication method, and project structure to you. It provides routing, request/response handling, and templating — the minimum required to build a web application — without imposing opinions on everything else.

Flask's philosophy is that each project's needs differ, and a framework should serve those needs rather than the other way around. This makes Flask excellent for REST APIs, microservices, prototypes, and any project where you want precise control over your stack. Django gives you everything built-in at the cost of having to work within its conventions. Flask gives you a solid foundation and lets you compose the rest yourself.

You should understand Flask because it is the most common Python web framework for APIs in the industry alongside FastAPI, and understanding how it works makes you a better developer even when using higher-level frameworks.

---

## 2. Mental Model

Flask is a WSGI application — a Python callable that accepts HTTP requests and returns HTTP responses. The web server (Gunicorn, uWSGI) calls your Flask app with each request. Flask's router matches the URL to a view function. The view function runs, accesses request data, builds a response, and returns it.

```
HTTP Request arrives
        │
        ▼
WSGI Server (Gunicorn)
        │ calls Flask app
        ▼
Flask Application
  ├── URL Router: matches path → view function
  ├── Before-request hooks run (auth, logging)
  ├── View function executes
  │     ├── Reads from: request (headers, body, args)
  │     ├── Reads from: session, g (request-scoped storage)
  │     └── Returns: Response (JSON, HTML, or file)
  └── After-request hooks run (CORS headers, logging)
        │
        ▼
HTTP Response sent back
```

Context locals (`request`, `session`, `g`, `current_app`) are thread-local or coroutine-local — safe to use without passing them explicitly even in concurrent environments because Flask ensures each request gets its own copy.

---

## 3. Progressive Examples

### Level 1: Minimal Flask App and Routing

```python
# app.py
from flask import Flask, request, jsonify

app = Flask(__name__)   # __name__ tells Flask where to find resources

# Route: maps URL path to a Python function
@app.route("/")
def index():
    return "Hello, World!"   # Flask wraps string in a 200 OK Response

# Dynamic URL segments
@app.route("/users/<int:user_id>")   # <type:variable_name>
def get_user(user_id):
    return jsonify({"user_id": user_id, "name": "Alice"})

# Multiple HTTP methods on one route
@app.route("/items", methods=["GET", "POST"])
def items():
    if request.method == "GET":
        return jsonify({"items": ["apple", "banana"]})
    if request.method == "POST":
        data = request.get_json()
        return jsonify({"created": data}), 201   # 201 Created

# URL converters: <string:x> (default), <int:x>, <float:x>, <path:x>, <uuid:x>
@app.route("/files/<path:filename>")  # path: allows slashes in the value
def serve_file(filename):
    return f"Serving: {filename}"

# Run the development server
if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)
    # debug=True: auto-reload on code changes, detailed error pages
    # NEVER use debug=True in production
```

```bash
# Install and run
pip install flask
flask run                         # uses FLASK_APP env var
flask --app app.py run            # explicit
flask --app app.py run --debug    # debug mode
FLASK_APP=app.py FLASK_DEBUG=1 flask run  # environment variables
```

### Level 2: Request, Response, and Blueprint Structure

```python
from flask import Flask, request, jsonify, abort, make_response, url_for
from flask import Blueprint

# --- Accessing request data ---

@app.route("/search")
def search():
    # Query parameters: /search?q=flask&page=2
    query = request.args.get("q", "")          # get with default
    page = request.args.get("page", 1, type=int)  # type conversion
    all_args = request.args.to_dict()

    return jsonify({"query": query, "page": page})

@app.route("/register", methods=["POST"])
def register():
    # JSON body
    data = request.get_json()
    if data is None:
        abort(400, description="Expected JSON body")

    name = data.get("name")
    email = data.get("email")

    # Form data (Content-Type: application/x-www-form-urlencoded)
    # name = request.form.get("name")

    # Headers
    auth_header = request.headers.get("Authorization")
    user_agent = request.headers.get("User-Agent")

    # Files
    # uploaded = request.files.get("photo")
    # uploaded.save("/uploads/" + uploaded.filename)

    return jsonify({"registered": name, "email": email}), 201

# --- Building responses ---

@app.route("/download")
def download():
    response = make_response("file content here")
    response.headers["Content-Disposition"] = "attachment; filename=data.txt"
    response.headers["Content-Type"] = "text/plain"
    return response

# Returning tuples: (body, status_code) or (body, status, headers)
@app.route("/created")
def created():
    return jsonify({"id": 42}), 201

@app.route("/redirect-demo")
def redirect_demo():
    from flask import redirect
    return redirect(url_for("index"))   # url_for builds URL from function name

# --- Blueprints: modular application structure ---
# users.py
users_bp = Blueprint("users", __name__, url_prefix="/users")

@users_bp.route("/")
def list_users():
    return jsonify({"users": []})

@users_bp.route("/<int:user_id>")
def get_user(user_id):
    return jsonify({"id": user_id})

@users_bp.route("/", methods=["POST"])
def create_user():
    data = request.get_json()
    return jsonify({"created": data["name"]}), 201

# Register blueprint in app factory
def create_app():
    app = Flask(__name__)
    app.register_blueprint(users_bp)
    # Now: /users/, /users/<id>
    return app
```

### Level 3: Error Handling, Before/After Request Hooks

```python
from flask import Flask, request, jsonify, g
import time
import logging

app = Flask(__name__)

# --- Error handlers ---

@app.errorhandler(400)
def bad_request(error):
    return jsonify({
        "error": "Bad Request",
        "message": str(error.description)
    }), 400

@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Not Found", "path": request.path}), 404

@app.errorhandler(500)
def server_error(error):
    # Log the full traceback for 500 errors
    app.logger.error(f"Server error: {error}")
    return jsonify({"error": "Internal Server Error"}), 500

# Custom exception class
class APIError(Exception):
    def __init__(self, message, status_code=400):
        self.message = message
        self.status_code = status_code

@app.errorhandler(APIError)
def handle_api_error(error):
    return jsonify({"error": error.message}), error.status_code

@app.route("/risky")
def risky():
    if some_condition:
        raise APIError("Validation failed: email required", 422)
    return jsonify({"ok": True})

# --- Before/After request hooks ---

@app.before_request
def start_timer():
    g.start_time = time.time()     # g is request-scoped storage

@app.before_request
def authenticate():
    # Skip auth for public routes
    if request.endpoint in ("health", "login"):
        return
    token = request.headers.get("Authorization", "").removeprefix("Bearer ")
    if not token or not validate_token(token):
        return jsonify({"error": "Unauthorized"}), 401
    g.current_user = get_user_from_token(token)

@app.after_request
def add_timing_header(response):
    elapsed = time.time() - g.start_time
    response.headers["X-Response-Time"] = f"{elapsed:.4f}s"
    return response

@app.after_request
def add_cors_headers(response):
    response.headers["Access-Control-Allow-Origin"] = "*"
    response.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE, OPTIONS"
    response.headers["Access-Control-Allow-Headers"] = "Content-Type, Authorization"
    return response

@app.teardown_appcontext
def close_db(error):
    """Runs after each request, even on error. Close connections here."""
    db = g.pop("db", None)
    if db is not None:
        db.close()
```

### Level 4: Database Integration with SQLAlchemy

```python
from flask import Flask, request, jsonify, g
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import DeclarativeBase
import os

class Base(DeclarativeBase):
    pass

db = SQLAlchemy(model_class=Base)

def create_app():
    app = Flask(__name__)
    app.config["SQLALCHEMY_DATABASE_URI"] = os.environ.get(
        "DATABASE_URL", "sqlite:///dev.db"
    )
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
    db.init_app(app)
    return app

app = create_app()

# --- Models ---

class User(db.Model):
    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(255), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, server_default=db.func.now())

    posts = db.relationship("Post", back_populates="author", lazy="dynamic")

    def to_dict(self):
        return {"id": self.id, "name": self.name, "email": self.email}

class Post(db.Model):
    __tablename__ = "posts"

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    body = db.Column(db.Text)
    user_id = db.Column(db.Integer, db.ForeignKey("users.id"), nullable=False)

    author = db.relationship("User", back_populates="posts")

# --- CRUD routes ---

@app.route("/users", methods=["GET"])
def list_users():
    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 20, type=int)

    pagination = db.paginate(
        db.select(User).order_by(User.created_at.desc()),
        page=page, per_page=per_page
    )
    return jsonify({
        "users": [u.to_dict() for u in pagination.items],
        "total": pagination.total,
        "pages": pagination.pages,
        "page": page,
    })

@app.route("/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = db.get_or_404(User, user_id)   # raises 404 if not found
    return jsonify(user.to_dict())

@app.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if not data or "email" not in data or "name" not in data:
        return jsonify({"error": "name and email required"}), 400

    if db.session.execute(db.select(User).filter_by(email=data["email"])).scalar():
        return jsonify({"error": "Email already exists"}), 409

    user = User(name=data["name"], email=data["email"])
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict()), 201

@app.route("/users/<int:user_id>", methods=["PUT"])
def update_user(user_id):
    user = db.get_or_404(User, user_id)
    data = request.get_json()
    if "name" in data:
        user.name = data["name"]
    if "email" in data:
        user.email = data["email"]
    db.session.commit()
    return jsonify(user.to_dict())

@app.route("/users/<int:user_id>", methods=["DELETE"])
def delete_user(user_id):
    user = db.get_or_404(User, user_id)
    db.session.delete(user)
    db.session.commit()
    return "", 204   # 204 No Content

# Create tables
with app.app_context():
    db.create_all()
```

### Level 5: Configuration, Testing, and Production Setup

```python
# config.py — environment-based configuration
import os

class Config:
    SECRET_KEY = os.environ.get("SECRET_KEY") or "dev-only-secret"
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///dev.db"

class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ["DATABASE_URL"]

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"

config = {
    "development": DevelopmentConfig,
    "production": ProductionConfig,
    "testing": TestingConfig,
}

# app.py — application factory pattern
def create_app(config_name=None):
    app = Flask(__name__)
    config_name = config_name or os.environ.get("FLASK_ENV", "development")
    app.config.from_object(config[config_name])

    db.init_app(app)

    from .users import users_bp
    from .auth import auth_bp
    app.register_blueprint(users_bp)
    app.register_blueprint(auth_bp)

    return app

# --- Testing with pytest ---
import pytest

@pytest.fixture
def app():
    app = create_app("testing")
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

def test_create_user(client):
    response = client.post("/users", json={"name": "Alice", "email": "alice@test.com"})
    assert response.status_code == 201
    data = response.get_json()
    assert data["name"] == "Alice"

def test_duplicate_email(client):
    client.post("/users", json={"name": "Alice", "email": "alice@test.com"})
    response = client.post("/users", json={"name": "Bob", "email": "alice@test.com"})
    assert response.status_code == 409

def test_get_nonexistent_user(client):
    response = client.get("/users/9999")
    assert response.status_code == 404

# --- Production with Gunicorn ---
# gunicorn.conf.py
workers = 4                   # 2 × CPU cores + 1
worker_class = "sync"         # or "gevent" for async I/O
bind = "0.0.0.0:8000"
timeout = 30
keepalive = 5
accesslog = "-"               # stdout
errorlog = "-"

# Run:
# gunicorn --config gunicorn.conf.py "app:create_app()"
```

### Level 6: Authentication, Caching, and Rate Limiting

```python
import jwt
import time
import functools
from flask import Flask, request, jsonify, g, current_app
from flask_caching import Cache
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
app.config["SECRET_KEY"] = "production-secret-from-env"
cache = Cache(app, config={"CACHE_TYPE": "SimpleCache"})
limiter = Limiter(get_remote_address, app=app, default_limits=["200/day", "50/hour"])

# --- JWT Authentication ---

def create_token(user_id: int) -> str:
    payload = {
        "user_id": user_id,
        "exp": time.time() + 3600,    # expires in 1 hour
        "iat": time.time(),
    }
    return jwt.encode(payload, current_app.config["SECRET_KEY"], algorithm="HS256")

def require_auth(f):
    """Decorator that validates JWT and injects user_id into request context."""
    @functools.wraps(f)
    def decorated(*args, **kwargs):
        auth = request.headers.get("Authorization", "")
        if not auth.startswith("Bearer "):
            return jsonify({"error": "Missing or invalid token"}), 401
        token = auth[7:]
        try:
            payload = jwt.decode(token, current_app.config["SECRET_KEY"], algorithms=["HS256"])
            g.user_id = payload["user_id"]
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route("/auth/login", methods=["POST"])
@limiter.limit("10/minute")    # rate limit login attempts
def login():
    data = request.get_json()
    user = verify_credentials(data.get("email"), data.get("password"))
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401
    token = create_token(user.id)
    return jsonify({"token": token, "user_id": user.id})

@app.route("/profile")
@require_auth
def profile():
    return jsonify({"user_id": g.user_id})

# --- Caching ---

@app.route("/stats")
@cache.cached(timeout=300)    # cache this response for 5 minutes
def get_stats():
    # Expensive database query only runs once per 5 minutes
    return jsonify({"total_users": User.query.count()})

@app.route("/users/<int:user_id>/followers")
@cache.memoize(timeout=60)    # cache per-argument: separate cache for each user_id
def get_followers(user_id):
    followers = compute_followers(user_id)
    return jsonify({"count": len(followers)})

def update_user(user_id, data):
    # Clear cache when data changes
    cache.delete_memoized(get_followers, user_id)
    # Update in database...
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using the development server in production**

```bash
# WRONG: never run this in production
flask run          # development server, single-threaded, no stability guarantees

# CORRECT: use a production WSGI server
pip install gunicorn
gunicorn -w 4 "myapp:create_app()"

# Or with uWSGI:
uwsgi --http :8000 --module myapp:app --processes 4
```

**Mistake 2: Using `app.config["SECRET_KEY"]` with a hardcoded value**

```python
# WRONG: secret in source code means anyone who reads the code can forge sessions
app.config["SECRET_KEY"] = "mysecretkey"

# CORRECT: read from environment
import os
app.config["SECRET_KEY"] = os.environ.get("SECRET_KEY") or abort(500, "SECRET_KEY not set")
# In production, set SECRET_KEY to a long random value:
# python -c "import secrets; print(secrets.token_hex(32))"
```

**Mistake 3: Not using the application factory pattern**

```python
# WRONG: creating app at module level makes testing and configuration difficult
app = Flask(__name__)
db.init_app(app)
# Problem: can't create multiple app instances with different configs for testing

# CORRECT: application factory — create the app on demand
def create_app(config_name="development"):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    db.init_app(app)
    return app
# Testing: create_app("testing")   Production: create_app("production")
```

**Mistake 4: Accessing `request` or `g` outside request context**

```python
# WRONG: request is only available inside a request context
def utility_function():
    user_data = request.get_json()   # RuntimeError if called outside a request

# CORRECT: pass data explicitly
def utility_function(data):
    process(data)

@app.route("/process")
def process_route():
    data = request.get_json()
    result = utility_function(data)   # pass data explicitly
    return jsonify(result)
```

**Mistake 5: Not handling database session errors**

```python
# WRONG: no error handling — transaction left open on failure
@app.route("/transfer", methods=["POST"])
def transfer():
    from_account = Account.query.get(request.json["from_id"])
    from_account.balance -= request.json["amount"]
    db.session.commit()    # if this fails, we've deducted without crediting!

# CORRECT: wrap related operations in a transaction
@app.route("/transfer", methods=["POST"])
def transfer():
    try:
        from_account = db.get_or_404(Account, request.json["from_id"])
        to_account = db.get_or_404(Account, request.json["to_id"])
        amount = request.json["amount"]

        if from_account.balance < amount:
            return jsonify({"error": "Insufficient funds"}), 400

        from_account.balance -= amount
        to_account.balance += amount
        db.session.commit()       # both changes committed atomically
        return jsonify({"status": "ok"})
    except Exception as e:
        db.session.rollback()     # undo ALL changes if anything fails
        app.logger.error(f"Transfer failed: {e}")
        return jsonify({"error": "Transfer failed"}), 500
```

---

## 5. The "Why Does This Work" Layer

### How Flask's Context Locals Work

Flask uses thread-local (or for async: coroutine-local) storage to make `request`, `g`, `session`, and `current_app` available globally without passing them as function arguments. When Gunicorn handles multiple concurrent requests using multiple threads, each thread gets its own copy of these proxies via `werkzeug.local.LocalProxy`.

When you access `request.method`, the proxy looks up the actual request object for the current thread from a stack that Flask maintains. At the start of each request, Flask pushes a request context onto this stack. At the end, it pops it and triggers teardown functions.

This is why you get `RuntimeError: Working outside of application context` when you access `current_app` or `db` outside of a request or application context — there's nothing on the stack to look up.

### Why the Application Factory Pattern Matters

Creating the Flask app at module level (`app = Flask(__name__)`) means the moment you import the module, the app is created with whatever configuration is currently in the environment. You can't create a test version with an in-memory database — it's already created.

The factory pattern (`create_app(config)`) defers creation until you call the function. Tests can call `create_app("testing")` and get an isolated instance with an SQLite in-memory database. The production server calls `create_app("production")` and gets the production configuration. The same codebase serves both without any conditional logic in route handlers.

---

## 6. Quick Reference

### Application Structure

```
myapp/
├── __init__.py          # create_app() factory
├── config.py            # configuration classes
├── models.py            # SQLAlchemy models
├── extensions.py        # db = SQLAlchemy(), cache = Cache()
├── users/
│   ├── __init__.py
│   ├── routes.py        # Blueprint definition
│   └── schemas.py       # validation/serialization
├── auth/
│   ├── routes.py
│   └── utils.py
└── tests/
    ├── conftest.py      # pytest fixtures
    ├── test_users.py
    └── test_auth.py
```

### Essential Patterns

```python
# Minimal JSON API route
@app.route("/resource/<int:id>", methods=["GET"])
def get_resource(id):
    item = db.get_or_404(Resource, id)
    return jsonify(item.to_dict())

# Create with validation
@app.route("/resource", methods=["POST"])
def create_resource():
    data = request.get_json()
    if not data or "name" not in data:
        return jsonify({"error": "name required"}), 400
    item = Resource(**data)
    db.session.add(item)
    db.session.commit()
    return jsonify(item.to_dict()), 201

# Auth decorator usage
@app.route("/protected")
@require_auth
def protected():
    return jsonify({"user_id": g.user_id})
```

### Common Extensions

| Extension | Install | Use |
|-----------|---------|-----|
| Flask-SQLAlchemy | `pip install flask-sqlalchemy` | Database ORM |
| Flask-Migrate | `pip install flask-migrate` | DB migrations |
| Flask-Login | `pip install flask-login` | Session auth |
| Flask-JWT-Extended | `pip install flask-jwt-extended` | JWT auth |
| Flask-Caching | `pip install flask-caching` | Response cache |
| Flask-Limiter | `pip install flask-limiter` | Rate limiting |
| Flask-CORS | `pip install flask-cors` | CORS headers |
| Flask-WTF | `pip install flask-wtf` | Form handling |
| Marshmallow | `pip install marshmallow` | Serialization |
