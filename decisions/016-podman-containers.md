# 016 — Podman as the container runtime

**Status:** Decided (2026-11-23)

## Context
The backend (API, worker) and PostgreSQL run in containers, locally and on the Hetzner VPS ([003](003-audio-storage-cdn.md)).

## Options considered
- **Podman (chosen).**
- Docker: the default assumed by the original draft and by `vibra-backend/CLAUDE.md`.

## Choice
**Podman** for the .NET backend, the database and Valkey (confirmed 2026-11-23).

## Consequences
- Container files stay OCI-standard (`Containerfile`/`Dockerfile`, images), so they are portable.
- **Drift with `vibra-backend/CLAUDE.md`:** it says `docker compose up -d` and "Testcontainers needs Docker". Per this repo's rules the sibling is not edited from here; **ask which wins** and update it from the backend repo.

## Open items (all **to verify**, Sprint 1 spike)
- Testcontainers against Podman (Docker-compatible socket / `DOCKER_HOST`; the Ryuk reaper may need configuration).
- Compose workflow: `podman compose` / `podman-compose` locally; on the VPS, Podman Quadlet (systemd) or compose.
- .NET Aspire AppHost with Podman as the container runtime, if Aspire is used instead of compose.
- CI: GitHub Actions runners ship Docker; Podman availability for the container publish step.
