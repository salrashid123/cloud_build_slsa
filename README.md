
## Deterministic builds with Bazel and Cloud Build SLSA provenance

Sample which demonstrates end-to-end build+push of a container image which includes a verifiable GCP [build provenance statement](https://cloud.google.com/build/docs/securing-builds/generate-validate-build-provenance)

eg

```
source -> 
   git push -> 
      cloud_build_trigger -> 
         deterministic build -> 
            push container image to artifact registry -> 
               signed SLSA statement
```


### Setup

You must first configure a git repo to use for this test which has a connection to GCP cloud build:

see [Create and manage build triggers]()https://cloud.google.com/build/docs/automating-builds/create-manage-triggers)


```bash
export PROJECT_ID=`gcloud config get-value core/project`
export PROJECT_NUMBER=`gcloud projects describe $PROJECT_ID --format='value(projectNumber)'`
export CLOUD_BUILD_SERVICE_AGENT="service-$PROJECT_NUMBER@gcp-sa-cloudbuild.iam.gserviceaccount.com"

gcloud source repos create cloud_build_slsa
gcloud source repos clone cloud_build_slsa

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$CLOUD_BUILD_SERVICE_AGENT" \
  --role="roles/secretmanager.admin"

gcloud builds connections create github conn1 --region=us-central1
```


```bash
go install github.com/slsa-framework/slsa-verifier/v2/cli/slsa-verifier@latest

export IMAGE=us-central1-docker.pkg.dev/srashid-test2/repo1/test@sha256:ad0f88043f0e8a22aa52000e324523f496b5377ea2690f7d3f86a454997efa45

export BUILDER_ID="https://cloudbuild.googleapis.com/GoogleHostedWorker"
export SOURCE="gs://srashid-test2_cloudbuild/source/1728138804.200539-7c3518f252d54daaa666badfb86d85ab.tgz#1728138804875155"

gcloud artifacts docker images describe $IMAGE --format json --show-provenance > provenance.json

slsa-verifier verify-image "$IMAGE" \
--provenance-path provenance.json \
--source-uri $SOURCE \
--builder-id=$BUILDER_ID
```


## References

- [Deterministic container hashes and container signing using Cosign, Kaniko and Google Cloud Build](https://github.com/salrashid123/cosign_kaniko_cloud_build)
- [Deterministic builds with go + bazel + grpc + docker](https://github.com/salrashid123/go-grpc-bazel-docker)
- [Deterministic container hashes and container signing using Cosign, Bazel and Google Cloud Build](https://github.com/salrashid123/cosign_bazel_cloud_build)
- [Deterministic container images with java and GCP APIs using bazel](https://github.com/salrashid123/java-bazel-docker)
- [https://github.com/salrashid123/python-bazel-docker](Deterministic container images with python and GCP APIs using bazel)
- [Deterministic container images with c++ and GCP APIs using bazel](https://github.com/salrashid123/cpp-bazel-docker)
- [Deterministic builds with nodejs + bazel + docker](https://github.com/salrashid123/nodejs-bazel-docker)




