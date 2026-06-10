# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| LearningAPI | The core Django app — contains the data models, serializers, views (API endpoints), migrations, and tests that make up the actual "Learning Operations" API. |
| LearningPlatform | The Django project package — holds project-wide configuration like settings.py, the root urls.py, and wsgi.py. This is the "glue" that ties all the apps together. |
| LogViewer | A small Django app dedicated to viewing application logs through a web page (its own views/urls/templates). |
| config | Deployment/infrastructure configuration — nginx configs and a YAML config file used when running the app in production or via Docker. |
| static | Source static assets (CSS/JS/images) used by Django templates before they're collected. |
| staticfiles | The output directory where Django's collectstatic command gathers all static files for production serving. |
| templates | Shared/global HTML templates used across the project (e.g., for Django admin or LogViewer). |
| logs | Where the running application writes its log files. |
| .github | GitHub-specific configuration, typically CI/CD workflows (GitHub Actions). |
| .vscode | Editor-specific settings/config for VS Code (debugging launch configs, recommended extensions, etc.). |
| .git | Git's internal repository data — version control history, not project code. |

Other notable top-level files: `manage.py` (Django's CLI entry point), `Pipfile`/`Pipfile.lock` (Python dependency management), `Dockerfile` and `entrypoint.sh` (containerization), `pytest.ini` (test config).

## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|------------------------------------------|
| models | Defines the database schema as Python classes (Django ORM models) — split into sub-packages like coursework, people, and skill to organize related models. |
| serializers | DRF serializers that convert model instances to/from JSON for the API, and handle input validation. |
| views | The API endpoint logic — handles incoming requests and returns responses, one file per resource (e.g., book_view.py, auth.py). |
| migrations | Auto-generated history of database schema changes, used by Django to create/update the actual database tables. |
| fixtures | JSON files with sample/seed data that can be loaded into the database (e.g., for development or testing). |
| tests | The automated test suite for the app's models, views, and endpoints. |

Other notable files: `admin.py` (Django admin site config), `middleware.py` and `decorators.py` (request-handling helpers), `metrics.py` (app metrics), `utils.py` (shared helper functions).

## 3. What is the Pipfile?

Pipfile is the dependency manifest for projects managed by Pipenv (Python's equivalent of package.json for Node, or requirements.txt but with more structure). Looking at Pipfile:

- `[[source]]` — tells Pipenv where to download packages from (PyPI, the official Python package index).
- `[packages]` — the production dependencies this Django project needs to run (Django itself, DRF, auth libraries, database driver, logging tools, etc.).
- `[dev-packages]` — dependencies only needed during development/testing (linters like pylint, testing tools like pytest, the debugger debugpy). These won't be installed in production.
- `[requires]` — pins the Python interpreter version (3.11.11) so everyone on the team (and any deployment server) uses the same Python version.
- `[scripts]` — shortcut commands you can run with `pipenv run <name>` (here, `migrate` runs `python3 manage.py migrate`).

Its purpose is reproducibility: anyone who runs `pipenv install` gets the exact same set of packages, avoiding "works on my machine" problems caused by version drift. The companion file Pipfile.lock (if present) pins exact versions/hashes for full reproducibility.

## 4. Key packages

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| django | The core web framework this entire project is built on. It provides the ORM (mapping Python classes to PostgreSQL tables), the URL routing system, the admin site (django.contrib.admin), authentication/user models (django.contrib.auth), middleware pipeline, settings system, and the request/response lifecycle. Every other package in this project is a plugin that hooks into Django — without it, nothing else works. |
| djangorestframework | Adds the tooling needed to turn this Django project into a REST API rather than a server-rendered HTML site. It provides Serializers (convert model instances ↔ JSON), ViewSets/APIView classes (handle GET/POST/PUT/DELETE logic), authentication classes (here, TokenAuthentication is configured in REST_FRAMEWORK settings), permission classes (IsAuthenticated is the default), and pagination (LimitOffsetPagination, 50 items/page). This is why the project is called "learn-ops-api" — DRF is what makes the JSON endpoints possible. |
| django-allauth | Handles user authentication and registration, including third-party/social login. In this project's INSTALLED_APPS, you can see allauth, allauth.account, allauth.socialaccount, and allauth.socialaccount.providers.github — meaning users can sign up/log in directly or via "Sign in with GitHub". It's paired with dj-rest-auth, which exposes allauth's login/logout/registration flows as REST API endpoints (so a frontend like React can call them) instead of allauth's default HTML login pages. Settings like SOCIALACCOUNT_LOGIN_ON_GET and ACCOUNT_EMAIL_VERIFICATION in settings.py configure how that login flow behaves. |

A quick way to see the chain: django is the foundation → djangorestframework turns it into an API → django-allauth (+ dj-rest-auth) bolts authentication/social-login onto that API.

## 5. What does `decorators.py` do?


## 6. What is a serializer, and how does it fit the request/response cycle?


## 7. One model and what it represents


## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View | | |
| ViewSet | | |

## 9. What replaces templates and why?