FROM mcr.microsoft.com/azure-cli:azurelinux3.0

SHELL ["/bin/bash", "-lc"]

ARG HOST_USER=toolbox
ARG HOST_UID=1000
ARG HOST_GID=1000
ARG HOST_HOME=/home/toolbox

RUN tdnf -y update \
 && tdnf -y install \
      ca-certificates \
      curl \
      git \
      jq \
      tar \
      gzip \
      unzip \
      gawk \
      coreutils \
      findutils \
      shadow-utils \
      util-linux \
      sudo \
 && tdnf -y clean all

RUN set -euo pipefail; \
  ARCH="amd64"; \
  KVER="$(curl -fsSL https://dl.k8s.io/release/stable.txt)"; \
  curl -fsSLo /usr/local/bin/kubectl \
    "https://dl.k8s.io/release/${KVER}/bin/linux/${ARCH}/kubectl"; \
  chmod +x /usr/local/bin/kubectl; \
  kubectl version --client=true --output=yaml || true

RUN set -euo pipefail; \
  curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash; \
  helm version

RUN set -euo pipefail; \
  ARCH="amd64"; \
  YQ_VER="v4.44.3"; \
  curl -fsSLo /usr/local/bin/yq \
    "https://github.com/mikefarah/yq/releases/download/${YQ_VER}/yq_linux_${ARCH}"; \
  chmod +x /usr/local/bin/yq; \
  yq --version

RUN set -euo pipefail; \
  ARCH="amd64"; \
  curl -fsSLo /tmp/k9s.tar.gz \
    "https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_${ARCH}.tar.gz"; \
  tar -xzf /tmp/k9s.tar.gz -C /tmp k9s; \
  install -m 0755 /tmp/k9s /usr/local/bin/k9s; \
  rm -rf /tmp/k9s /tmp/k9s.tar.gz; \
  k9s version || true

RUN set -euo pipefail; \
  if [ "${HOST_UID}" = "0" ]; then \
    echo "Using root user"; \
  else \
    groupadd -g "${HOST_GID}" "${HOST_USER}" || true; \
    useradd -m -u "${HOST_UID}" -g "${HOST_GID}" -d "${HOST_HOME}" -s /bin/bash "${HOST_USER}" || true; \
    echo "${HOST_USER} ALL=(ALL) NOPASSWD:ALL" > "/etc/sudoers.d/${HOST_USER}"; \
    chmod 0440 "/etc/sudoers.d/${HOST_USER}"; \
  fi

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

WORKDIR ${HOST_HOME}
