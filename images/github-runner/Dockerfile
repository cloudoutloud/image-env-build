# Build from base runner helm image has the agent pre installed
FROM --platform=linux/amd64 summerwind/actions-runner:ubuntu-22.04

USER root

ENV HELM_VERSION v3.15.3
ENV YQ_VERSION v4.44.2

ARG DEBIAN_FRONTEND=noninteractive

# install basic dependencies
RUN apt update && apt install -y \
    vim xxd gnupg make rsync && \
    apt clean && rm -rf /var/lib/apt/lists/*

# install yq
RUN curl -L https://github.com/mikefarah/yq/releases/download/$YQ_VERSION/yq_linux_amd64 -o /usr/local/bin/yq && \
    chmod a+x /usr/local/bin/yq

# install latest stable kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
RUN install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# install helm
RUN curl -LO "https://get.helm.sh/helm-$HELM_VERSION-linux-amd64.tar.gz"
RUN tar -zxvf helm-$HELM_VERSION-linux-amd64.tar.gz
RUN mv linux-amd64/helm /usr/local/bin/helm

# switch to runner user, because runner agent service should not run by Sudo
USER runner
