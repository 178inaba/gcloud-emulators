# Google Cloud Emulators

[![Publish Firestore Emulator](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-firestore.yml/badge.svg)](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-firestore.yml)
[![Publish Pub/Sub Emulator](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-pubsub.yml/badge.svg)](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-pubsub.yml)
[![Publish Firebase Auth Emulator](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-firebase-auth.yml/badge.svg)](https://github.com/178inaba/gcloud-emulators/actions/workflows/publish-firebase-auth.yml)

## Firestore / Datastore Emulator

```console
$ docker run -d --name datastore-emulator -e DATABASE_MODE=datastore-mode -p 8080:8080 ghcr.io/178inaba/firestore-emulator
```

### GitHub Actions

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      datastore-emulator:
        image: ghcr.io/178inaba/firestore-emulator
        ports:
          - "8080:8080"
        env:
          DATABASE_MODE: datastore-mode
    env:
      DATASTORE_EMULATOR_HOST: localhost:8080
    steps:
      - name: Test
        run: |
          # Replace this with running tests for your app.
          curl -f localhost:8080
```

### Docker Compose

```yaml
services:
  firestore:
    image: ghcr.io/178inaba/firestore-emulator
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "localhost:8080"]

  datastore:
    image: ghcr.io/178inaba/firestore-emulator
    ports:
      - "8081:8080"
    environment:
      DATABASE_MODE: datastore-mode
    healthcheck:
      test: ["CMD", "curl", "-f", "localhost:8080"]

  app:
    build: .
    environment:
      FIRESTORE_EMULATOR_HOST: firestore:8080
      DATASTORE_EMULATOR_HOST: datastore:8080
    depends_on:
      firestore:
        condition: service_healthy
      datastore:
        condition: service_healthy
```

### Environment Variables

#### `DATABASE_MODE`

The database mode to start the Firestore Emulator in.

The valid options are:

- `firestore-native` (default): start the emulator in Firestore Native
- `datastore-mode`: start the emulator in Datastore Mode

## Pub/Sub Emulator

```console
$ docker run -d --name pubsub-emulator -p 8085:8085 ghcr.io/178inaba/pubsub-emulator
```

### GitHub Actions

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      pubsub-emulator:
        image: ghcr.io/178inaba/pubsub-emulator
        ports:
          - "8085:8085"
    env:
      PUBSUB_EMULATOR_HOST: localhost:8085
    steps:
      - name: Test
        run: |
          # Replace this with running tests for your app.
          curl -f localhost:8085
```

### Docker Compose

```yaml
services:
  pubsub:
    image: ghcr.io/178inaba/pubsub-emulator
    ports:
      - "8085:8085"
    healthcheck:
      test: ["CMD", "curl", "-f", "localhost:8085"]

  app:
    build: .
    environment:
      PUBSUB_EMULATOR_HOST: pubsub:8085
    depends_on:
      pubsub:
        condition: service_healthy
```

## Firebase Auth Emulator

Unlike the other images, this one is built from the [`firebase-tools`](https://www.npmjs.com/package/firebase-tools) npm package rather than the gcloud CLI, because the Firebase Auth emulator ships with `firebase-tools`.

```console
$ docker run -d --name firebase-auth-emulator -p 9099:9099 ghcr.io/178inaba/firebase-auth-emulator
```

The Emulator UI is disabled so that starting the container requires no network access. To enable it, mount your own `firebase.json` over `/app/firebase.json`.

### GitHub Actions

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      firebase-auth-emulator:
        image: ghcr.io/178inaba/firebase-auth-emulator
        ports:
          - "9099:9099"
    env:
      FIREBASE_AUTH_EMULATOR_HOST: localhost:9099
    steps:
      - name: Test
        run: |
          # Replace this with running tests for your app.
          curl -f localhost:9099
```

### Docker Compose

```yaml
services:
  firebase-auth:
    image: ghcr.io/178inaba/firebase-auth-emulator
    ports:
      - "9099:9099"
    healthcheck:
      test: ["CMD", "curl", "-f", "localhost:9099"]

  app:
    build: .
    environment:
      FIREBASE_AUTH_EMULATOR_HOST: firebase-auth:9099
    depends_on:
      firebase-auth:
        condition: service_healthy
```

### Environment Variables

#### `FIREBASE_PROJECT`

The project ID to start the Firebase Auth Emulator with.

The default is `demo-project`. Firebase treats project IDs prefixed with `demo-` as offline-only, so the emulator never reaches real Firebase services.

Clients may connect with any project ID regardless of this value, so you usually do not need to change it.

## License

[MIT](LICENSE)

## Author

Masahiro Furudate (a.k.a. [178inaba](https://github.com/178inaba))  
<178inaba.git@gmail.com>
