FROM node:lts-slim@sha256:55fb5f5a527ea2c5c4a95f963e48ef90aa38c7b346054fdc8ee9957a2b55d205

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci --omit=dev && npm cache clean --force

COPY firebase.json ./

ENV FIREBASE_PROJECT=demo-project

EXPOSE 9099

HEALTHCHECK CMD curl -f localhost:9099

CMD ["sh", "-c", "exec node_modules/.bin/firebase emulators:start --only auth --project $FIREBASE_PROJECT"]
