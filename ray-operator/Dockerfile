# Build arguments
ARG SOURCE_CODE=.
ARG CI_CONTAINER_VERSION="unknown"

# Start of Go versioning workaround for lack of go-toolset for version 1.22
FROM registry-proxy.engineering.redhat.com/rh-osbs/openshift-golang-builder:v1.22.2 AS golang

FROM registry.redhat.io/ubi8/ubi:latest AS builder

ARG GOLANG_VERSION=1.22.2

# Install system dependencies
#RUN dnf upgrade -y && dnf install -y \
  #  gcc \
  #  make \
  #  openssl-devel \
   # && dnf clean all && rm -rf /var/cache/yum

# Install Go
ENV PATH=/usr/local/go/bin:$PATH

COPY --from=golang /usr/lib/golang /usr/local/go
# End of Go versioning workaround


WORKDIR /workspace
USER root
# Copy the Go Modules manifests
COPY ${SOURCE_CODE}/ray-operator/go.mod go.mod
COPY ${SOURCE_CODE}/ray-operator/go.sum go.sum

# Copy the go source
COPY ${SOURCE_CODE}/ray-operator/main.go main.go
COPY ${SOURCE_CODE}/ray-operator/apis/ apis/
COPY ${SOURCE_CODE}/ray-operator/controllers/ controllers/

# Build
USER root

RUN CGO_ENABLED=1 GOOS=linux GOARCH=amd64 go build -tags strictfipsruntime -a -o manager main.go

FROM registry.access.redhat.com/ubi9/ubi-minimal:9.3
WORKDIR /
COPY --from=builder /workspace/manager .
USER 65532:65532

ENTRYPOINT ["/manager"]
LABEL com.redhat.component="odh-kuberay-operator-controller-container" \
      name="managed-open-data-hub/odh-kuberay-operator-controller-container-rhel8" \
      version="${CI_CONTAINER_VERSION}" \
      git.url="${CI_KUBERAY_UPSTREAM_URL}" \
      git.commit="${CI_KUBERAY_UPSTREAM_COMMIT}" \
      summary="odh-kuberay-operator-controller-container" \
      io.openshift.expose-services="" \
      io.k8s.display-name="odh-kuberay-operator-controller-container" \
      maintainer="['managed-open-data-hub@redhat.com']" \
      description="Manages lifecycle of RayClusters, RayJobs, RayServices and associated Kubernetes resources" \
      com.redhat.license_terms="https://www.redhat.com/licenses/Red_Hat_Standard_EULA_20191108.pdf"

