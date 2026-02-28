# Flask Complete Reference Guide
> From Zero to Expert — Everything You Need to Master Flask

---

## 1. What is Flask & How It Works

Flask is a **micro web framework** for Python. "Micro" means it has a small core and leaves the rest to extensions — it doesn't force an ORM, form library, or project structure on you.

### Flask vs Django

| | Flask | Django |
|--|-------|--------|
| Size | Micro — minimal core | Batteries-included |
| ORM | Your choice (SQLAlchemy, etc.) | Built-in Django ORM |
| Admin | Extension needed | Built-in |
| Forms | Extension needed | Built-in |
| Learning curve | Gentler | Steeper |
| Flexibility | Very high | Opinionated |
| Best for | APIs, microservices, custom apps | Full-featured web apps |

### How a Request Flows

```
Browser/Client
     │  HTTP Request (GET /users/42)
     ▼
WSGI Server (Gunicorn/uWSGI)
     │
     ▼
Flask App (WSGI Application)
     │
     ├─► URL Router  →  finds view function for /users/<id>
     │
     ├─► Before-request hooks run
     │
     ├─► View Function executes
     │        │
     │        └─► Accesses: request, session, g
     │            Queries: Database
     │            Renders: Template / builds JSON
     │            Returns: Response object
     │
     ├─► After-request hooks run
     │
     └─► HTTP Response sent back to client
```

### Core Objects

| Object | What It Is |
|--------|-----------|
| `Flask` | The application object |
| `request` | The incoming HTTP request (thread/context local) |
| `response` | What you return from a view |
| `session` | Cookie-based client-side session dict |
| `g` | Request-scoped global storage |
| `current_app` | Proxy to the active application |
| `url_for` | Builds URLs from endpoint names |
| `render_template` | Renders a Jinja2 template |

---

## 2. Installation & Project Setup

### Install Flask

```bash
# Create and activate a virtual environment first (always!)
python3 -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows

# Install Flask
pip install flask

# Install common extensions
pip install flask-sqlalchemy flask-migrate flask-login
pip install flask-wtf flask-mail flask-caching
pip install python-dotenv        # For .env files
```

### Recommended Project Structure

```
myapp/
├── app/                        ← Application package
│   ├── __init__.py             ← App factory (create_app)
│   ├── models.py               ← Database models
│   ├── extensions.py           ← Extension instances
│   ├── auth/                   ← Auth blueprint
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   └── forms.py
│   ├── main/                   ← Main blueprint
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── api/                    ← API blueprint
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── templates/              ← Jinja2 templates
│   │   ├── base.html
│   │   ├── auth/
│   │   │   ├── login.html
│   │   │   └── register.html
│   │   └── main/
│   │       └── index.html
│   └── static/                 ← CSS, JS, images
│       ├── css/
│       ├── js/
│       └── img/
├── migrations/                 ← Flask-Migrate files
├── tests/                      ← Test suite
│   ├── conftest.py
│   └── test_routes.py
├── .env                        ← Environment variables (never commit!)
├── .env.example                ← Template for .env
├── .gitignore
├── config.py                   ← Configuration classes
├── requirements.txt
└── wsgi.py                     ← Entry point for production
```

### Essential Files

```python
# wsgi.py — production entry point
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run()
```

```
# .env
FLASK_APP=wsgi.py
FLASK_DEBUG=1
SECRET_KEY=your-secret-key-here-change-in-production
DATABASE_URL=sqlite:///dev.db
```

```
# .gitignore
.venv/
__pycache__/
*.pyc
.env
instance/
*.db
```

### Running the Dev Server

```bash
flask run                         # Uses FLASK_APP env var
flask run --debug                 # Debug mode with reloader
flask run --host=0.0.0.0          # Accessible on network
flask run --port=8080             # Custom port

# Or set env vars first
export FLASK_APP=wsgi.py
export FLASK_DEBUG=1
flask run
```

---

## 3. Your First Flask App

### Minimal App

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello, Flask!"

if __name__ == "__main__":
    app.run(debug=True)
```

```bash
python app.py
# or
flask --app app run --debug
```

### Slightly More Complete App

```python
from flask import Flask, render_template, request, redirect, url_for, flash

app = Flask(__name__)
app.secret_key = "dev-secret-change-in-prod"

# In-memory "database"
users = []

@app.route("/")
def index():
    return render_template("index.html", users=users)

@app.route("/add", methods=["GET", "POST"])
def add_user():
    if request.method == "POST":
        name = request.form.get("name", "").strip()
        if not name:
            flash("Name cannot be empty", "error")
        else:
            users.append(name)
            flash(f"Added {name}!", "success")
            return redirect(url_for("index"))
    return render_template("add_user.html")

@app.route("/user/<int:user_id>")
def user_detail(user_id):
    if user_id >= len(users):
        return "Not found", 404
    return f"<h1>{users[user_id]}</h1>"

if __name__ == "__main__":
    app.run(debug=True)
```

---

## 4. Routing

### Basic Routes

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Home page"

@app.route("/about")
def about():
    return "About page"

# Multiple URLs for one view
@app.route("/")
@app.route("/home")
def home():
    return "Home"
```

### URL Variables (Converters)

```python
# String (default) — accepts any text without a slash
@app.route("/user/<username>")
def user_profile(username):
    return f"Profile: {username}"

# int — accepts positive integers
@app.route("/post/<int:post_id>")
def show_post(post_id):
    return f"Post #{post_id}"

# float
@app.route("/price/<float:amount>")
def show_price(amount):
    return f"Price: ${amount:.2f}"

# path — like string but accepts slashes
@app.route("/files/<path:filepath>")
def show_file(filepath):
    return f"File: {filepath}"    # e.g., /files/docs/report.pdf

# uuid
@app.route("/item/<uuid:item_id>")
def show_item(item_id):
    return f"Item: {item_id}"

# any — one of a fixed set
@app.route("/theme/<any(light, dark, auto):theme>")
def set_theme(theme):
    return f"Theme: {theme}"
```

### HTTP Methods

```python
# Specify allowed methods
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        # Handle form submission
        return redirect(url_for("index"))
    return render_template("login.html")

# Method-specific decorators (Flask 2.0+)
@app.get("/users")
def list_users():
    return "List of users"

@app.post("/users")
def create_user():
    return "Created", 201

@app.put("/users/<int:id>")
def update_user(id):
    return "Updated"

@app.delete("/users/<int:id>")
def delete_user(id):
    return "Deleted", 204

@app.patch("/users/<int:id>")
def patch_user(id):
    return "Patched"
```

### URL Building with `url_for`

```python
from flask import url_for

# Basic
url_for("index")                  # "/"
url_for("user_profile", username="alice")   # "/user/alice"
url_for("show_post", post_id=42)  # "/post/42"

# Extra kwargs become query string
url_for("index", page=2, sort="name")  # "/?page=2&sort=name"

# External URL (full URL with domain)
url_for("index", _external=True)  # "http://localhost:5000/"

# HTTPS
url_for("index", _external=True, _scheme="https")

# In templates:
# <a href="{{ url_for('index') }}">Home</a>
# <a href="{{ url_for('user_profile', username=user.name) }}">Profile</a>
```

### Redirects and Trailing Slashes

```python
# Trailing slash behaviour:
@app.route("/projects/")   # Canonical URL with slash
def projects():            # Accessing /projects redirects to /projects/
    return "Projects"

@app.route("/about")       # No trailing slash
def about():               # Accessing /about/ returns 404
    return "About"

# Manual redirect
from flask import redirect
@app.route("/old-page")
def old_page():
    return redirect(url_for("new_page"))           # 302 by default
    return redirect(url_for("new_page"), code=301) # Permanent redirect

# Redirect to external URL
return redirect("https://example.com")
```

---

## 5. Request & Response

### The `request` Object

