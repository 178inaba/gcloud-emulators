FROM node:lts-slim@sha256:235600a8101ab264e117b1768e925532262668dc9b581ef1dd7d96ced463b8e7

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
