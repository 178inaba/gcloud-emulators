FROM gcr.io/google.com/cloudsdktool/google-cloud-cli:560.0.0-emulators

ENV DATABASE_MODE=firestore-native

EXPOSE 8080

HEALTHCHECK CMD curl -f localhost:8080

CMD ["sh", "-c", "exec gcloud emulators firestore start --database-mode $DATABASE_MODE --host-port 0.0.0.0:8080"]
