### Updated base image to golang in order to build with go modules
### Bazel build cannot work with the Tekton library because the current
### KFP Bazel does not support go.mod "replace" on key dependencies.
# Copyright 2021 The Kubeflow Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# 1. Build api server application
FROM golang:1.19.3-buster as builder
RUN apt-get update && apt-get install -y cmake clang musl-dev openssl
WORKDIR /go/src/github.com/kubeflow/pipelines
COPY . .
RUN GO111MODULE=on go build -o /bin/apiserver backend/src/apiserver/*.go
# Check licenses and comply with license terms.
RUN ./hack/install-go-licenses.sh
# First, make sure there's no forbidden license.
RUN go-licenses check ./backend/src/apiserver
RUN go-licenses csv ./backend/src/apiserver > /tmp/licenses.csv && \
    diff /tmp/licenses.csv backend/third_party_licenses/apiserver.csv && \
    go-licenses save ./backend/src/apiserver --save_path /tmp/NOTICES

# Remove sample_config.json file
RUN rm /config/sample_config.json

# 2. Start api web server
FROM debian:stable

ARG COMMIT_SHA=unknown
ENV COMMIT_SHA=${COMMIT_SHA}
ARG TAG_NAME=unknown
ENV TAG_NAME=${TAG_NAME}

WORKDIR /bin

COPY backend/src/apiserver/config/ /config
COPY --from=builder /bin/apiserver /bin/apiserver

# Copy licenses and notices.
COPY --from=builder /tmp/licenses.csv /third_party/licenses.csv
COPY --from=builder /tmp/NOTICES /third_party/NOTICES
RUN chmod +x /bin/apiserver

# Adding CA certificate so API server can download pipeline through URL and wget is used for liveness/readiness probe command
RUN apt-get update && apt-get install -y ca-certificates wget

# Pin sample doc links to the commit that built the backend image
# Commented out due to no commit sha for non-release build
# RUN sed -E "s#/(blob|tree)/master/#/\1/${COMMIT_SHA}/#g" -i /config/sample_config.json && \
#     sed -E "s/%252Fmaster/%252F${COMMIT_SHA}/#g" -i /config/sample_config.json

# Expose apiserver port
EXPOSE 8888

# Start the apiserver
CMD /bin/apiserver --config=/config --sampleconfig=/config/sample_config.json -logtostderr=true
