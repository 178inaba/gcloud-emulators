FROM node:lts-slim@sha256:3638d9a6fe4030bd716be989438248074489337ba3275657f93595428be4fc03

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
