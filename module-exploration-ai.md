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


## 4. Key packages

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| django | |
| djangorestframework | |
| django-allauth | |

## 5. What does `decorators.py` do?


## 6. What is a serializer, and how does it fit the request/response cycle?


## 7. One model and what it represents


## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View | | |
| ViewSet | | |

## 9. What replaces templates and why?