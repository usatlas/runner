# Replace value with the latest runner release version
# source: https://github.com/actions/runner/releases
ARG RUNNER_VERSION="2.329.0"
# Replace value with the latest runner-container-hooks release version
# source: https://github.com/actions/runner-container-hooks/releases
ARG RUNNER_CONTAINER_HOOKS_VERSION="0.8.0"
ARG RUNNER_ARCH="x64"

FROM gitlab-registry.cern.ch/linuxsupport/alma9-base AS runner
ARG RUNNER_VERSION
ARG RUNNER_CONTAINER_HOOKS_VERSION
ARG RUNNER_ARCH

# Runner environment
ENV RUNNER_MANUALLY_TRAP_SIG=1
ENV ACTIONS_RUNNER_PRINT_LOG_TO_STDOUT=1

# Install dependencies
RUN dnf install -y curl unzip sudo tar gzip \
    && dnf clean all

# Create runner user
RUN groupadd -g 123 docker \
    && useradd -m -u 1001 -s /bin/bash runner \
    && usermod -aG wheel,docker runner \
    && echo "runner ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/runner \
    && chmod 440 /etc/sudoers.d/runner

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
