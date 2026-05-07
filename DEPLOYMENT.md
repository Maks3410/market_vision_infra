# CI/CD setup

The project uses three GitHub repositories:

- `market_vision_backend` builds and pushes `maksig/market-vision-backend`.
- `market-vision-frontend` builds and pushes `maksig/market-vision-frontend`.
- `market_vision_infra` stores the production compose file and deploys the server.

## Deployment flow

1. A push to `master` in backend or frontend starts that repository's workflow.
2. The workflow builds its Docker image and pushes `latest` to Docker Hub.
3. The workflow sends a `repository_dispatch` event to `market_vision_infra`.
4. The infra workflow uploads `docker-compose.prod.yml` to the server.
5. The infra workflow runs `docker compose pull` and `docker compose up -d`.
6. The infra workflow creates `/opt/market_vision/.env` from GitHub Secrets for future compose commands on the server.

## Files by repository

Put these files into `market_vision_infra`:

```text
.gitignore
.github/workflows/deploy.yml
docker-compose.prod.yml
DEPLOYMENT.md
```

Put these files into `market_vision_backend`:

```text
.dockerignore
.github/workflows/build-and-dispatch.yml
```

Put these files into `market-vision-frontend`:

```text
.dockerignore
.github/workflows/build-and-dispatch.yml
```

## Backend repository secrets

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `INFRA_REPOSITORY`
- `INFRA_REPO_DISPATCH_TOKEN`

`INFRA_REPOSITORY` example:

```text
your-github-username/market_vision_infra
```

## Frontend repository secrets

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `INFRA_REPOSITORY`
- `INFRA_REPO_DISPATCH_TOKEN`

## Infra repository secrets

- `SERVER_HOST`
- `SERVER_PORT`
- `SERVER_USER`
- `SERVER_SSH_KEY`
- `SERVER_APP_PATH`
- `DJANGO_SECRET_KEY`
- `ALLOWED_HOSTS`
- `CORS_ALLOWED_ORIGINS`
- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `POSTGRES_PUBLISHED_PORT`

## Dispatch token

`INFRA_REPO_DISPATCH_TOKEN` must be a GitHub token that can call `repository_dispatch` for the infra repository.

For a classic token:

- Use `repo` scope for a private infra repository.
- Use `public_repo` scope for a public infra repository.

For a fine-grained token:

- Give it access to `market_vision_infra`.
- Grant `Contents: Read and write`.

## Example infra secret values

`SERVER_APP_PATH`

```text
/opt/market_vision
```

`ALLOWED_HOSTS`

```text
your-domain.com,www.your-domain.com,server-ip,localhost,127.0.0.1
```

`CORS_ALLOWED_ORIGINS`

```text
https://your-domain.com,https://www.your-domain.com
```

`POSTGRES_PUBLISHED_PORT`

```text
5433
```

The app containers connect to Postgres inside Docker as `pgdb:5432`.
`POSTGRES_PUBLISHED_PORT` only controls which server host port is exposed outside Docker.

## Server requirements

- Docker installed.
- Docker Compose plugin installed.
- The deploy user can use Docker.
- The directory from `SERVER_APP_PATH` exists or can be created by the deploy user.
- Port `80` is open.
- The selected `POSTGRES_PUBLISHED_PORT` is not used by another service.
- Port `5555` is open only if Flower should be public.
- If the Docker Hub images are private, the server must be logged in to Docker Hub.

Run once on the server:

```bash
mkdir -p /opt/market_vision
docker --version
docker compose version
```

Useful server diagnostics:

```bash
cd /opt/market_vision
docker compose --env-file .env -f docker-compose.prod.yml ps
docker compose --env-file .env -f docker-compose.prod.yml logs backend --tail=200
```