```python
from flask import request

@app.route("/submit", methods=["POST"])
def submit():
    # URL / path info
    request.url            # Full URL: http://example.com/submit?x=1
    request.base_url       # Without query: http://example.com/submit
    request.host_url       # http://example.com/
    request.path           # /submit
    request.full_path      # /submit?x=1
    request.host           # example.com
    request.method         # "POST"
    request.scheme         # "http" or "https"
    request.is_secure      # True if https

    # Query string (?key=value)
    request.args.get("page")             # Single value (None if missing)
    request.args.get("page", 1, type=int)  # With default and type coercion
    request.args["page"]                 # Raises KeyError if missing
    request.args.getlist("tag")          # Multi-value: ?tag=a&tag=b → ['a','b']
    dict(request.args)                   # All query params as dict

    # Form data (POST with application/x-www-form-urlencoded or multipart)
    request.form.get("username")
    request.form["username"]             # Raises KeyError if missing
    request.form.getlist("hobby")        # Multi-value checkboxes

    # JSON body (Content-Type: application/json)
    data = request.get_json()            # Returns dict or None
    data = request.get_json(force=True)  # Parse even without Content-Type header
    data = request.get_json(silent=True) # Return None instead of error on bad JSON

    # Raw body
    request.data                         # Raw bytes
    request.get_data(as_text=True)       # As string

    # Headers
    request.headers.get("Authorization")
    request.headers.get("Content-Type")
    request.headers["X-Custom"]

    # Cookies
    request.cookies.get("session_id")

    # Files
    file = request.files.get("avatar")
    file = request.files["avatar"]       # Raises if missing

    # Remote address
    request.remote_addr                  # Client IP
    request.access_route                 # List of IPs (proxies + client)
    # For real IP behind proxy:
    request.headers.get("X-Forwarded-For", request.remote_addr)

    # Content type
    request.content_type
    request.is_json                      # True if JSON content type
    request.mimetype                     # Just the mimetype, no params
```

### Building Responses

```python
from flask import make_response, jsonify, Response

# String → auto-wrapped in 200 response
return "Hello"
return "Hello", 200
return "Not Found", 404
return "Created", 201, {"Location": "/new/resource"}

# Template
return render_template("page.html"), 200

# JSON (sets Content-Type: application/json)
return jsonify({"name": "Alice", "age": 30})
return jsonify(users=[u.to_dict() for u in users])

# Full control with make_response
response = make_response(render_template("page.html"))
response.status_code = 200
response.headers["X-Custom-Header"] = "value"
response.headers["Cache-Control"] = "no-cache"
response.set_cookie("user_id", "42", httponly=True, samesite="Lax")
response.delete_cookie("old_cookie")
return response

# Streaming response
def generate():
    for i in range(100):
        yield f"data: {i}\n\n"

return Response(generate(), mimetype="text/event-stream")

# File download
from flask import send_file, send_from_directory
return send_file("report.pdf", as_attachment=True)
return send_file("report.pdf", download_name="my-report.pdf", as_attachment=True)
return send_from_directory("uploads", filename, as_attachment=True)
```

---

## 6. Templates with Jinja2

Flask uses the Jinja2 templating engine. Templates live in the `templates/` folder.

### Rendering Templates

```python
from flask import render_template

@app.route("/")
def index():
    users = User.query.all()
    return render_template(
        "index.html",
        title="Home",
        users=users,
        current_year=2024
    )
```

