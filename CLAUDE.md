# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Docker container images for Google Cloud emulators (Firestore/Datastore, Pub/Sub, and Firebase Auth). This repository contains no application code — only Dockerfiles and CI/CD workflows.

Firestore and Pub/Sub share a `google-cloud-cli` base image. Firebase Auth does not — its emulator ships with the `firebase-tools` npm package, so that image is built on `node` instead.

## Build and Test

No build system or test framework. Everything runs through Docker and GitHub Actions.

### Firestore Emulator

```bash
# Build
docker build -t firestore-emulator firestore/

# Run (Firestore Native mode)
docker run -p 8080:8080 firestore-emulator

# Run (Datastore mode)
docker run -e DATABASE_MODE=datastore-mode -p 8080:8080 firestore-emulator

# Health check
curl -f localhost:8080
```

### Pub/Sub Emulator

```bash
# Build
docker build -t pubsub-emulator pubsub/

# Run
docker run -p 8085:8085 pubsub-emulator

# Health check
curl -f localhost:8085
```

### Firebase Auth Emulator

```bash
# Build
docker build -t firebase-auth-emulator firebase-auth/

# Run
docker run -p 9099:9099 firebase-auth-emulator

# Run with a different project ID
docker run -e FIREBASE_PROJECT=my-own-project -p 9099:9099 firebase-auth-emulator

# Health check
curl -f localhost:9099
```

## Repository Structure

- `firestore/Dockerfile` — Firestore emulator image (port 8080)
- `pubsub/Dockerfile` — Pub/Sub emulator image (port 8085)
- `firebase-auth/` — Firebase Auth emulator image (port 9099)
  - `Dockerfile` — `node:lts-slim` base pinned by digest
  - `package.json` / `package-lock.json` — pins `firebase-tools`, which provides the emulator
  - `firebase.json` — host/port, Emulator UI, and single-project settings (no CLI flags exist for these)
- `.github/workflows/` — CI/CD workflows
  - `auto-merge-dependabot.yml` — Auto-approve and auto-merge Dependabot PRs using GitHub App token
  - `reusable-publish-emulator.yml` — Reusable (`workflow_call`) workflow: build, push, attest, and smoke-test one emulator image; shared by the three `publish-*.yml` callers below
  - `publish-firestore.yml` — Calls `reusable-publish-emulator.yml` to build and publish the Firestore image on merge to main (only on `firestore/` path changes)
  - `publish-pubsub.yml` — Calls `reusable-publish-emulator.yml` to build and publish the Pub/Sub image on merge to main (only on `pubsub/` path changes)
  - `publish-firebase-auth.yml` — Calls `reusable-publish-emulator.yml` to build and publish the Firebase Auth image on merge to main (only on `firebase-auth/` path changes)
  - `test.yml` — Build and health-check test every emulator image on PR
- `examples/docker-compose/` — Example Go application using Firestore/Datastore

## CI/CD

- **PR tests**: Build image, start container, verify HTTP 200 via curl health check
- **Publish**: On merge to main, push multi-platform (amd64/arm64) images to both Docker Hub (`178inaba/*`) and GHCR (`ghcr.io/178inaba/*`). Each `publish-*.yml` is a thin caller (own path filters, permissions, and per-image inputs) around the shared `reusable-publish-emulator.yml` workflow, which does the actual build/push/attest/smoke-test; the three callers stay separate files so each keeps its own `on.push.paths` and doesn't cross-trigger the others
- **Image tags**: `latest`, version number, commit SHA
  - gcloud-based images: version read from the first line of the Dockerfile (e.g., `555.0.0`)
  - Firebase Auth: the `firebase-tools` version read from `firebase-auth/package-lock.json`
- **Dependabot**: Daily automated updates for base images (gcloud version tags, and the `node` digest for Firebase Auth), npm packages (`firebase-tools`), and GitHub Actions

## Important Notes

- Firestore Dockerfile uses `sh -c` to run the command because it needs shell expansion for the `DATABASE_MODE` environment variable
- Pub/Sub emulator requires the `beta` subcommand: `gcloud beta emulators pubsub start`
- Do not add a JRE to the Firebase Auth image. The Firestore and Pub/Sub emulators run downloaded JARs, but the auth emulator is implemented in `firebase-tools` itself
- Firebase Auth image disables the Emulator UI in `firebase.json`. Enabling it makes `firebase-tools` download a UI zip at container start, which would make startup depend on network access
- The Firebase Auth base image is pinned to the `node:lts-slim` digest rather than a patch version tag, so that Dependabot tracks LTS releases only. A patch-pinned tag would let it bump to non-LTS majors
- Publish workflows use path filters. Each one watches its own Dockerfile-related paths, its own workflow file, and the shared `reusable-publish-emulator.yml` workflow (so a change to the shared logic republishes all three images). The gcloud-based ones only watch their Dockerfile; `publish-firebase-auth.yml` also watches `package.json`, `package-lock.json`, and `firebase.json`, since its version and base image are not in the Dockerfile
- Every `docker/build-push-action` step must set a unique, stable `scope=` on both `cache-from` and `cache-to`. The GitHub Actions cache backend defaults to a single shared scope across all steps, so an unscoped step overwrites another step's cache index instead of coexisting with it. `reusable-publish-emulator.yml` reuses its existing `image-name` input as the scope (already required and unique per caller), so the three images keep separate cache indexes without a dedicated scope input
