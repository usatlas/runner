# https://docs.docker.com/reference/dockerfile#automatic-platform-args-in-the-global-scope
# TARGETOS:
# TARGETARCH: amd64, arm64
ARG TARGETOS
ARG TARGETARCH

# Replace value with the latest runner release version
# source: https://github.com/actions/runner/releases
ARG RUNNER_VERSION="2.329.0"
# Replace value with the latest runner-container-hooks release version
# source: https://github.com/actions/runner-container-hooks/releases
ARG RUNNER_CONTAINER_HOOKS_VERSION="0.8.0"

# Replace value with latest docker version from https://github.com/moby/moby/releases
ARG DOCKER_VERSION="29.0.1"
# Replace value with latest buildx version from https://github.com/docker/buildx/releases
ARG BUILDX_VERSION="0.30.0"

FROM gitlab-registry.cern.ch/linuxsupport/alma9-base AS runner
ARG TARGETOS
ARG TARGETARCH
ARG RUNNER_VERSION
ARG RUNNER_CONTAINER_HOOKS_VERSION
ARG DOCKER_VERSION
ARG BUILDX_VERSION

# Runner environment
ENV RUNNER_MANUALLY_TRAP_SIG=1
ENV ACTIONS_RUNNER_PRINT_LOG_TO_STDOUT=1

# Install dependencies
# For HEP_OSLibs: https://gitlab.cern.ch/linuxsupport/rpms/HEP_OSlibs
RUN <<EOF
set -euxo pipefail
dnf install -y curl unzip sudo tar gzip
case "$TARGETARCH" in
  amd64) arch_path="x86_64"; ;;
  arm64) arch_path="aarch64"; ;;
  *) echo "Unsupported TARGETARCH: $TARGETARCH"; exit 1; ;;
esac
dnf install -y https://linuxsoft.cern.ch/wlcg/el9/${arch_path}/wlcg-repo-1.0.0-1.el9.noarch.rpm
dnf install -y HEP_OSlibs
dnf clean all
EOF

# Create runner user
RUN <<EOF
set -euxo pipefail
groupadd -g 123 docker
useradd -m -u 1001 -s /bin/bash runner
usermod -aG wheel,docker runner
echo "runner ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/runner
chmod 440 /etc/sudoers.d/runner
EOF

# Install docker
RUN <<EOF
set -euxo pipefail
case "$TARGETARCH" in
    amd64)  DOCKER_ARCH="x86_64" ;;
    arm64)  DOCKER_ARCH="aarch64" ;;
    *) echo "Unsupported TARGETARCH: $TARGETARCH"; exit 1 ;;
esac
echo "Using Docker arch: $DOCKER_ARCH"

# Download Docker static binaries
curl -fLo docker.tgz "https://download.docker.com/${TARGETOS}/static/stable/${DOCKER_ARCH}/docker-${DOCKER_VERSION}.tgz"
tar zxvf docker.tgz
rm docker.tgz

# Install docker-buildx plugin
mkdir -p /usr/local/lib/docker/cli-plugins
curl -fLo /usr/local/lib/docker/cli-plugins/docker-buildx "https://github.com/docker/buildx/releases/download/v${BUILDX_VERSION}/buildx-v${BUILDX_VERSION}.linux-${TARGETARCH}"
chmod +x /usr/local/lib/docker/cli-plugins/docker-buildx
EOF

WORKDIR /home/runner

# Download & extract GitHub Actions runner
RUN curl -f -L -o runner.tar.gz \
      https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-${RUNNER_ARCH}-${RUNNER_VERSION}.tar.gz \
    && tar xzf runner.tar.gz \
    && rm runner.tar.gz

# Download & extract runner container hooks
RUN curl -f -L -o runner-container-hooks.zip \
      https://github.com/actions/runner-container-hooks/releases/download/v${RUNNER_CONTAINER_HOOKS_VERSION}/actions-runner-hooks-k8s-${RUNNER_CONTAINER_HOOKS_VERSION}.zip \
    && unzip runner-container-hooks.zip -d ./k8s \
    && rm runner-container-hooks.zip

USER runner
