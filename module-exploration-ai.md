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

A decorator is a function that wraps another function to add behavior before or after it runs, without changing the wrapped function's own code. In Python, a decorator is just a function that:

1. Takes a function as input
2. Defines a new function (a "wrapper") that does something extra
3. Returns that wrapper

When you put @decorator_name above a function definition, Python replaces the original function with whatever the decorator returns.

How this file's decorators work

def is_instructor():
    def decorator(func):
        def __wrapper(request, *args, **kwargs):
            if request.user.groups.filter(name='Instructors').exists():
                return func(request, *args, **kwargs)
            else:
                return Response(
                    {'message': 'You must be an instructor'},
                    status=status.HTTP_401_UNAUTHORIZED
                )
        return __wrapper
    return decorator

This is actually three layers of functions, which trips a lot of people up the first time:

is_instructor() — the outer function. Calling it (with the parentheses) is what produces the actual decorator. This pattern lets you configure a decorator if needed (this one doesn't take arguments, but the structure allows it).
decorator(func) — receives the view method you're decorating (e.g., create).

__wrapper(request, *args, **kwargs) — the function that actually runs instead of the original view method.
So the logic is: "Before letting the real view function run, check whether request.user belongs to the Instructors group. If yes, run the real function (func(...)) and return its result. If no, short-circuit and return a 401 Unauthorized response — the real view never even executes."

is_staff() is the identical pattern, just checking for the Staff group instead.

How it's used in the project
In course_view.py:26:

@method_decorator(is_instructor())
def create(self, request):
    ...
method_decorator is a Django helper that lets you apply a normal function decorator (like is_instructor()) to a class method — Django REST Framework class-based views need this small adapter because the decorator above was written assuming request is the first argument, but on a class method the first argument is actually self.

So this is a permission gate: only users in the "Instructors" group can call create, update, or destroy on CourseViewSet. Everyone else gets a 401 before any course data is touched. This is a clean way to enforce authorization rules in one reusable place instead of repeating the same if check inside every view method.

## 6. What is a serializer, and how does it fit the request/response cycle?

What a serializer is
Django stores data as Python objects (model instances) backed by database rows. But an API communicates over HTTP using JSON text. A serializer is the translator between those two worlds:

Serialization (Python object → JSON): turns a model instance into a dict-like structure DRF can render as JSON for the HTTP response.
Deserialization (JSON → Python object): takes incoming JSON from a request body, validates it, and turns it into Python data you can use to create/update a model instance.
Example from this project
user_serializer.py:


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'first_name', 'last_name', 'username', 'email', 'groups')
ModelSerializer is a shortcut — give it a model and a list of fields, and DRF auto-generates the conversion logic for those fields based on the model's field types. No need to manually write "take first_name and put it in the JSON as first_name."

nssuser_serializer.py does the same for the NssUser model, exposing slack_handle, github_handle, mentor, and user.

The CohortSerializer defined inside cohort_view.py:543 shows a more advanced version — it adds computed/derived fields that don't exist directly as database columns:


class CohortSerializer(serializers.ModelSerializer):
    courses = CohortCourseSerializer(many=True)
    attendance_sheet_url = serializers.SerializerMethodField()
    ...
    def get_attendance_sheet_url(self, obj):
        try:
            return obj.info.attendance_sheet_url
        except Exception:
            return ""
SerializerMethodField says "don't pull this from a model attribute directly — call this get_<field_name> method to compute the value." This is how the API can return convenience data (like coaches, students, is_instructor, URLs pulled from a related CohortInfo object) bundled into one JSON response, even though that data spans multiple tables/models.

How it fits the request/response cycle
Here's the full loop, using CohortViewSet.list as the concrete example (cohort_view.py:154-194):

Request comes in — a GET to /cohorts.
View method runs (list) — it queries the database: cohorts = Cohort.objects.all() (plus filtering/annotation).
Serialization — serializer = CohortSerializer(cohorts, many=True, context={'request': request}). This wraps the queryset of Cohort model instances.
.data triggers the conversion — serializer.data walks each Cohort object, pulls out the fields listed in Meta.fields, calls any SerializerMethodField methods, and produces plain Python dicts/lists.
Response is returned — Response(serializer.data, status=status.HTTP_200_OK). DRF's renderer turns those Python dicts into a JSON string and sets the Content-Type: application/json header.
For writes (POST/PUT), it runs in reverse: the JSON body lands in request.data, and (in this project, views often build model instances manually rather than using serializer.save(), e.g. cohort.name = request.data["name"]) — but the response is still built by handing the saved model instance back to a serializer so the client gets a consistent JSON shape back.

In short: models = database shape, serializers = JSON shape, and the serializer is the bridge that lets a view stay focused on "what data do I need" while the JSON formatting concern lives in one reusable place.



## 7. One model and what it represents

What a Django model is
A Django model is a Python class that defines the structure of a database table. Each class attribute that's a models.XField becomes a column, each instance of the class represents one row, and Django's ORM (Object-Relational Mapper) lets you query/create/update/delete rows using Python instead of writing raw SQL.


class Tag(models.Model):
    name = models.CharField(max_length=25)
This single class definition tells Django: "create a tag table with an auto-generated id primary key and a name text column (max 25 chars)." From then on, Tag.objects.all(), Tag.objects.create(name="python"), tag.delete(), etc. all work without SQL.

The models/ folder in this project is organized into sub-packages — coursework/ (courses, books, projects, assessments) and people/ (cohorts, NSS users, student teams, etc.) — reflecting the two major real-world domains the API tracks.

Example: Cohort model
cohort.py represents a student cohort — i.e., a specific class/group of students going through the NSS program together (e.g., "Web Dev Cohort 47"). Its fields:


class Cohort(models.Model):
    name = models.CharField(max_length=55, unique=True)
    slack_channel = models.CharField(max_length=55, unique=False)
    start_date = models.DateField(...)
    end_date = models.DateField(...)
    break_start_date = models.DateField(...)
    break_end_date = models.DateField(...)
    active = models.BooleanField(default=False)
Why the API needs to track this
A cohort is the central organizing unit of the whole platform — almost everything else in the system relates back to it:

Scheduling: start_date, end_date, and the break dates let the API answer "is this cohort currently active?" (there's even a helper method is_active_on_date() for exactly this).
Communication: slack_channel ties the cohort to a real Slack channel so the platform (or instructors) know where to post announcements for that group.
Curriculum tracking: through the courses relationship (CohortCourse), the API knows which course (client-side vs. server-side curriculum) each cohort is currently working through, and migrate() in cohort_view.py uses this to bulk-advance every student in a cohort to the next course's first project.
Roster management: properties like coaches and students (computed via NssUserCohort, the join table linking users to cohorts) let the API answer "who's in this cohort?" and "who's teaching it?" without storing redundant data — it's derived from relationships.
Personalization per user: is_instructor (a settable property, populated per-request) lets the API tell the current logged-in user whether they're an instructor for that specific cohort, which the frontend likely uses to show/hide instructor-only controls.
In short, Cohort exists because the program is structured around groups of students moving through curriculum together on a schedule — and nearly every feature (assignments, assessments, Slack integration, GitHub org assignment, course progression) needs to ask "which cohort does this belong to, and is that cohort currently active?"


## 8. Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View | | |
| ViewSet | | |

## 9. What replaces templates and why?