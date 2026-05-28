# Tech Stack (AI)

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| `.env` | `/` (root) | `POSTGRES_DB` | Name of the PostgreSQL database | Read by the `database` and `postgres_exporter` services via `env_file` |
| `.env` | `/` (root) | `POSTGRES_USER` | PostgreSQL login username | Used in the `pg_isready` healthcheck and the `DATA_SOURCE_NAME` connection string |
| `.env` | `/` (root) | `DATA_SOURCE_NAME` | Full PostgreSQL connection URL for the exporter | Consumed by `postgres_exporter` to connect and scrape DB metrics |
| `docker-compose.yml` | `/` (root) | `image` | Docker image + version for each service (e.g. `postgres:16`) | Tells Docker which container image to pull when starting a service |
| `docker-compose.yml` | `/` (root) | `ports` | Host-to-container port mappings (e.g. `5432:5432`) | Exposes a container's internal port on the host machine |
| `docker-compose.yml` | `/` (root) | `depends_on` | Startup order / health dependencies between services | Ensures dependent services (e.g. `api`) wait until `database` is healthy |
| `valkey/docker-compose.yml` | `/valkey` | `image` | Valkey image to run (`valkey/valkey:latest`) | Specifies the in-memory data store container to pull |
| `valkey/docker-compose.yml` | `/valkey` | `ports` | Maps container port `6379` to host port `6379` | Allows the API and other services to reach Valkey on the default Redis-compatible port |
| `valkey/docker-compose.yml` | `/valkey` | `command` | Valkey server startup flags (persistence + log level) | Configures save intervals and verbosity without needing a separate config file |
| `learn-ops-client/.env` | `learn-ops-client/` | `REACT_APP_API_URI` | Base URL the React app uses to call the Django API | Injected at build time by Create React App into all `fetch`/`axios` calls |
| `learn-ops-client/.env` | `learn-ops-client/` | `REACT_APP_ENV` | Tells the front-end which environment it's running in | Used to toggle dev-only features or logging |
| `learn-ops-client/.env` | `learn-ops-client/` | `CHOKIDAR_USEPOLLING` | Forces file-watcher polling instead of native events | Required for hot-reload to work inside Docker on Windows/WSL |
| `learn-ops-api/.env` | `learn-ops-api/` | `LEARNING_GITHUB_CALLBACK` | GitHub OAuth redirect URL after a user authorizes the app | Registered in the GitHub OAuth app settings; must match exactly |
| `learn-ops-api/.env` | `learn-ops-api/` | `LEARN_OPS_DJANGO_SECRET_KEY` | Django's cryptographic signing key | Used for sessions, CSRF tokens, and password reset links |
| `learn-ops-api/.env` | `learn-ops-api/` | `LEARN_OPS_ALLOWED_HOSTS` | Comma-separated list of hostnames Django will accept requests from | Prevents HTTP Host header attacks |
| `learn-ops-api/.env` | `learn-ops-api/` | `VALKEY_HOST` | Hostname of the Valkey (Redis-compatible) service | Tells Django where to connect for caching and session storage |
| `learn-ops-api/.env` | `learn-ops-api/` | `SLACK_BOT_TOKEN` | Bot token for posting Slack notifications from the API | Used by the Slack SDK to authenticate outgoing messages |
| `learn-ops-api/.env` | `learn-ops-api/` | `DEBUG` | Toggles Django debug mode on or off | When `True`, shows detailed error pages; must be `False` in production |
| `learn-ops-api/.env.template` | `learn-ops-api/` | *(same keys as `.env`)* | Placeholder copy of `.env` checked into version control | Copy to `.env` and fill in real values — never commit the real `.env` |
| `service-monarch/.env` | `service-monarch/` | `GH_PAT` | GitHub Personal Access Token for the Monarch service | Authenticates GitHub API calls (reading repos, webhook events, etc.) |
| `service-monarch/.env` | `service-monarch/` | `SLACK_WEBHOOK_URL` | Incoming webhook URL for posting messages to a Slack channel | Used by Monarch to send automated Slack notifications |
| `service-monarch/.env` | `service-monarch/` | `SLACK_TOKEN` | Slack bot OAuth token | Authenticates Slack API calls that need more than a simple webhook |
| `pytest.ini` | `learn-ops-api/` | `DJANGO_SETTINGS_MODULE` | Tells pytest-django which settings file to load for tests | Must point to the correct settings so the test DB is configured properly |
| `pytest.ini` | `learn-ops-api/` | `testpaths` | Directory pytest scans for test files | Limits the test run to `LearningAPI/tests` so unrelated files are not picked up |
| `pytest.ini` | `learn-ops-api/` | `addopts` | Default CLI flags applied to every test run | Sets options like `--reuse-db`, `--nomigrations`, `-v`, and `--tb=short` |
| `nginx.api.conf` | `learn-ops-api/config/` | `server_name` | Domain the Nginx virtual host responds to (`learningapi.nss.team`) | Routes incoming HTTPS traffic to the correct block for the production API |
| `nginx.api.conf` | `learn-ops-api/config/` | `proxy_pass` | Upstream address Nginx forwards requests to (`http://127.0.0.1:8000`) | Connects the public-facing Nginx to the Django Gunicorn process |
| `nginx.api.conf` | `learn-ops-api/config/` | `ssl_certificate` | Path to the TLS certificate file managed by Certbot | Enables HTTPS on port 443 for the production API domain |
| `nginx.client.conf` | `learn-ops-api/config/` | `server_name` | Domain the Nginx virtual host responds to (`learning.nss.team`) | Routes incoming HTTPS traffic to the production React client |
| `nginx.client.conf` | `learn-ops-api/config/` | `root` | Filesystem path to the built React app | Nginx serves the static `index.html` and assets from this directory |
| `nginx.client.conf` | `learn-ops-api/config/` | `ssl_certificate` | Path to the TLS certificate file managed by Certbot | Enables HTTPS on port 443 for the production client domain |