### Base Template (Inheritance)

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My App{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
    {% block styles %}{% endblock %}
</head>
<body>
    <nav>
        <a href="{{ url_for('main.index') }}">Home</a>
        {% if current_user.is_authenticated %}
            <a href="{{ url_for('auth.logout') }}">Logout</a>
        {% else %}
            <a href="{{ url_for('auth.login') }}">Login</a>
        {% endif %}
    </nav>

    <main>
        <!-- Flash messages -->
        {% with messages = get_flashed_messages(with_categories=True) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }}">{{ message }}</div>
                {% endfor %}
            {% endif %}
        {% endwith %}

        {% block content %}{% endblock %}
    </main>

    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
    {% block scripts %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/main/index.html -->
{% extends "base.html" %}

{% block title %}Home — {{ super() }}{% endblock %}

{% block content %}
<h1>Welcome</h1>
{% endblock %}
```

### Jinja2 Syntax

```jinja2
{# Comment — not rendered #}

{# Variables #}
{{ variable }}
{{ user.name }}
{{ user["name"] }}
{{ items[0] }}
{{ "Hello " ~ name }}          {# String concatenation #}

{# Filters #}
{{ name | upper }}
{{ name | lower }}
{{ name | title }}
{{ text | truncate(100) }}
{{ text | truncate(100, killwords=False, end="...") }}
{{ items | length }}
{{ price | round(2) }}
{{ value | default("N/A") }}
{{ value | default("N/A", boolean=True) }}   {# Also replaces falsy #}
{{ html_content | safe }}        {# Don't escape — only for trusted content! #}
{{ "<b>text</b>" | escape }}     {# Escape HTML #}
{{ items | join(", ") }}
{{ items | first }}
{{ items | last }}
{{ items | sort }}
{{ items | sort(attribute="name") }}
{{ items | groupby("category") }}
{{ items | selectattr("active") | list }}
{{ items | rejectattr("deleted") | list }}
{{ items | map(attribute="name") | list }}
{{ number | filesizeformat }}    {# 100 → "100 Bytes" #}
{{ date | datetimeformat }}      {# Requires extension #}
{{ value | int }}
{{ value | float }}
{{ value | string }}
{{ value | list }}

{# Tests — use with 'is' #}
{% if x is defined %}
{% if x is none %}
{% if x is number %}
{% if x is string %}
{% if x is iterable %}
{% if x is mapping %}
{% if x is sequence %}
{% if x is odd %}
{% if x is even %}
{% if x is divisibleby(3) %}

{# Statements: if #}
{% if user.is_admin %}
    <a href="/admin">Admin</a>
{% elif user.is_moderator %}
    <a href="/mod">Mod Panel</a>
{% else %}
    <p>Hello, {{ user.name }}</p>
{% endif %}

{# Statements: for #}
{% for user in users %}
    <li>{{ loop.index }}. {{ user.name }}</li>
{% else %}
    <p>No users found.</p>         {# Renders if list is empty #}
{% endfor %}

{# Loop variables #}
loop.index        {# 1-based index #}
loop.index0       {# 0-based index #}
loop.revindex     {# Reverse index #}
loop.first        {# True if first iteration #}
loop.last         {# True if last iteration #}
loop.length       {# Total number of items #}
loop.changed(val) {# True if val differs from last iteration #}

{# Macros — reusable template functions #}
{% macro render_field(field, label) %}
    <div class="field">
        <label>{{ label }}</label>
        {{ field }}
        {% if field.errors %}
            {% for error in field.errors %}
                <span class="error">{{ error }}</span>
            {% endfor %}
        {% endif %}
    </div>
{% endmacro %}

{# Call the macro #}
{{ render_field(form.username, "Username") }}

{# Import macro from another file #}
{% from "macros.html" import render_field %}
{% import "macros.html" as macros %}
{{ macros.render_field(form.username, "Username") }}

{# Include a sub-template #}
{% include "partials/navbar.html" %}
{% include "partials/footer.html" ignore missing %}

{# Set variables #}
{% set total = items | length %}
{% set full_name = first ~ " " ~ last %}

{# Whitespace control #}
{%- if x %} ...trimming before
{% if x -%} ...trimming after
```

### Custom Filters & Context Processors

```python
# Custom Jinja2 filter
@app.template_filter("timeago")
def timeago_filter(dt):
    """Convert datetime to '3 hours ago' style string."""
    from datetime import datetime
    diff = datetime.utcnow() - dt
    if diff.days > 365:
        return f"{diff.days // 365} years ago"
    if diff.days > 30:
        return f"{diff.days // 30} months ago"
    if diff.days > 0:
        return f"{diff.days} days ago"
    hours = diff.seconds // 3600
    if hours > 0:
        return f"{hours} hours ago"
    minutes = diff.seconds // 60
    return f"{minutes} minutes ago"

# Register without decorator
app.jinja_env.filters["timeago"] = timeago_filter

# Context processor — inject variables into all templates
@app.context_processor
def inject_globals():
    return {
        "site_name": "MyApp",
        "current_year": 2024,
        "nav_items": [
            {"url": url_for("main.index"), "label": "Home"},
            {"url": url_for("main.about"), "label": "About"},
        ]
    }

# Global functions available in all templates
@app.template_global()
def format_currency(amount, currency="USD"):
    return f"{currency} {amount:,.2f}"
```

---

## 7. Static Files

```python
# Accessing static files in views
url_for("static", filename="css/main.css")
url_for("static", filename="js/app.js")
url_for("static", filename="img/logo.png")
```

```html
<!-- In templates -->
<link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
<script src="{{ url_for('static', filename='js/app.js') }}"></script>
<img src="{{ url_for('static', filename='img/logo.png') }}" alt="Logo">
```

```python
# Blueprint static files — each blueprint can have its own static folder
auth_bp = Blueprint("auth", __name__,
                    template_folder="templates",
                    static_folder="static",
                    static_url_path="/auth/static")

# In templates for blueprint static:
url_for("auth.static", filename="auth-specific.css")
```

---

## 8. Forms & User Input

### Manual Form Handling

```python
@app.route("/register", methods=["GET", "POST"])
def register():
    errors = {}

    if request.method == "POST":
        username = request.form.get("username", "").strip()
        email    = request.form.get("email", "").strip()
        password = request.form.get("password", "")

        # Validation
        if not username:
            errors["username"] = "Username is required"
        elif len(username) < 3:
            errors["username"] = "Username must be at least 3 characters"

        if not email or "@" not in email:
            errors["email"] = "Valid email required"

        if len(password) < 8:
            errors["password"] = "Password must be at least 8 characters"

        if not errors:
            # Process the valid data
            create_user(username, email, password)
            flash("Registered successfully!", "success")
            return redirect(url_for("auth.login"))

    return render_template("register.html",
                           errors=errors,
                           form_data=request.form)
```

### Flask-WTF (Recommended for Forms)

```bash
pip install flask-wtf
```

```python
# config.py
WTF_CSRF_ENABLED  = True
SECRET_KEY        = "your-secret-key"   # Required for CSRF tokens
```

```python
# forms.py
from flask_wtf import FlaskForm
from wtforms import (StringField, PasswordField, EmailField, TextAreaField,
                     IntegerField, FloatField, BooleanField, SelectField,
                     SelectMultipleField, FileField, HiddenField, SubmitField)
from wtforms.validators import (DataRequired, Email, EqualTo, Length, Optional,
                                NumberRange, URL, Regexp, ValidationError)
from flask_wtf.file import FileField, FileRequired, FileAllowed

class LoginForm(FlaskForm):
    email    = EmailField("Email", validators=[DataRequired(), Email()])
    password = PasswordField("Password", validators=[DataRequired()])
    remember = BooleanField("Remember me")
    submit   = SubmitField("Log In")

class RegisterForm(FlaskForm):
    username = StringField("Username", validators=[
        DataRequired(),
        Length(min=3, max=20, message="Username must be 3-20 characters"),
        Regexp(r"^[\w.-]+$", message="Only letters, numbers, dots, hyphens")
    ])
    email    = EmailField("Email", validators=[DataRequired(), Email()])
    password = PasswordField("Password", validators=[
        DataRequired(),
        Length(min=8, message="Minimum 8 characters")
    ])
    confirm  = PasswordField("Confirm Password", validators=[
        DataRequired(),
        EqualTo("password", message="Passwords must match")
    ])
    submit   = SubmitField("Register")

    # Custom validator — method name must be validate_<fieldname>
    def validate_username(self, field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError("Username already taken")

    def validate_email(self, field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError("Email already registered")

class PostForm(FlaskForm):
    title    = StringField("Title", validators=[DataRequired(), Length(max=200)])
    body     = TextAreaField("Body", validators=[DataRequired()])
    category = SelectField("Category", choices=[
        ("tech", "Technology"),
        ("life", "Lifestyle"),
        ("news", "News")
    ])
    tags     = SelectMultipleField("Tags", choices=[
        ("python", "Python"),
        ("flask", "Flask")
    ])
    avatar   = FileField("Avatar", validators=[
        FileAllowed(["jpg", "png", "gif"], "Images only!")
    ])
    draft    = BooleanField("Save as draft", default=False)
    submit   = SubmitField("Publish")
```

```python
# routes.py
from .forms import LoginForm, RegisterForm

@app.route("/login", methods=["GET", "POST"])
def login():
    form = LoginForm()
    if form.validate_on_submit():     # True if POST and all validators pass
        user = User.query.filter_by(email=form.email.data).first()
        if user and user.check_password(form.password.data):
            login_user(user, remember=form.remember.data)
            next_page = request.args.get("next")
            return redirect(next_page or url_for("main.index"))
        flash("Invalid email or password", "error")
    return render_template("auth/login.html", form=form)
```

```html
<!-- templates/auth/login.html -->
{% extends "base.html" %}
{% block content %}
<form method="POST" novalidate>
    {{ form.hidden_tag() }}    {# CSRF token — required! #}

    <div>
        {{ form.email.label }}
        {{ form.email(class="form-control",
                      placeholder="you@example.com") }}
        {% for error in form.email.errors %}
            <span class="error">{{ error }}</span>
        {% endfor %}
    </div>

    <div>
        {{ form.password.label }}
        {{ form.password(class="form-control") }}
        {% for error in form.password.errors %}
            <span class="error">{{ error }}</span>
        {% endfor %}
    </div>

    {{ form.remember() }} {{ form.remember.label }}

    {{ form.submit(class="btn btn-primary") }}
</form>
{% endblock %}
```

---

## 9. Blueprints & Application Structure

Blueprints let you split your app into modular components — each with its own routes, templates, and static files.

### Creating a Blueprint

```python
# app/auth/__init__.py
from flask import Blueprint

auth_bp = Blueprint(
    "auth",                         # Blueprint name (used in url_for)
    __name__,
    template_folder="templates",    # Blueprint-specific templates
    url_prefix="/auth"              # All routes prefixed with /auth
)

from . import routes                # Import routes to register them
```

```python
# app/auth/routes.py
from flask import render_template, redirect, url_for, flash, request
from . import auth_bp
from .forms import LoginForm, RegisterForm

@auth_bp.route("/login", methods=["GET", "POST"])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for("main.index"))  # "blueprint.function"
    return render_template("auth/login.html", form=form)

@auth_bp.route("/logout")
def logout():
    # logout_user()
    return redirect(url_for("auth.login"))

@auth_bp.route("/register", methods=["GET", "POST"])
def register():
    form = RegisterForm()
    if form.validate_on_submit():
        pass
    return render_template("auth/register.html", form=form)
```

```python
# app/main/__init__.py
from flask import Blueprint
main_bp = Blueprint("main", __name__)
from . import routes
```

```python
# app/main/routes.py
from flask import render_template
from . import main_bp

@main_bp.route("/")
def index():
    return render_template("main/index.html")

@main_bp.route("/about")
def about():
    return render_template("main/about.html")
```

### Registering Blueprints

```python
# app/__init__.py
from flask import Flask

def create_app(config=None):
    app = Flask(__name__)

    # Register blueprints
    from .auth import auth_bp
    from .main import main_bp
    from .api  import api_bp

    app.register_blueprint(auth_bp, url_prefix="/auth")
    app.register_blueprint(main_bp)
    app.register_blueprint(api_bp, url_prefix="/api/v1")

    return app
```

### `url_for` with Blueprints

```python
# Format: "blueprint_name.function_name"
url_for("auth.login")
url_for("auth.register")
url_for("main.index")
url_for("api.get_users")

# From inside the same blueprint, use relative:
url_for(".login")       # Same as url_for("auth.login") inside auth blueprint
```

---

## 10. Application Factory Pattern

The factory pattern is the standard for production Flask apps. It lets you create multiple app instances (e.g., one for testing), avoids circular imports, and enables using extensions without a circular reference to the app.

### Extensions Module

```python
# app/extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager
from flask_mail import Mail
from flask_caching import Cache
from flask_wtf.csrf import CSRFProtect

db        = SQLAlchemy()
migrate   = Migrate()
login_mgr = LoginManager()
mail      = Mail()
cache     = Cache()
csrf      = CSRFProtect()
```

### The Factory Function

```python
# app/__init__.py
from flask import Flask
from .extensions import db, migrate, login_mgr, mail, cache, csrf
from .config import config_map

def create_app(config_name="default"):
    app = Flask(__name__, instance_relative_config=True)

    # Load configuration
    app.config.from_object(config_map[config_name])
    app.config.from_pyfile("config.py", silent=True)  # Instance config (optional)

    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    login_mgr.init_app(app)
    mail.init_app(app)
    cache.init_app(app)
    csrf.init_app(app)

    # Configure login manager
    login_mgr.login_view = "auth.login"
    login_mgr.login_message = "Please log in to access this page."
    login_mgr.login_message_category = "warning"

    # Register blueprints
    from .auth import auth_bp
    from .main import main_bp
    from .api  import api_bp
    app.register_blueprint(auth_bp, url_prefix="/auth")
    app.register_blueprint(main_bp)
    app.register_blueprint(api_bp, url_prefix="/api/v1")

    # Register error handlers
    from .errors import register_error_handlers
    register_error_handlers(app)

    # Shell context (for 'flask shell')
    @app.shell_context_processor
    def make_shell_context():
        from .models import User, Post
        return {"db": db, "User": User, "Post": Post}

    return app
```

---

## 11. Configuration

### Config Classes

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    """Base configuration."""
    SECRET_KEY                  = os.environ.get("SECRET_KEY") or "fallback-dev-key"
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_RECORD_QUERIES   = False
    MAIL_SERVER                 = os.environ.get("MAIL_SERVER", "localhost")
    MAIL_PORT                   = int(os.environ.get("MAIL_PORT", 587))
    MAIL_USE_TLS                = os.environ.get("MAIL_USE_TLS", "True") == "True"
    MAIL_USERNAME               = os.environ.get("MAIL_USERNAME")
    MAIL_PASSWORD               = os.environ.get("MAIL_PASSWORD")
    MAX_CONTENT_LENGTH          = 16 * 1024 * 1024   # 16 MB max upload
    UPLOAD_FOLDER               = os.path.join(os.path.dirname(__file__), "uploads")
    ALLOWED_EXTENSIONS          = {"png", "jpg", "jpeg", "gif", "pdf"}

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG                   = True
    SQLALCHEMY_DATABASE_URI = os.environ.get("DEV_DATABASE_URL") or "sqlite:///dev.db"
    SQLALCHEMY_ECHO         = True     # Log SQL queries
    CACHE_TYPE              = "SimpleCache"

class TestingConfig(Config):
    TESTING                 = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"
    WTF_CSRF_ENABLED        = False    # Disable CSRF for testing
    CACHE_TYPE              = "SimpleCache"

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL")
    CACHE_TYPE              = "RedisCache"
    CACHE_REDIS_URL         = os.environ.get("REDIS_URL", "redis://localhost:6379/0")

    @classmethod
    def init_app(cls, app):
        Config.init_app(app)
        # Production-specific setup: logging, error emails, etc.
        import logging
        from logging.handlers import RotatingFileHandler
        handler = RotatingFileHandler("app.log", maxBytes=10_000, backupCount=3)
        handler.setLevel(logging.INFO)
        app.logger.addHandler(handler)

config_map = {
    "development": DevelopmentConfig,
    "testing":     TestingConfig,
    "production":  ProductionConfig,
    "default":     DevelopmentConfig,
}
```

### Accessing Config

```python
from flask import current_app

app.config["SECRET_KEY"]
current_app.config["DEBUG"]

# Check if in debug mode
app.debug
current_app.debug
```

---

## 12. Database with Flask-SQLAlchemy

```bash
pip install flask-sqlalchemy
```

### Defining Models

```python
# app/models.py
from datetime import datetime
from .extensions import db

class User(db.Model):
    __tablename__ = "users"

    id            = db.Column(db.Integer, primary_key=True)
    username      = db.Column(db.String(80), unique=True, nullable=False, index=True)
    email         = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    is_active     = db.Column(db.Boolean, default=True, nullable=False)
    is_admin      = db.Column(db.Boolean, default=False, nullable=False)
    created_at    = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)
    updated_at    = db.Column(db.DateTime, default=datetime.utcnow,
                              onupdate=datetime.utcnow)
    avatar_url    = db.Column(db.String(255))
    bio           = db.Column(db.Text)

    # Relationships
    posts         = db.relationship("Post", back_populates="author",
                                    lazy="dynamic", cascade="all, delete-orphan")
    profile       = db.relationship("UserProfile", back_populates="user",
                                    uselist=False)  # One-to-one

    def set_password(self, password):
        from werkzeug.security import generate_password_hash
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        from werkzeug.security import check_password_hash
        return check_password_hash(self.password_hash, password)

    def to_dict(self):
        return {
            "id":         self.id,
            "username":   self.username,
            "email":      self.email,
            "created_at": self.created_at.isoformat(),
        }

    def __repr__(self):
        return f"<User {self.username!r}>"

# Many-to-many association table
post_tags = db.Table(
    "post_tags",
    db.Column("post_id", db.Integer, db.ForeignKey("posts.id"), primary_key=True),
    db.Column("tag_id",  db.Integer, db.ForeignKey("tags.id"),  primary_key=True),
)

class Post(db.Model):
    __tablename__ = "posts"

    id         = db.Column(db.Integer, primary_key=True)
    title      = db.Column(db.String(200), nullable=False)
    slug       = db.Column(db.String(200), unique=True, nullable=False, index=True)
    body       = db.Column(db.Text, nullable=False)
    published  = db.Column(db.Boolean, default=False)
    views      = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    user_id    = db.Column(db.Integer, db.ForeignKey("users.id"), nullable=False)

    # Relationships
    author   = db.relationship("User", back_populates="posts")
    tags     = db.relationship("Tag", secondary=post_tags, back_populates="posts")
    comments = db.relationship("Comment", back_populates="post",
                               lazy="dynamic", order_by="Comment.created_at")

class Tag(db.Model):
    __tablename__ = "tags"
    id    = db.Column(db.Integer, primary_key=True)
    name  = db.Column(db.String(50), unique=True, nullable=False)
    posts = db.relationship("Post", secondary=post_tags, back_populates="tags")

class Comment(db.Model):
    __tablename__ = "comments"
    id         = db.Column(db.Integer, primary_key=True)
    body       = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    post_id    = db.Column(db.Integer, db.ForeignKey("posts.id"), nullable=False)
    author_id  = db.Column(db.Integer, db.ForeignKey("users.id"), nullable=False)
    post       = db.relationship("Post", back_populates="comments")
    author     = db.relationship("User")
```

### Column Types

```python
db.Integer          # Int
db.BigInteger       # Big int
db.Float            # Float
db.Numeric(10, 2)   # Decimal (precision, scale) — use for money
db.String(n)        # VARCHAR(n)
db.Text             # Unlimited text
db.Boolean          # Bool
db.DateTime         # datetime
db.Date             # date
db.Time             # time
db.JSON             # JSON (PostgreSQL native, SQLite text)
db.LargeBinary      # BLOB
db.Enum("a","b","c")  # ENUM
```

### Querying

```python
# Create
user = User(username="alice", email="alice@example.com")
user.set_password("secure123")
db.session.add(user)
db.session.commit()

# Bulk insert
db.session.add_all([User(...), User(...), User(...)])
db.session.commit()

# Read — all()
users = User.query.all()

# Read — filter
user = User.query.filter_by(username="alice").first()
user = User.query.filter_by(username="alice").first_or_404()
user = User.query.get(42)              # By primary key (deprecated in SQLAlchemy 2)
user = db.session.get(User, 42)        # Preferred in SQLAlchemy 2

# filter() — more powerful
users = User.query.filter(User.email.endswith("@example.com")).all()
users = User.query.filter(User.is_active == True).all()
users = User.query.filter(User.created_at >= "2024-01-01").all()

# Multiple conditions
from sqlalchemy import and_, or_, not_
users = User.query.filter(
    and_(User.is_active == True, User.is_admin == False)
).all()

users = User.query.filter(
    or_(User.username == "alice", User.username == "bob")
).all()

# IN
users = User.query.filter(User.id.in_([1, 2, 3])).all()

# LIKE / ilike
users = User.query.filter(User.username.like("a%")).all()
users = User.query.filter(User.username.ilike("A%")).all()  # Case-insensitive

# Order, limit, offset
posts = Post.query.order_by(Post.created_at.desc()).limit(10).all()
posts = Post.query.order_by(Post.title.asc()).offset(20).limit(10).all()

# Count
count = User.query.count()
count = User.query.filter_by(is_active=True).count()

# Pagination
page     = request.args.get("page", 1, type=int)
per_page = 20
pagination = Post.query.order_by(Post.created_at.desc())\
                       .paginate(page=page, per_page=per_page, error_out=False)

posts       = pagination.items        # Items on current page
total       = pagination.total        # Total count
pages       = pagination.pages        # Total pages
has_next    = pagination.has_next
has_prev    = pagination.has_prev
next_num    = pagination.next_num
prev_num    = pagination.prev_num

# Update
user = User.query.get(1)
user.email = "new@example.com"
db.session.commit()

# Bulk update
User.query.filter_by(is_active=False).update({"is_active": True})
db.session.commit()

# Delete
user = User.query.get(1)
db.session.delete(user)
db.session.commit()

# Bulk delete
User.query.filter_by(is_active=False).delete()
db.session.commit()

# Joins
posts = db.session.query(Post, User)\
          .join(User, Post.user_id == User.id)\
          .filter(User.is_active == True)\
          .all()

# Aggregates
from sqlalchemy import func
result = db.session.query(func.count(User.id)).scalar()
result = db.session.query(func.avg(Post.views)).scalar()
result = db.session.query(
    User.username,
    func.count(Post.id).label("post_count")
).join(Post).group_by(User.id).all()
```

### Session Management

```python
# All changes are staged until commit
try:
    user = User(username="alice")
    db.session.add(user)
    post = Post(title="Hello", author=user)
    db.session.add(post)
    db.session.commit()
except Exception:
    db.session.rollback()
    raise
finally:
    db.session.remove()   # Return session to pool (done automatically in requests)
```

---

## 13. Database Migrations with Flask-Migrate

```bash
pip install flask-migrate
```

```python
# app/extensions.py
from flask_migrate import Migrate
migrate = Migrate()

# app/__init__.py
migrate.init_app(app, db)
```

```bash
# One-time init
flask db init             # Creates migrations/ folder

# Create a migration after changing models
flask db migrate -m "add user avatar column"

# Review the generated migration in migrations/versions/

# Apply migrations
flask db upgrade

# Rollback one migration
flask db downgrade

# Show migration history
flask db history

# Show current revision
flask db current

# Stamp the database to a specific revision (no migration)
flask db stamp head
```

---

## 14. Authentication with Flask-Login

```bash
pip install flask-login
```

```python
# app/extensions.py
from flask_login import LoginManager
login_mgr = LoginManager()
```

```python
# app/models.py
from flask_login import UserMixin

class User(UserMixin, db.Model):
    # UserMixin provides:
    # is_authenticated, is_active, is_anonymous, get_id()
    ...
```

```python
# app/__init__.py — configure login manager
from .extensions import login_mgr
login_mgr.init_app(app)
login_mgr.login_view         = "auth.login"
login_mgr.login_message      = "Please log in to access this page."
login_mgr.login_message_category = "warning"

# User loader — tells Flask-Login how to load a user from session
@login_mgr.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

```python
# app/auth/routes.py
from flask_login import login_user, logout_user, login_required, current_user

@auth_bp.route("/login", methods=["GET", "POST"])
def login():
    if current_user.is_authenticated:
        return redirect(url_for("main.index"))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user and user.check_password(form.password.data):
            login_user(user, remember=form.remember.data)
            # Safe redirect back to the page they were trying to access
            next_page = request.args.get("next")
            if not next_page or not next_page.startswith("/"):
                next_page = url_for("main.index")
            return redirect(next_page)
        flash("Invalid email or password.", "error")
    return render_template("auth/login.html", form=form)

@auth_bp.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for("main.index"))

# Protecting routes
@main_bp.route("/dashboard")
@login_required
def dashboard():
    return render_template("main/dashboard.html")

# Check in views
@main_bp.route("/admin")
@login_required
def admin():
    if not current_user.is_admin:
        abort(403)
    return render_template("admin.html")
```

```html
<!-- In templates -->
{% if current_user.is_authenticated %}
    <p>Hello, {{ current_user.username }}!</p>
    <a href="{{ url_for('auth.logout') }}">Logout</a>
{% else %}
    <a href="{{ url_for('auth.login') }}">Login</a>
{% endif %}
```

### Role-Based Access Control (Simple)

```python
from functools import wraps
from flask import abort
from flask_login import current_user

def admin_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if not current_user.is_authenticated or not current_user.is_admin:
            abort(403)
        return f(*args, **kwargs)
    return decorated

def role_required(*roles):
    def decorator(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            if not current_user.is_authenticated:
                abort(401)
            if current_user.role not in roles:
                abort(403)
            return f(*args, **kwargs)
        return decorated
    return decorator

@app.route("/admin")
@login_required
@admin_required
def admin_panel():
    ...

@app.route("/moderate")
@login_required
@role_required("admin", "moderator")
def moderate():
    ...
```

---

## 15. REST APIs & Flask-RESTful

### Pure Flask JSON API

```python
from flask import jsonify, request, abort
from functools import wraps

# Simple decorator for token auth
def require_api_key(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        key = request.headers.get("X-API-Key")
        if key != current_app.config["API_KEY"]:
            return jsonify(error="Unauthorized"), 401
        return f(*args, **kwargs)
    return decorated

@api_bp.route("/users", methods=["GET"])
def get_users():
    page     = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 20, type=int)
    search   = request.args.get("search", "")

    query = User.query
    if search:
        query = query.filter(User.username.ilike(f"%{search}%"))

    pagination = query.paginate(page=page, per_page=per_page, error_out=False)

    return jsonify({
        "users":      [u.to_dict() for u in pagination.items],
        "total":      pagination.total,
        "page":       page,
        "per_page":   per_page,
        "pages":      pagination.pages,
        "has_next":   pagination.has_next,
        "has_prev":   pagination.has_prev,
    })

@api_bp.route("/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())

@api_bp.route("/users", methods=["POST"])
def create_user():
    data = request.get_json()
    if not data:
        return jsonify(error="JSON body required"), 400

    # Validate required fields
    required = ["username", "email", "password"]
    missing = [f for f in required if f not in data]
    if missing:
        return jsonify(error=f"Missing fields: {missing}"), 422

    user = User(username=data["username"], email=data["email"])
    user.set_password(data["password"])
    db.session.add(user)
    try:
        db.session.commit()
    except Exception:
        db.session.rollback()
        return jsonify(error="Username or email already exists"), 409

    return jsonify(user.to_dict()), 201

@api_bp.route("/users/<int:user_id>", methods=["PUT"])
@login_required
def update_user(user_id):
    user = User.query.get_or_404(user_id)
    if user.id != current_user.id and not current_user.is_admin:
        return jsonify(error="Forbidden"), 403

    data = request.get_json() or {}
    for field in ["username", "email", "bio"]:
        if field in data:
            setattr(user, field, data[field])

    db.session.commit()
    return jsonify(user.to_dict())

@api_bp.route("/users/<int:user_id>", methods=["DELETE"])
@login_required
@admin_required
def delete_user(user_id):
    user = User.query.get_or_404(user_id)
    db.session.delete(user)
    db.session.commit()
    return "", 204
```

### Flask-RESTful

```bash
pip install flask-restful
```

```python
from flask_restful import Api, Resource, reqparse, fields, marshal_with, abort

api = Api(app)    # Or attach to blueprint

# Define output schema
user_fields = {
    "id":         fields.Integer,
    "username":   fields.String,
    "email":      fields.String,
    "created_at": fields.DateTime(dt_format="iso8601"),
}

# Request parser (input validation)
user_parser = reqparse.RequestParser()
user_parser.add_argument("username", required=True, help="Username required")
user_parser.add_argument("email",    required=True, help="Email required",
                         location="json")
user_parser.add_argument("password", required=True, location="json")

class UserListResource(Resource):
    @marshal_with(user_fields)
    def get(self):
        return User.query.all()

    @marshal_with(user_fields)
    def post(self):
        args = user_parser.parse_args()
        user = User(username=args["username"], email=args["email"])
        user.set_password(args["password"])
        db.session.add(user)
        db.session.commit()
        return user, 201

class UserResource(Resource):
    @marshal_with(user_fields)
    def get(self, user_id):
        return User.query.get_or_404(user_id)

    @marshal_with(user_fields)
    def put(self, user_id):
        user = User.query.get_or_404(user_id)
        args = user_parser.parse_args()
        user.username = args["username"]
        db.session.commit()
        return user

    def delete(self, user_id):
        user = User.query.get_or_404(user_id)
        db.session.delete(user)
        db.session.commit()
        return "", 204

api.add_resource(UserListResource, "/api/users")
api.add_resource(UserResource,     "/api/users/<int:user_id>")
```

### JWT Authentication

```bash
pip install flask-jwt-extended
```

```python
from flask_jwt_extended import (
    JWTManager, create_access_token, create_refresh_token,
    jwt_required, get_jwt_identity, get_jwt
)

jwt = JWTManager(app)
app.config["JWT_SECRET_KEY"]            = "jwt-secret"
app.config["JWT_ACCESS_TOKEN_EXPIRES"]  = timedelta(hours=1)
app.config["JWT_REFRESH_TOKEN_EXPIRES"] = timedelta(days=30)

@app.route("/auth/token", methods=["POST"])
def get_token():
    data = request.get_json()
    user = User.query.filter_by(email=data["email"]).first()
    if not user or not user.check_password(data["password"]):
        return jsonify(error="Bad credentials"), 401

    access_token  = create_access_token(identity=user.id,
                                         additional_claims={"role": user.role})
    refresh_token = create_refresh_token(identity=user.id)
    return jsonify(access_token=access_token, refresh_token=refresh_token)

@app.route("/auth/refresh", methods=["POST"])
@jwt_required(refresh=True)
def refresh():
    user_id      = get_jwt_identity()
    access_token = create_access_token(identity=user_id)
    return jsonify(access_token=access_token)

@app.route("/api/protected")
@jwt_required()
def protected():
    user_id = get_jwt_identity()
    claims  = get_jwt()
    user    = User.query.get(user_id)
    return jsonify(user=user.to_dict())
```

---

## 16. Error Handling

```python
# app/errors.py
from flask import jsonify, render_template, request

def register_error_handlers(app):

    def wants_json():
        """True if client prefers JSON (API request)."""
        return (request.path.startswith("/api/") or
                request.accept_mimetypes.best == "application/json")

    @app.errorhandler(400)
    def bad_request(e):
        if wants_json():
            return jsonify(error="Bad request", message=str(e)), 400
        return render_template("errors/400.html", error=e), 400

    @app.errorhandler(401)
    def unauthorized(e):
        if wants_json():
            return jsonify(error="Unauthorized"), 401
        return render_template("errors/401.html"), 401

    @app.errorhandler(403)
    def forbidden(e):
        if wants_json():
            return jsonify(error="Forbidden"), 403
        return render_template("errors/403.html"), 403

    @app.errorhandler(404)
    def not_found(e):
        if wants_json():
            return jsonify(error="Not found"), 404
        return render_template("errors/404.html"), 404

    @app.errorhandler(405)
    def method_not_allowed(e):
        return jsonify(error="Method not allowed"), 405

    @app.errorhandler(409)
    def conflict(e):
        return jsonify(error="Conflict", message=str(e)), 409

    @app.errorhandler(422)
    def unprocessable(e):
        return jsonify(error="Unprocessable entity", message=str(e)), 422

    @app.errorhandler(429)
    def too_many_requests(e):
        return jsonify(error="Too many requests"), 429

    @app.errorhandler(500)
    def internal_error(e):
        db.session.rollback()    # Rollback on server error
        if wants_json():
            return jsonify(error="Internal server error"), 500
        return render_template("errors/500.html"), 500

# Custom exceptions
class APIError(Exception):
    status_code = 400
    def __init__(self, message, status_code=None, payload=None):
        super().__init__(message)
        self.message    = message
        self.payload    = payload
        if status_code: self.status_code = status_code

    def to_dict(self):
        rv = dict(self.payload or {})
        rv["error"]   = type(self).__name__
        rv["message"] = self.message
        return rv

@app.errorhandler(APIError)
def handle_api_error(e):
    return jsonify(e.to_dict()), e.status_code

# Raise anywhere:
raise APIError("User not found", status_code=404)
raise APIError("Validation failed", status_code=422,
               payload={"fields": {"email": "Invalid"}})

# abort() shorthand
from flask import abort
abort(404)
abort(403, description="Admin access required")
```

---

## 17. Middleware & Hooks

### Request Hooks

```python
@app.before_request
def before_every_request():
    """Runs before every request."""
    g.start_time = time.time()

    # Maintenance mode check
    if app.config.get("MAINTENANCE_MODE") and request.path != "/maintenance":
        return render_template("maintenance.html"), 503

@app.after_request
def after_every_request(response):
    """Runs after every request (if no unhandled exception)."""
    # Add security headers to every response
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"]         = "SAMEORIGIN"
    response.headers["X-XSS-Protection"]        = "1; mode=block"

    # Log request duration
    if hasattr(g, "start_time"):
        elapsed = time.time() - g.start_time
        app.logger.debug(f"{request.method} {request.path} — {elapsed:.3f}s")

    return response

@app.teardown_request
def teardown_request(exception):
    """Runs at the end of every request, even if an exception occurred."""
    if exception:
        app.logger.error(f"Request teardown with error: {exception}")

@app.teardown_appcontext
def teardown_appcontext(exception):
    """Runs when the app context is torn down."""
    db.session.remove()

# Blueprint-level hooks (only for that blueprint's requests)
@auth_bp.before_request
def check_auth():
    pass

@auth_bp.after_request
def add_cors(response):
    response.headers["Access-Control-Allow-Origin"] = "*"
    return response
```

### Application Context & `g`

```python
from flask import g, current_app

# g is a namespace for storing data during a single request
# It's reset for every new request

@app.before_request
def load_current_user():
    user_id = session.get("user_id")
    g.user = User.query.get(user_id) if user_id else None

# Now use g.user in any view or template during this request
@app.route("/")
def index():
    if g.user:
        return f"Hello, {g.user.username}"
    return "Hello, stranger"

# Push/pop app context manually (outside request, e.g. in scripts)
with app.app_context():
    db.create_all()
    users = User.query.all()
```

### CORS

```bash
pip install flask-cors
```

```python
from flask_cors import CORS

# Allow all origins (development)
CORS(app)

# Specific origins (production)
CORS(app, resources={
    r"/api/*": {
        "origins":          ["https://myapp.com", "https://www.myapp.com"],
        "methods":          ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
        "allow_headers":    ["Content-Type", "Authorization", "X-API-Key"],
        "supports_credentials": True,
    }
})

# Per-blueprint
from flask_cors import cross_origin

@api_bp.route("/data")
@cross_origin(origins="https://myapp.com")
def data():
    ...
```

---

## 18. Sessions, Cookies & Caching

### Sessions

Flask sessions are stored in a **signed cookie** on the client (not server-side).

```python
from flask import session

# Store in session
session["user_id"]  = user.id
session["username"] = user.username
session.permanent   = True    # Use PERMANENT_SESSION_LIFETIME config

# Read from session
user_id = session.get("user_id")

# Delete from session
session.pop("user_id", None)

# Clear all session data
session.clear()

# Config
app.config["SECRET_KEY"]               = "must-be-set-and-secret"
app.config["SESSION_COOKIE_SECURE"]    = True   # HTTPS only
app.config["SESSION_COOKIE_HTTPONLY"]  = True   # No JS access
app.config["SESSION_COOKIE_SAMESITE"] = "Lax"
app.config["PERMANENT_SESSION_LIFETIME"] = timedelta(days=30)
```

### Server-Side Sessions (Flask-Session)

```bash
pip install flask-session
```

```python
from flask_session import Session

app.config["SESSION_TYPE"]             = "redis"
app.config["SESSION_REDIS"]            = redis.from_url("redis://localhost")
app.config["SESSION_PERMANENT"]        = True
app.config["SESSION_USE_SIGNER"]       = True
Session(app)
```

### Cookies

```python
# Set cookie in response
response = make_response(render_template("page.html"))
response.set_cookie(
    "user_prefs",
    value        = "dark-mode",
    max_age      = 60 * 60 * 24 * 30,  # 30 days in seconds
    secure       = True,                # HTTPS only
    httponly     = True,                # No JS access
    samesite     = "Lax",
    path         = "/",
    domain       = ".example.com"       # Accessible on subdomains
)

# Read cookie
prefs = request.cookies.get("user_prefs")

# Delete cookie
response.delete_cookie("user_prefs")
```

### Caching with Flask-Caching

```bash
pip install flask-caching
```

```python
from flask_caching import Cache

cache = Cache()
cache.init_app(app, config={
    "CACHE_TYPE":          "RedisCache",      # or "SimpleCache" for dev
    "CACHE_REDIS_URL":     "redis://localhost:6379/0",
    "CACHE_DEFAULT_TIMEOUT": 300,             # 5 minutes
})

# Cache a view function
@app.route("/")
@cache.cached(timeout=60)
def index():
    return render_template("index.html", posts=Post.query.all())

# Cache with dynamic key
@app.route("/user/<int:id>")
@cache.cached(timeout=120, key_prefix="user_%s")
def user_profile(id):
    ...

# Cache arbitrary data
def get_popular_posts():
    posts = cache.get("popular_posts")
    if posts is None:
        posts = Post.query.order_by(Post.views.desc()).limit(10).all()
        cache.set("popular_posts", posts, timeout=300)
    return posts

# Memoize (cache based on function args)
@cache.memoize(timeout=60)
def get_user_feed(user_id, page=1):
    return Post.query.filter_by(author_id=user_id).paginate(page=page)

# Invalidate cache
cache.delete("popular_posts")
cache.delete_memoized(get_user_feed, user_id=42)
cache.clear()   # Clear everything
```

---

## 19. File Uploads

```python
import os
from pathlib import Path
from werkzeug.utils import secure_filename

UPLOAD_FOLDER    = Path("uploads")
ALLOWED_EXTENSIONS = {"png", "jpg", "jpeg", "gif", "pdf", "docx"}
MAX_CONTENT_LENGTH = 16 * 1024 * 1024   # 16 MB

app.config["UPLOAD_FOLDER"]      = UPLOAD_FOLDER
app.config["MAX_CONTENT_LENGTH"] = MAX_CONTENT_LENGTH

def allowed_file(filename):
    return "." in filename and \
           filename.rsplit(".", 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route("/upload", methods=["GET", "POST"])
@login_required
def upload_file():
    if request.method == "POST":
        if "file" not in request.files:
            flash("No file part", "error")
            return redirect(request.url)

        file = request.files["file"]

        if file.filename == "":
            flash("No file selected", "error")
            return redirect(request.url)

        if not allowed_file(file.filename):
            flash("File type not allowed", "error")
            return redirect(request.url)

        # Sanitize filename — very important!
        filename = secure_filename(file.filename)

        # Make filename unique
        import uuid
        ext       = filename.rsplit(".", 1)[1]
        filename  = f"{uuid.uuid4().hex}.{ext}"

        save_path = app.config["UPLOAD_FOLDER"] / filename
        save_path.parent.mkdir(parents=True, exist_ok=True)
        file.save(save_path)

        # Save to DB
        current_user.avatar_url = filename
        db.session.commit()

        flash("File uploaded successfully", "success")
        return redirect(url_for("main.profile"))

    return render_template("upload.html")

@app.route("/uploads/<filename>")
def uploaded_file(filename):
    return send_from_directory(app.config["UPLOAD_FOLDER"], filename)
```

```html
<!-- Upload form — must have enctype! -->
<form method="POST" enctype="multipart/form-data">
    {{ form.hidden_tag() }}
    <input type="file" name="file" accept=".jpg,.png,.gif">
    <button type="submit">Upload</button>
</form>
```

---

## 20. Email, Background Tasks & Extensions

### Email with Flask-Mail

```bash
pip install flask-mail
```

```python
from flask_mail import Mail, Message

mail = Mail()
mail.init_app(app)

app.config.update(
    MAIL_SERVER   = "smtp.gmail.com",
    MAIL_PORT     = 587,
    MAIL_USE_TLS  = True,
    MAIL_USERNAME = os.environ["GMAIL_USER"],
    MAIL_PASSWORD = os.environ["GMAIL_PASS"],
    MAIL_DEFAULT_SENDER = ("MyApp", os.environ["GMAIL_USER"]),
)

def send_welcome_email(user):
    msg = Message(
        subject    = "Welcome to MyApp!",
        recipients = [user.email],
        html       = render_template("emails/welcome.html", user=user),
        body       = f"Welcome, {user.username}! Visit us at example.com",
    )
    mail.send(msg)

def send_password_reset(user, token):
    reset_url = url_for("auth.reset_password", token=token, _external=True)
    msg = Message(
        subject    = "Reset Your Password",
        recipients = [user.email],
        html       = render_template("emails/reset.html",
                                     user=user, reset_url=reset_url)
    )
    mail.send(msg)
```

### Background Tasks with Celery

```bash
pip install celery redis flower
```

```python
# app/celery_app.py
from celery import Celery

def make_celery(app):
    celery = Celery(
        app.import_name,
        backend  = app.config["CELERY_RESULT_BACKEND"],
        broker   = app.config["CELERY_BROKER_URL"],
    )
    celery.conf.update(app.config)

    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return self.run(*args, **kwargs)

    celery.Task = ContextTask
    return celery

# app/__init__.py
celery = make_celery(app)
```

```python
# app/tasks.py
from .celery_app import celery
from .extensions import mail
from flask_mail import Message

@celery.task
def send_email_async(subject, recipients, html_body):
    msg = Message(subject=subject, recipients=recipients, html=html_body)
    mail.send(msg)

@celery.task(bind=True, max_retries=3)
def process_image(self, image_path):
    try:
        # Heavy processing here
        result = resize_and_compress(image_path)
        return result
    except Exception as exc:
        raise self.retry(exc=exc, countdown=60)

# Usage in a route
@app.route("/register", methods=["POST"])
def register():
    # ... create user ...
    send_email_async.delay(
        subject    = "Welcome!",
        recipients = [user.email],
        html_body  = render_template("emails/welcome.html", user=user)
    )
    return redirect(url_for("main.index"))
```

```bash
# Run Celery worker
celery -A wsgi.celery worker --loglevel=info

# Monitor with Flower
celery -A wsgi.celery flower --port=5555
```

### Rate Limiting with Flask-Limiter

```bash
pip install flask-limiter
```

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    get_remote_address,
    app          = app,
    default_limits = ["200 per day", "50 per hour"],
    storage_uri    = "redis://localhost:6379",
)

@app.route("/api/login", methods=["POST"])
@limiter.limit("5 per minute")       # Stricter limit for login
def login():
    ...

@app.route("/api/public")
@limiter.exempt                       # No limit on this route
def public():
    ...
```

---

## 21. Testing Flask Apps

### Setup

```bash
pip install pytest pytest-flask coverage
```

```python
# tests/conftest.py
import pytest
from app import create_app
from app.extensions import db as _db
from app.models import User

@pytest.fixture(scope="session")
def app():
    """Create app with testing config."""
    app = create_app("testing")
    with app.app_context():
        _db.create_all()
        yield app
        _db.drop_all()

@pytest.fixture(scope="function")
def db(app):
    """Fresh database for each test."""
    with app.app_context():
        _db.session.begin_nested()
        yield _db
        _db.session.rollback()

@pytest.fixture
def client(app):
    return app.test_client()

@pytest.fixture
def runner(app):
    return app.test_cli_runner()

@pytest.fixture
def user(db):
    u = User(username="testuser", email="test@example.com")
    u.set_password("password123")
    db.session.add(u)
    db.session.commit()
    return u

@pytest.fixture
def auth_client(client, user):
    """Logged-in test client."""
    client.post("/auth/login", data={
        "email":    user.email,
        "password": "password123",
    }, follow_redirects=True)
    return client
```

```python
# tests/test_routes.py
def test_index(client):
    response = client.get("/")
    assert response.status_code == 200
    assert b"Welcome" in response.data

def test_login_page(client):
    response = client.get("/auth/login")
    assert response.status_code == 200

def test_login_valid(client, user):
    response = client.post("/auth/login", data={
        "email":    user.email,
        "password": "password123",
    }, follow_redirects=True)
    assert response.status_code == 200
    assert b"Dashboard" in response.data

def test_login_invalid(client, user):
    response = client.post("/auth/login", data={
        "email":    user.email,
        "password": "wrongpassword",
    })
    assert b"Invalid email or password" in response.data

def test_protected_route_requires_login(client):
    response = client.get("/dashboard", follow_redirects=False)
    assert response.status_code == 302
    assert "/auth/login" in response.headers["Location"]

def test_dashboard_when_logged_in(auth_client):
    response = auth_client.get("/dashboard")
    assert response.status_code == 200

def test_create_user_api(client):
    response = client.post("/api/users",
        json={"username": "alice", "email": "alice@test.com",
              "password": "secret123"})
    assert response.status_code == 201
    data = response.get_json()
    assert data["username"] == "alice"

def test_get_user_api(client, user):
    response = client.get(f"/api/users/{user.id}")
    assert response.status_code == 200
    assert response.get_json()["email"] == user.email

def test_404(client):
    response = client.get("/nonexistent")
    assert response.status_code == 404

# Testing flash messages
def test_register_flash(client):
    response = client.post("/auth/register", data={...},
                           follow_redirects=True)
    assert b"Registration successful" in response.data

# Testing JSON APIs
def test_json_response(client):
    response = client.get("/api/users",
                          headers={"Accept": "application/json"})
    assert response.content_type == "application/json"
    data = response.get_json()
    assert "users" in data
```

```bash
# Run tests
pytest
pytest -v                          # Verbose
pytest tests/test_routes.py        # Specific file
pytest -k "login"                  # Filter by name
pytest --tb=short                  # Short tracebacks

# Coverage
coverage run -m pytest
coverage report
coverage html                      # HTML report in htmlcov/
```

---

## 22. Deployment

### Production WSGI Servers

```bash
# Gunicorn (Linux/macOS)
pip install gunicorn
gunicorn wsgi:app
gunicorn wsgi:app --workers 4 --bind 0.0.0.0:8000
gunicorn wsgi:app --workers 4 --threads 2 --worker-class gthread

# uWSGI
pip install uwsgi
uwsgi --http 0.0.0.0:8000 --module wsgi:app --processes 4 --threads 2

# Waitress (Windows-compatible)
pip install waitress
waitress-serve --port=8000 wsgi:app
```

### Gunicorn Config File

```python
# gunicorn.conf.py
import multiprocessing

bind             = "0.0.0.0:8000"
workers          = multiprocessing.cpu_count() * 2 + 1
worker_class     = "gthread"
threads          = 2
timeout          = 30
keepalive        = 5
accesslog        = "-"          # Log to stdout
errorlog         = "-"
loglevel         = "info"
preload_app      = True         # Load app before forking workers
```

```bash
gunicorn -c gunicorn.conf.py wsgi:app
```

### Nginx Configuration

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # Serve static files directly (much faster than going through Flask)
    location /static/ {
        alias /var/www/myapp/app/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    location /uploads/ {
        alias /var/www/myapp/uploads/;
        expires 7d;
    }

    location / {
        proxy_pass         http://127.0.0.1:8000;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_redirect     off;
        client_max_body_size 16M;
    }
}
```

### systemd Service

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Flask Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myapp
EnvironmentFile=/var/www/myapp/.env
ExecStart=/var/www/myapp/.venv/bin/gunicorn \
          -c gunicorn.conf.py wsgi:app
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
KillMode=mixed
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable myapp
sudo systemctl start  myapp
sudo systemctl status myapp
sudo journalctl -u myapp -f     # Follow logs
```

### Docker

```dockerfile
# Dockerfile
FROM python:3.12-slim

WORKDIR /app

# Install dependencies first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app code
COPY . .

# Create non-root user
RUN useradd --create-home appuser && chown -R appuser /app
USER appuser

EXPOSE 8000

CMD ["gunicorn", "-c", "gunicorn.conf.py", "wsgi:app"]
```

```yaml
# docker-compose.yml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      - db
      - redis
    volumes:
      - ./uploads:/app/uploads
    restart: unless-stopped

  db:
    image: postgres:16
    environment:
      POSTGRES_DB:       myapp
      POSTGRES_USER:     myapp
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./app/static:/static:ro
    depends_on:
      - web

volumes:
  postgres_data:
```

---

## 23. Security Best Practices

### CSRF Protection

```python
# Flask-WTF handles this automatically with form.hidden_tag()
# For AJAX requests, include CSRF token in headers

# Get token for AJAX
from flask_wtf.csrf import generate_csrf

@app.route("/api/csrf-token")
def csrf_token():
    return jsonify(csrf_token=generate_csrf())

# In JavaScript:
# fetch('/api/csrf-token')
#   .then(r => r.json())
#   .then(data => {
#     headers['X-CSRFToken'] = data.csrf_token;
#   });

# Flask-WTF checks X-CSRFToken header automatically
```

### Password Hashing

```python
from werkzeug.security import generate_password_hash, check_password_hash

# Never store plain text passwords!
hash = generate_password_hash("my-password", method="pbkdf2:sha256", salt_length=16)
check_password_hash(hash, "my-password")   # True

# Even better: use bcrypt
from flask_bcrypt import Bcrypt
bcrypt = Bcrypt(app)
hash   = bcrypt.generate_password_hash("password").decode("utf-8")
bcrypt.check_password_hash(hash, "password")
```

### SQL Injection Prevention

```python
# ALWAYS use parameterized queries — SQLAlchemy does this automatically
# Never do string formatting in queries!

# ❌ DANGEROUS — SQL injection possible
db.engine.execute(f"SELECT * FROM users WHERE username = '{username}'")

# ✅ SAFE — SQLAlchemy ORM
User.query.filter_by(username=username).first()

# ✅ SAFE — parameterized raw SQL
db.session.execute(
    text("SELECT * FROM users WHERE username = :name"),
    {"name": username}
)
```

### XSS Prevention

```python
# Jinja2 auto-escapes HTML in templates — never disable this for user input!
# ❌ DON'T DO THIS for user content
{{ user_content | safe }}

# ✅ Jinja2 escapes by default
{{ user_content }}

# For intentional HTML content use bleach to sanitize
pip install bleach
import bleach
clean = bleach.clean(user_html, tags=["p","a","b","i"], attributes={"a": ["href"]})
```

### Security Headers

```python
@app.after_request
def security_headers(response):
    response.headers["X-Content-Type-Options"]    = "nosniff"
    response.headers["X-Frame-Options"]            = "SAMEORIGIN"
    response.headers["X-XSS-Protection"]           = "1; mode=block"
    response.headers["Referrer-Policy"]            = "strict-origin-when-cross-origin"
    response.headers["Permissions-Policy"]         = "geolocation=(), microphone=()"
    response.headers["Content-Security-Policy"]    = (
        "default-src 'self'; "
        "script-src 'self' https://cdn.jsdelivr.net; "
        "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; "
        "font-src 'self' https://fonts.gstatic.com; "
        "img-src 'self' data: https:; "
        "connect-src 'self'"
    )
    # HSTS — only if you're 100% committed to HTTPS
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
    return response
```

### Environment Variables & Secrets

```python
# Never hardcode secrets!
# ❌ WRONG
app.config["SECRET_KEY"] = "my-secret-key"

# ✅ RIGHT
app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]

# Use python-dotenv for local dev
from dotenv import load_dotenv
load_dotenv()  # Load .env file

# Add .env to .gitignore!
# Provide .env.example with dummy values as documentation
```

---

## 24. Quick Reference Card

### Flask CLI Commands

```bash
flask run                          # Run dev server
flask run --debug --host=0.0.0.0
flask shell                        # Python REPL with app context
flask routes                       # Show all registered routes
flask db init                      # Init migrations
flask db migrate -m "message"      # Create migration
flask db upgrade                   # Apply migrations
flask db downgrade                 # Rollback one migration
```

### Common Imports

```python
from flask import (
    Flask, Blueprint,
    render_template, render_template_string,
    request, redirect, url_for,
    abort, flash, make_response,
    jsonify, Response,
    send_file, send_from_directory,
    session, g, current_app,
    stream_with_context,
)
from flask_sqlalchemy import SQLAlchemy
from flask_migrate    import Migrate
from flask_login      import (LoginManager, UserMixin, login_user,
                               logout_user, login_required, current_user)
from flask_wtf        import FlaskForm
from flask_mail       import Mail, Message
from flask_caching    import Cache
from flask_cors       import CORS
from werkzeug.utils   import secure_filename
from werkzeug.security import generate_password_hash, check_password_hash
```

### HTTP Status Codes Reference

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST that created a resource |
| 204 | No Content | Successful DELETE |
| 301 | Moved Permanently | Permanent redirect |
| 302 | Found | Temporary redirect |
| 400 | Bad Request | Malformed request data |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Wrong HTTP method |
| 409 | Conflict | Duplicate resource (e.g. username taken) |
| 422 | Unprocessable Entity | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unhandled exception |
| 503 | Service Unavailable | Maintenance / overloaded |

### Essential Extensions

| Extension | Install | Purpose |
|-----------|---------|---------|
| Flask-SQLAlchemy | `flask-sqlalchemy` | ORM / database |
| Flask-Migrate | `flask-migrate` | DB schema migrations |
| Flask-Login | `flask-login` | User sessions & auth |
| Flask-WTF | `flask-wtf` | Forms + CSRF protection |
| Flask-Mail | `flask-mail` | Email sending |
| Flask-Caching | `flask-caching` | Response & data caching |
| Flask-CORS | `flask-cors` | Cross-Origin headers |
| Flask-Limiter | `flask-limiter` | Rate limiting |
| Flask-JWT-Extended | `flask-jwt-extended` | JWT tokens for APIs |
| Flask-Admin | `flask-admin` | Auto admin panel |
| Flask-RESTful | `flask-restful` | REST API helpers |
| Flask-Marshmallow | `flask-marshmallow` | Serialization + validation |
| Flask-Bcrypt | `flask-bcrypt` | Password hashing |
| Flask-Session | `flask-session` | Server-side sessions |
| Celery | `celery` | Background task queue |

### Patterns Quick Reference

```python
# Return JSON with status code
return jsonify({"error": "Not found"}), 404

# Redirect after POST (Post/Redirect/Get pattern)
return redirect(url_for("main.index"))

# Get request data safely
page = request.args.get("page", 1, type=int)
data = request.get_json() or {}
name = request.form.get("name", "").strip()

# 404 or object
user = User.query.get_or_404(user_id)
user = User.query.filter_by(username=name).first_or_404()

# Flash then redirect
flash("Saved!", "success")
return redirect(url_for("main.index"))

# Abort with status code
abort(404)
abort(403)

# Build absolute URL (for emails, etc.)
url = url_for("auth.verify", token=token, _external=True)

# Paginate query results
pagination = Post.query.paginate(page=page, per_page=20, error_out=False)

# Access current user anywhere (with Flask-Login)
from flask_login import current_user
current_user.is_authenticated
current_user.username
```

### The App Factory in One File

```python
# Minimal but complete app factory
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app(config=None):
    app = Flask(__name__)
    app.config["SECRET_KEY"]             = "dev"
    app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///dev.db"

    if config:
        app.config.update(config)

    db.init_app(app)

    from .routes import main
    app.register_blueprint(main)

    with app.app_context():
        db.create_all()

    return app
```
