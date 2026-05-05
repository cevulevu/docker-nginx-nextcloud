# Nextcloud Docker Stack

A self-hosted Nextcloud deployment using Docker Compose with a custom Nginx reverse proxy, MariaDB, and Redis.

## Architecture

```
[Browser]
    |
    | :8080
    ↓
[Nginx - custom image]      ← reverse proxy, single entry point
    |
    | proxy_pass http://app:80 (internal)
    ↓
[Nextcloud]
    |              |
    | dbnet        | redisnet
    ↓              ↓
[MariaDB]       [Redis]
```

All services communicate through isolated Docker networks. MariaDB and Redis are not exposed outside of Docker.

## Services

| Service | Image | Role |
|---|---|---|
| proxy | custom (nginx:alpine) | Reverse proxy, entry point |
| app | nextcloud | Main application |
| db | mariadb:10.6 | Database |
| redis | redis:alpine | Caching |

## Custom Nginx Image

Instead of using a plain `nginx` image, a custom image is built from `nginx-custom/`:

```
nginx-custom/
  Dockerfile          # FROM nginx:alpine, removes default.conf, copies custom config
  nginx.conf          # Main config, includes conf.d/
  conf.d/
    nextcloud.conf    # VirtualHost — proxies requests to Nextcloud container
```

This approach bakes the config into the image, making it self-contained and deployable to any environment without needing files present on the host.

## Networks

| Network | Connects |
|---|---|
| proxynet | proxy ↔ app |
| dbnet | app ↔ db |
| redisnet | app ↔ redis |

## Healthcheck

MariaDB has a healthcheck configured. Nextcloud waits for the database to be healthy before starting, preventing connection errors on first boot.

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

## Setup

1. Clone the repository:
```bash
git clone https://github.com/cevulevu/docker-nginx-nextcloud.git
cd docker-nginx-nextcloud
```

2. Create the `.env` file (never commit this):
```bash
cp .env.example .env
# edit .env with your values
```

3. Start the stack:
```bash
docker compose up -d --build
```

4. Access Nextcloud at `http://localhost:8080`

## Environment Variables

| Variable | Description |
|---|---|
| MYSQL_ROOT_PASSWORD | MariaDB root password |
| MYSQL_PASSWORD | Nextcloud DB user password |
| MYSQL_DATABASE | Database name |
| MYSQL_USER | Database user |
