
Reference: <https://cloud.google.com/artifact-registry/docs/docker/authentication#json-key>

If you have a service account key

```bash
cat <PATH-TO-KEY-FILE> | base64 | docker login -u _json_key_base64 --password-stdin \
https://us-central1-docker.pkg.dev
```