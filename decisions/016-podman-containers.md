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

## Verified (Sprint 1 spike, 2026-11-23)
- **Testcontainers:** works on rootless Podman 6.2.0 with `DOCKER_HOST=unix:///run/user/$UID/podman/podman.sock` (user `podman.socket` enabled). Ryuk works with no configuration. There is no `Testcontainers.Valkey` NuGet package: use a generic container.
- **Gotcha (machine-specific):** if the user session runs with `oom_score_adj=200`, `exec` in containers created via the API fails (`crun: write to /proc/self/oom_score_adj: Permission denied`, exit 126) and the Postgres module's `pg_isready` wait hangs forever. Fix: `.WithCreateParameterModifier(p => p.HostConfig.OomScoreAdj = 200)`.
- **Compose:** `podman compose up -d` works locally (external provider `docker-compose` 5.8.0 over the Podman socket). **VPS: Compose** (verified manually by the user). Quadlet not used.
- **CI:** the `ubuntu-24.04` runner ships Podman 4.9.3, Buildah 1.33.7 and Docker 28.0.4. Publish with `podman build` + `podman login` + `podman push` to GHCR using `GITHUB_TOKEN`.

## Open items (**to verify**)
- CI publish workflow was drafted but not run on GitHub.
- .NET Aspire AppHost with Podman (`ASPIRE_CONTAINER_RUNTIME=podman`), if Aspire is used instead of compose.
