# Docker Compose Projects

A growing collection of Docker Compose setups for my projects. Each top-level folder contains the Compose configuration needed to build, run, and tear down one project as a complete multi-container environment with a single command.

The purpose of this repository is twofold. First, it is a working reference I use to run my own projects consistently on any machine. Second, it is a portfolio of how I design, structure, and operate containerised applications with Docker Compose.

---

## Table of Contents

1. [Repository Layout](#repository-layout)
2. [Projects](#projects)
3. [Prerequisites](#prerequisites)
4. [Quick Start](#quick-start)
5. [Skills and Concepts Demonstrated](#skills-and-concepts-demonstrated)
6. [Conventions Used Across Projects](#conventions-used-across-projects)
7. [Configuration and Environment Variables](#configuration-and-environment-variables)
8. [Everyday Command Reference](#everyday-command-reference)
9. [Networking](#networking)
10. [Data Persistence and Volumes](#data-persistence-and-volumes)
11. [Health Checks and Startup Order](#health-checks-and-startup-order)
12. [Profiles and Environment Overrides](#profiles-and-environment-overrides)
13. [Security Practices](#security-practices)
14. [Troubleshooting](#troubleshooting)
15. [Adding a New Project](#adding-a-new-project)
16. [Roadmap](#roadmap)
17. [License](#license)

---

## Repository Layout

Each project lives in its own folder at the top level and is self-contained. A project folder can be copied out of this repository and run on its own.

```
docker-compose/
|-- Cribsearch/
|-- Ecommerce/
|-- ShikaTicket/
`-- README.md
```

A typical project folder follows this shape. Not every project needs every file.

```
ProjectName/
|-- docker-compose.yml          Base definition of services, networks, and volumes
|-- docker-compose.override.yml Optional local development overrides
|-- .env.example                Template of required environment variables
|-- .env                        Real values (never committed)
|-- Dockerfile                  Image build instructions for custom services
|-- .dockerignore               Files excluded from the build context
`-- README.md                   Project-specific notes (optional)
```

---

## Projects

| Folder | Description |
| --- | --- |
| `Cribsearch` | Compose environment for the Cribsearch project. |
| `Ecommerce` | Compose environment for the Ecommerce project. |
| `ShikaTicket` | Compose environment for the ShikaTicket project. |

More folders will be added as I containerise additional projects. Each folder documents the services it runs, the ports it exposes, and the variables it requires.

---

## Prerequisites

You need the following installed on your machine:

- Docker Engine 24 or newer
- Docker Compose v2 (bundled with current Docker Desktop and available as the `docker compose` plugin on Linux)
- Git

Verify your installation:

```bash
docker --version
docker compose version
git --version
```

This repository uses the modern `docker compose` command (with a space), not the legacy standalone `docker-compose` binary.

---

## Quick Start

Clone the repository and move into the project you want to run:

```bash
git clone https://github.com/J-Thumi/docker-compose.git
cd docker-compose/<ProjectName>
```

Create your environment file from the template, then edit it with real values:

```bash
cp .env.example .env
```

Build and start every service in the background:

```bash
docker compose up -d --build
```

Check that everything is running and healthy:

```bash
docker compose ps
```

Follow the logs:

```bash
docker compose logs -f
```

Stop and remove the containers and network, keeping your data volumes:

```bash
docker compose down
```

Stop and remove everything including volumes (this deletes stored data):

```bash
docker compose down -v
```

---

## Skills and Concepts Demonstrated

This repository is meant to show practical, working knowledge of Docker Compose rather than toy examples. The areas it covers are:

**Multi-service orchestration.** Defining applications as several cooperating containers, such as an application server, a database, a cache, and supporting tools, and running them as one unit.

**Image building.** Using `build` contexts and Dockerfiles for custom services, and pulling pinned versions of official images for infrastructure services.

**Service networking.** Using user-defined networks so services reach each other by service name, and exposing only the ports that must be reachable from the host.

**Persistent storage.** Using named volumes for databases and bind mounts for local development, with a clear separation between data that must survive and files that should not.

**Configuration management.** Keeping configuration out of images and out of version control by using environment variables, `.env` files, and variable substitution with defaults.

**Startup ordering and health.** Using `healthcheck` and `depends_on` with `condition: service_healthy` so dependent services start only when their dependencies are genuinely ready, not merely started.

**Environment separation.** Using override files and profiles to run the same stack differently in development and production-like setups.

**Operational hygiene.** Restart policies, log handling, resource awareness, and clean teardown procedures.

**Reproducibility.** Pinning image versions and documenting every step so a new machine can reach a working environment from a fresh clone.

---

## Conventions Used Across Projects

To keep the projects consistent and easy to read, I follow these conventions.

**Compose file naming.** The main file is `docker-compose.yml`. Environment-specific changes go in separate override files rather than being edited into the base file.

**Pinned image tags.** Images use explicit version tags such as `postgres:16` instead of `latest`, so builds are repeatable and upgrades are deliberate.

**Descriptive service names.** Services are named for their role, for example `app`, `db`, `cache`, or `proxy`, because the service name doubles as its hostname on the network.

**Environment-driven configuration.** Ports, credentials, and hostnames come from variables with sensible defaults, using the `${VARIABLE:-default}` syntax where a default is appropriate.

**One concern per container.** Each container runs a single primary process.

**Explicit restart policy.** Long-running services declare `restart: unless-stopped` so they recover from crashes and host reboots without restarting after a deliberate stop.

**Comments where intent is not obvious.** Compose files include short comments explaining non-obvious decisions such as why a port is published or why a volume is mounted read-only.

**Secrets stay out of Git.** Real `.env` files are ignored. Only `.env.example` templates with placeholder values are committed.

---

## Configuration and Environment Variables

Every project that needs configuration ships a `.env.example` file listing each variable with a placeholder value. The workflow is always the same:

```bash
cp .env.example .env
# edit .env and set real values
```

Docker Compose automatically reads a file named `.env` in the project directory and uses it for variable substitution in the Compose file.

To see exactly what Compose will run after substitution and merging, render the resolved configuration:

```bash
docker compose config
```

This is the fastest way to catch a missing variable or a typo before starting anything. If a required variable is unset, Compose reports it rather than starting with a silent empty value.

---

## Everyday Command Reference

| Task | Command |
| --- | --- |
| Start in background | `docker compose up -d` |
| Start and rebuild images | `docker compose up -d --build` |
| Start only selected services | `docker compose up -d app db` |
| List services and status | `docker compose ps` |
| Follow all logs | `docker compose logs -f` |
| Follow one service's logs | `docker compose logs -f app` |
| Open a shell in a running service | `docker compose exec app sh` |
| Run a one-off command in a new container | `docker compose run --rm app <command>` |
| Restart a service | `docker compose restart app` |
| Rebuild one service without cache | `docker compose build --no-cache app` |
| Pull newer images | `docker compose pull` |
| Validate and print merged config | `docker compose config` |
| Stop and remove containers and network | `docker compose down` |
| Stop and also remove volumes | `docker compose down -v` |
| Show resource usage | `docker stats` |
| Remove unused images and cache | `docker system prune` |

Use `bash` instead of `sh` in the `exec` example if the image includes it.

---

## Networking

Compose creates a default network for each project, and every service joins it. Within that network, services reach each other by service name. For example, an application container connects to its database using the hostname `db` and the database's internal port, regardless of what is published to the host.

Key points I apply:

- **Internal versus published ports.** A service listening on a port inside the network does not need that port published to the host. Only entry points, such as a web server or an API, are published. Databases and caches are normally left unpublished in production-style setups and published only for local development when I need to connect a client.
- **Port mapping format.** Ports are written as `HOST:CONTAINER`. Changing the host side avoids conflicts with software already running on your machine.
- **Custom networks.** Where isolation matters, services are placed on separate named networks so that, for example, a database is reachable by the application but not by a public-facing proxy.

If a port is already in use on your machine, change the host side of the mapping in your `.env` file or Compose file and run `docker compose up -d` again.

---

## Data Persistence and Volumes

Containers are disposable. Anything that must survive a container being removed lives in a volume.

- **Named volumes** are used for databases and other stateful services. Docker manages their location, and they persist across `docker compose down`. They are removed only when you run `docker compose down -v` or delete them explicitly.
- **Bind mounts** map a folder from your machine into a container. I use them during development so code changes appear inside the container immediately, and for supplying configuration files, often mounted read-only with the `:ro` flag.

Useful commands:

```bash
docker volume ls
docker volume inspect <volume_name>
docker volume rm <volume_name>
```

To back up a named volume, run a temporary container that mounts the volume and archives its contents to a bind-mounted folder on the host.

---

## Health Checks and Startup Order

A container being started does not mean the software inside is ready. Databases in particular take time to accept connections. To avoid race conditions, projects in this repository use health checks:

```yaml
services:
  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    depends_on:
      db:
        condition: service_healthy
```

With this pattern, the `app` service waits until `db` reports healthy before it starts. The `$$` escapes the dollar sign so that the variable is evaluated inside the container rather than substituted by Compose.

You can inspect health status at any time:

```bash
docker compose ps
docker inspect --format='{{json .State.Health}}' <container_name>
```

This example is illustrative of the pattern. The exact health checks differ per project depending on the services involved.

---

## Profiles and Environment Overrides

Two mechanisms let one project serve several purposes without duplicating files.

**Override files.** Compose automatically merges `docker-compose.override.yml` on top of `docker-compose.yml` when both exist. I use this for development conveniences such as bind-mounted source code, extra published ports, and debug settings. To run without the override, specify files explicitly:

```bash
docker compose -f docker-compose.yml up -d
```

To layer a different environment on top of the base file:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

**Profiles.** Services can be tagged with a profile so they only start when requested. This is useful for optional tools such as admin interfaces, test runners, or monitoring:

```yaml
services:
  adminer:
    image: adminer
    profiles: ["tools"]
```

```bash
docker compose --profile tools up -d
```

Where a project uses overrides or profiles, its folder documents which ones exist and what they do.

---

## Security Practices

Even for personal projects I try to follow habits that carry over to real deployments:

- **No secrets in Git.** Credentials live in `.env`, which is excluded through `.gitignore`. Only placeholder values appear in `.env.example`.
- **Change default credentials.** Every default password in an example file is a placeholder that must be replaced before running anything beyond a local test.
- **Minimal exposure.** Only the ports that users or clients genuinely need are published to the host. Internal services stay on the internal network.
- **Pinned and official images.** Infrastructure services use official images at specific versions, which reduces surprise changes and makes vulnerabilities easier to track.
- **Read-only mounts.** Configuration files are mounted read-only wherever the container does not need to write to them.
- **Lean build contexts.** A `.dockerignore` file keeps local files, caches, and credentials out of image builds.
- **Non-root where practical.** Custom images run as an unprivileged user when the application allows it.

Services published on a machine with a public IP address should be placed behind a reverse proxy with TLS and appropriate firewall rules. The configurations here are intended primarily for development and demonstration.

---

## Troubleshooting

**A port is already allocated.** Another process or container is using that host port. Change the host side of the port mapping, or stop the conflicting process. On Linux or macOS, `lsof -i :<port>` shows what is using it.

**A service keeps restarting.** Read its logs with `docker compose logs <service>`. The most common causes are a missing environment variable, a wrong database hostname, or a failed dependency.

**The application cannot reach the database.** Confirm the application uses the service name as the hostname, not `localhost`. Inside a container, `localhost` refers to that container itself.

**Changes to the Dockerfile or code are not picked up.** Rebuild with `docker compose up -d --build`. If caching is the problem, use `docker compose build --no-cache <service>`.

**Environment variables seem to be ignored.** Run `docker compose config` to see the resolved values. Confirm the `.env` file is in the same directory you run the command from and that there are no stray spaces around the `=` sign.

**Database data is gone after teardown.** You most likely ran `docker compose down -v`, which removes volumes. Use plain `docker compose down` to keep data.

**Permission denied on a mounted folder.** The user inside the container does not own the host folder. Adjust ownership or the container's user setting.

**Out of disk space.** Remove unused images, containers, and build cache with `docker system prune`. Add `-a` to remove all unused images, and be aware that this also removes images you may want to keep.

---

## Adding a New Project

When I containerise another project, I follow this checklist so the repository stays consistent:

1. Create a new top-level folder named after the project.
2. Add a `docker-compose.yml` with pinned image versions and descriptive service names.
3. Add a `Dockerfile` and `.dockerignore` for any custom-built service.
4. Move all configuration into environment variables and provide a `.env.example` with placeholders.
5. Add health checks to stateful services and use `depends_on` conditions where order matters.
6. Use named volumes for data that must persist.
7. Run `docker compose config` to validate the file.
8. Test from a clean state: `docker compose down -v` followed by `docker compose up -d --build`.
9. Add a row to the Projects table in this README and document the services, ports, and required variables.
10. Confirm no secrets are committed before pushing.

---

## Roadmap

Planned improvements as the repository grows:

- Add a project-level README to every folder describing services, ports, and variables
- Add reverse proxy examples with automatic TLS
- Add monitoring and logging stacks that can be attached to any project through profiles
- Add continuous integration that validates every Compose file with `docker compose config`
- Add backup and restore scripts for stateful services
- Containerise additional projects

---

