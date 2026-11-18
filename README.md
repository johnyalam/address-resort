# Address Resort (Flask)

A learning project: I work through the official Flask documentation step by step and turn what I learn into a web application that will become a resort site.

- Flask Quickstart: https://flask.palletsprojects.com/en/stable/quickstart/
- Flask Tutorial (the "Flaskr" blog): https://flask.palletsprojects.com/en/stable/tutorial/

> **Where the project is today:** the code follows the official Quickstart and the Flaskr tutorial (users, login, posts, tests). The resort-specific features (rooms, bookings, addresses) are the next phase; see [Roadmap](#roadmap-from-blog-to-resort).

---

## What the app does right now

- Register, log in and log out (passwords are hashed, sessions use a signed cookie)
- Create, edit and delete posts (only the author can edit or delete)
- SQLite database, created with a custom `flask init-db` command
- Jinja2 templates with a shared base layout, plain CSS
- 24 automated tests with `pytest`

## Tech stack

| Part | Tool |
|---|---|
| Language | Python 3.12 |
| Framework | Flask |
| Database | SQLite (`sqlite3` from the standard library) |
| Templates | Jinja2 |
| Packaging | `pyproject.toml` with `flit_core` |
| Tests | `pytest` |

## Project structure

```
address-resort/
├── pyproject.toml            # project metadata, dependencies, pytest + coverage config
├── .gitignore                # .venv, instance/, __pycache__, *.db ...
└── flaskr/                   # the application package
    ├── __init__.py           # application factory: create_app()
    ├── db.py                 # SQLite connection, init-db command
    ├── schema.sql            # user and post tables
    ├── auth.py               # blueprint: register, login, logout, login_required
    ├── blog.py               # blueprint: index, create, update, delete
    ├── static/
    │   └── style.css         # site styling
    ├── templates/
    │   ├── base.html         # layout: nav bar, flash messages, blocks
    │   ├── hello.html        # first template example (render_template)
    │   ├── auth/
    │   │   ├── login.html
    │   │   └── register.html
    │   └── blog/
    │       ├── index.html
    │       ├── create.html
    │       └── update.html
    └── tests/
        ├── conftest.py       # fixtures: app, client, runner, auth
        ├── data.sql          # test data
        ├── test_factory.py
        ├── test_db.py
        ├── test_auth.py
        └── test_blog.py
```

At runtime Flask also creates an `instance/` folder holding `flaskr.sqlite`. It is git-ignored.

## Getting started

```bash
git clone https://github.com/johnyalam/address-resort.git
cd address-resort

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -e .                 # installs flask + the flaskr package
pip install pytest               # for the tests

flask --app flaskr init-db       # create the SQLite tables
flask --app flaskr run --debug   # http://127.0.0.1:5000
```

Run the tests:

```bash
python -m pytest flaskr/tests
```

---

## How I built it: the commit history, step by step

Each step below matches one commit, in the order I made them. To see exactly what changed in a step, run `git show <hash>`.

### Step 1: Create the environment (`fa2406b`, `4aea268`)

Created a virtual environment with `python -m venv .venv`, then added a `.gitignore`.

**Lesson:** my first commit accidentally included the whole `.venv/` folder. The "initial commit" removes it and adds `.gitignore`. A virtual environment is machine-specific and should never be committed. Create `.gitignore` *before* your first commit.

### Step 2: Hello World (`47e57c2`)

Quickstart: "A Minimal Application."

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return "<p>Hello, World!</p>"
```

Topics: the `Flask` class, `__name__`, the `@app.route()` decorator, returning HTML from a view.

### Step 3: Multiple routes (`1ce721f`)

Quickstart: "Routing."

- Query strings: `request.args.get("name", "World")`
- Escaping user input with `markupsafe.escape` (protects against injection)
- Variable rules: `<username>`, `<int:post_id>`, `<path:subpath>`
- Unique URLs: `/projects/` (trailing slash) versus `/about` (no trailing slash)

> This version of `hello.py` was later deleted. You can still read it with `git show 1ce721f:hello.py`.

### Step 4: Database initialization (`cb3d773`)

Moved from a single `hello.py` file to a real package (`flaskr/`), following the Flask tutorial.

- **Application factory** `create_app()` in `__init__.py`: configuration, instance folder, test config
- `db.py`: `get_db()` stores the connection on `g` (per-request storage); `close_db()` is registered with `teardown_appcontext`
- `schema.sql`: the `user` and `post` tables
- A custom CLI command: `flask --app flaskr init-db` (using `click`)
- `templates/hello.html` and `render_template()` with Jinja2 `{% if %}`

### Step 5: Authentication blueprint (`8ffe455`)

`auth.py` is a **Blueprint** registered in the factory.

- `/auth/register`: validates input, hashes the password with `generate_password_hash`, handles duplicate users via `IntegrityError`
- `/auth/login`: checks the hash with `check_password_hash`, stores `user_id` in `session`
- `@bp.before_app_request` loads the logged-in user into `g.user` on every request
- `/auth/logout`: `session.clear()`
- `login_required`: a custom **decorator** that redirects anonymous users to the login page
- `flash()` for error messages and `url_for()` + `redirect()` for navigation

### Step 6: HTML templates (`d554251`)

- `base.html`: the shared layout using **template inheritance** (`{% block %}`, `{% extends %}`)
- `login.html` and `register.html` extend the base
- `get_flashed_messages()` displays flashed messages
- `url_for('static', filename='style.css')` links the stylesheet

### Step 7: CSS (`b9aa34b`)

Added `static/style.css`: navigation bar, content area, flash messages, post layout. Flask serves anything in `static/` automatically.

### Step 8: Blog blueprint (`9caf780`)

`blog.py` and three templates give full **CRUD**:

- `index`: lists posts with a SQL `JOIN`
- `create`, `update`, `delete`: each protected with `@login_required`
- `get_post()`: returns **404** if the post is missing and **403** if you are not the author (`abort()`)
- `app.add_url_rule('/', endpoint='index')` so `url_for('index')` works
- `delete` accepts `POST` only (`methods=('POST',)`), never a link

### Step 9: Tests (`936088e`)

- `conftest.py` has fixtures: `app` (temporary database), `client`, `runner`, and an `auth` helper class
- `data.sql` is seed data for tests
- Tests cover the factory, database, authentication and blog views
- `@pytest.mark.parametrize` runs one test with many inputs
- `pyproject.toml` adds the package metadata and `pytest` / `coverage` settings

Result: **24 tests pass**.

---

## Important Flask topics covered (cheat sheet)

| Topic | Where to look in this repo |
|---|---|
| Minimal app, `@app.route` | commit `47e57c2` |
| Variable rules, query args, `escape` | commit `1ce721f` |
| Application factory | `flaskr/__init__.py` |
| Configuration and instance folder | `flaskr/__init__.py` |
| `g` and application context | `flaskr/db.py` |
| Custom CLI commands | `flaskr/db.py` (`init-db`) |
| Blueprints | `flaskr/auth.py`, `flaskr/blog.py` |
| Sessions and cookies | `flaskr/auth.py` (`login`) |
| Password hashing | `flaskr/auth.py` (`register`) |
| Request hooks (`before_app_request`) | `flaskr/auth.py` |
| Decorators (`login_required`) | `flaskr/auth.py` |
| Flash messages | `flaskr/auth.py`, `templates/base.html` |
| Jinja2 templates and inheritance | `flaskr/templates/` |
| Static files | `flaskr/static/style.css` |
| Error handling with `abort()` | `flaskr/blog.py` (`get_post`) |
| Testing with `pytest` | `flaskr/tests/` |
| Packaging with `pyproject.toml` | `pyproject.toml` |

## Known issues and small clean-ups

- `pyproject.toml` has `testpaths = ["tests"]`, but the tests live in `flaskr/tests/`. Running plain `pytest` prints a warning and falls back to searching recursively. Change it to `testpaths = ["flaskr/tests"]` (or move the tests folder to the repo root, as the official tutorial does).
- `SECRET_KEY` is `'dev'`. Fine for learning, but set a real random key in `instance/config.py` before deploying.
- `templates/hello.html` is not used by any route yet.

## Roadmap: from blog to resort

Ideas for the next commits (not built yet):

1. Rename the app and branding from "Flaskr" to "Address Resort"
2. Replace `post` with resort tables: `room`, `booking`, `guest`
3. Room list and room detail pages
4. Booking form with date validation
5. Admin-only pages (a role column on `user`)
6. Image uploads for room photos
7. Deploy (the tutorial's Deploy section: build a wheel, run with a production server)

## Credits

Based on the official [Flask documentation](https://flask.palletsprojects.com/) (Quickstart and Tutorial) by the Pallets Projects.