### 1b. How to Start It

All commands are run from the root of the `learn-ops-infrastructure` directory using `make <command>`.

#### First-time setup

```bash
make setup    # Run this once before anything else
make up       # Then bring the full system up
```

#### All Makefile commands

| Command | What it runs | What it does |
|---|---|---|
| `make setup` | `./scripts/setup.sh` | **First-time only.** Installs prerequisites, creates the Docker network, and configures the environment so the other commands work. |
| `make doctor` | `./scripts/setup.sh --doctor` | Checks your machine for missing dependencies or misconfiguration without changing anything. Run this if something seems broken. |
| `make up` | `docker compose up --build -d` | Builds all service images and starts every service in the background (database, api, client, prometheus, grafana, postgres_exporter). |
| `make up-api` | `docker compose up --build -d api` | Starts **only** the `api` service. Useful when you're only working on back-end changes and don't want to run the full stack. |
| `make up-client-api` | `docker compose up --build -d api client` | Starts the `api` and `client` services together. Use when you need the front-end and back-end but not the monitoring stack. |
| `make down` | `docker compose down` | Stops and removes all running containers, but **keeps your data volumes** (the database data survives). |
| `make restart` | `down` then `up --build -d` | Full stop + rebuild + start. Use when you've made config changes that require a clean restart. |
| `make logs` | `docker compose logs -f` | Streams live log output from all containers. Press `Ctrl+C` to stop watching. |
| `make ps` | `docker compose ps` | Shows the current status of every container (running, stopped, health state). |
| `make reset` | `docker compose down -v --remove-orphans` | **Destructive.** Stops all containers AND deletes all data volumes. Use when you need a completely clean slate (wipes the database). |

#### Key differences between the `up` variants

- `make up` — full stack, every service
- `make up-client-api` — front-end + back-end only (no monitoring)
- `make up-api` — back-end only (fastest, for API-only work)

#### `down` vs `reset`

| | `make down` | `make reset` |
|---|---|---|
| Stops containers | ✅ | ✅ |
| Removes containers | ✅ | ✅ |
| **Deletes data volumes** | ❌ (data kept) | ✅ **(data lost)** |
| Removes orphan containers | ❌ | ✅ |

### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
| client (React front-end) | 3000 | http://localhost:3000 |
| api (Django back-end) | 8000 | http://localhost:8000 |
| prometheus | 9090 | http://localhost:9090 |
| grafana | 3001 | http://localhost:3001 |
| postgres_exporter | 9187 | http://localhost:9187/metrics |
| database (PostgreSQL) | 5432 | `postgresql://localhost:5432` — connect with a DB client, not a browser |
| valkey | 6379 | TCP only — connect with a Redis-compatible client (e.g. `redis-cli -p 6379`) |

### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| | | |
| | | |
| | | |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| | | |
| | | |
| | | |

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| | | |

## 3. System Overview