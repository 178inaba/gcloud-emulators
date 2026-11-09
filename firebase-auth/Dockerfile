FROM node:lts-slim@sha256:5adad64f9a1fca2d51972caa65983634c8a774eb300d72b20f7f2a0f60368072

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
