# Patcher: Uptime Kuma Rootless ARM64 Build (+ Custom Patch)

This folder is a minimal repository you can push to GitHub to build and publish a patched, rootless Uptime Kuma Docker image for ARM64.

## What This Patch Changes

- Docker image build
  - Build frontend during image build (`npm run build`) so `dist/` is present.
  - Install full deps first, then `npm prune --omit=dev` to slim the final layer.
  - Ensure files are copied as user `node` to avoid permission issues.
  - Explicitly remove `cloudflared` from the final image (even if present in base).
  - Do not install `cloudflared` in the base image by default.
- Environment-driven controls (white‑label hosting)
  - `UPTIMEKUMA_WH_MAX_MONITORS`: caps total monitors; blocks adding beyond the cap.
  - `UPTIMEKUMA_WH_MIN_INTERVAL`: overrides minimum monitor interval (seconds).
  - `UPTIMEKUMA_WH_MAX_KEEP_DAYS`: caps data retention; disallows infinite retention.
  - `UPTIMEKUMA_WH_USERNAME` / `UPTIMEKUMA_WH_PASSWORD`: optional auto-create user on first start.
- Data retention enforcement
  - UI surfaces `whMaxKeepDays` to constrain inputs.
  - Server validates `keepDataPeriodDays` against the cap.
  - Cleanup job uses an effective (capped) retention when purging data.
- Monitor validation
  - Minimum interval uses `UPTIMEKUMA_WH_MIN_INTERVAL` when set; otherwise default.
- Setup defaults
  - If no DB config exists, default to SQLite without blocking setup.

## GitHub Action (in this folder)

- Path: `.github/workflows/docker-rootless-arm64.yml`
- Manual trigger with required input `tag` (e.g., `v1.23.4`).
- Steps:
  1. Clone `louislam/uptime-kuma` at the provided tag.
  2. Apply `uncommitted-changes.patch` from this folder.
  3. Build `docker/dockerfile` target `rootless` for `linux/arm64` on an ARM64 runner.
  4. Push to `ghcr.io/wiesinghilker/uptimekuma:latest`.

## Usage

1. Create a new GitHub repository from the `patcher/` contents.
2. Enable GitHub Packages permissions (Packages: write) for the repository.
3. Run the workflow manually and provide an upstream tag (e.g. `v1.23.4`).
4. The built image will be available at `ghcr.io/wiesinghilker/uptimekuma:latest`.

## Notes

- If GHCR visibility should be public, adjust the package settings in GitHub.
- If the patch fails to apply (e.g., upstream changes), the workflow will fail and report rejects.
- To pin a different tag instead of `:latest`, adjust the `tags:` in the workflow.
