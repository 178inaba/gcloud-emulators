FROM gcr.io/google.com/cloudsdktool/google-cloud-cli:567.0.0-emulators

EXPOSE 8085

HEALTHCHECK CMD curl -f localhost:8085

CMD ["gcloud", "beta", "emulators", "pubsub", "start", "--host-port=0.0.0.0:8085"]
