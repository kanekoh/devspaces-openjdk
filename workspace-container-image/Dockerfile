FROM registry.access.redhat.com/ubi10/ubi:10.0

ARG USER_HOME_DIR="/home/user"
ARG WORK_DIR="/projects"

ARG INSTALL_PACKAGES="procps-ng openssl git tar gzip zip xz unzip which shadow-utils bash zsh vi wget curl jq podman buildah skopeo python3-pip python3-devel fuse-overlayfs util-linux vim-minimal vim-enhanced java-21-openjdk.x86_64 maven"

ENV HOME=${USER_HOME_DIR}
ENV BUILDAH_ISOLATION=chroot

COPY --chown=0:0 entrypoint.sh /

# 必要なツールをインストール
RUN dnf install -y curl tar wget && \
    dnf clean all

RUN wget https://github.com/konveyor/editor-extensions/releases/download/v0.1.0/konveyor-v0.1.0.vsix && \
    mkdir -p ~/.vscode-plugins/ && \
    mv konveyor-v0.1.0.vsix ~/.vscode-plugins/

# ocクライアントをダウンロードして /usr/local/bin に展開
RUN curl -L -o /tmp/oc.tar.gz https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable-4.18/openshift-client-linux-4.18.19.tar.gz && \
    tar -xzvf /tmp/oc.tar.gz -C /usr/local/bin && \
    rm -f /tmp/oc.tar.gz

# (オプション) パーミッション確認
RUN chmod +x /usr/local/bin/oc /usr/local/bin/kubectl

RUN dnf --disableplugin=subscription-manager install -y ${INSTALL_PACKAGES}; \
    dnf update -y ; \
    dnf clean all ; \
    mkdir -p /usr/local/bin ; \
    mkdir -p ${WORK_DIR} ; \
    chgrp -R 0 /home ; \
    chmod -R g=u /home ${WORK_DIR} ; \
    chmod +x /entrypoint.sh ; \
    chown 0:0 /etc/passwd ; \
    chown 0:0 /etc/group ; \
    chmod g=u /etc/passwd /etc/group ; \
    # Setup for rootless podman
    setcap cap_setuid+ep /usr/bin/newuidmap ; \
    setcap cap_setgid+ep /usr/bin/newgidmap ; \
    touch /etc/subgid /etc/subuid ; \
    chown 0:0 /etc/subgid ; \
    chown 0:0 /etc/subuid ; \
    chmod -R g=u /etc/subuid /etc/subgid ; 

USER 1000
WORKDIR ${WORK_DIR}
ENTRYPOINT ["/usr/libexec/podman/catatonit","--","/entrypoint.sh"]
CMD [ "tail", "-f", "/dev/null" ]
