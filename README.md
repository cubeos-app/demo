# CubeOS Demo

CI/CD pipeline for [demo.cubeos.app](https://demo.cubeos.app).

Builds the CubeOS dashboard with `VITE_DEMO_MODE=true` and deploys it to the DMZ
via AWX. Demo mode source code lives in the
[dashboard repo](https://gitlab.nuclearlighters.net/products/cubeos/dashboard).

## How it works

1. **Build** — Clones the dashboard repo, runs `npm run build` with `VITE_DEMO_MODE=true`
2. **Package** — Builds a Docker image from the nginx base + built assets, pushes to `ghcr.io/cubeos-app/demo:latest`
3. **Deploy** — Triggers AWX job template 42 to pull and restart on both DMZ nodes

## Triggers

### Automatic (push to main)
Any push to this repo's `main` branch runs the full pipeline.

### Cross-repo trigger (dashboard pushes)
The dashboard repo can trigger a demo rebuild when demo-related code changes.
To set this up:

1. Create a pipeline trigger token in this repo:
   **Settings > CI/CD > Pipeline trigger tokens > Add trigger**
2. Add the token as `DEMO_TRIGGER_TOKEN` CI variable in the dashboard repo
3. Add this to the dashboard's `.gitlab-ci.yml`:
   ```yaml
   trigger:demo:
     stage: deploy
     trigger:
       project: products/cubeos/demo
       branch: main
     rules:
       - if: $CI_COMMIT_BRANCH == "main"
         changes:
           - src/composables/useDemoMode.js
           - src/mock/**/*
   ```

### Scheduled pipeline (optional)
Configure a daily schedule in **CI/CD > Schedules** to keep the demo in sync
with the latest dashboard changes, even without explicit triggers.

## CI Variables

| Variable | Level | Purpose |
|----------|-------|---------|
| `GHCR_USER` | Group | GitHub Container Registry username |
| `GHCR_TOKEN` | Group | GitHub Container Registry token |
| `AWX_HOST` | Group | AWX API base URL |
| `AWX_TOKEN` | Group | AWX API bearer token |
| `AWX_JOB_TEMPLATE_ID` | Project | AWX job template ID (42) |
