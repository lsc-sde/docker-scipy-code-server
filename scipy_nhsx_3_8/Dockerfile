# az acr build -r crlander -f Dockerfile -t vvcb/scipy-code-server:py-38-2023-02-23 .
# https://discourse.jupyter.org/t/how-to-configure-jupyterhub-to-run-code-server/11578/9
# https://github.com/coder/code-server
# https://github.com/betatim/vscode-binder


ARG OWNER=vvcb
ARG BASE_CONTAINER=jupyter/scipy-notebook:python-3.8
FROM $BASE_CONTAINER

LABEL maintainer="vvcb"
LABEL image="scipy-code-server-py38"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Install essentials for code-server and XFCE
RUN apt-get update --yes && \
    apt-get install --yes --quiet --no-install-recommends \
	build-essential \
    clang \
    cmake \
    curl \
    dbus-x11 \
    g++ \
    htop \
    iputils-ping \
    libopencv-dev \
    make \
    xfce4 \
    xfce4-panel \
    xfce4-session \
    xfce4-settings \
    xorg \
    xubuntu-icon-theme \
    && apt-get remove --yes --quiet light-locker \
    && apt-get autoremove --yes --quiet \
    && apt-get clean --quiet \
    && rm -rf /var/lib/apt/lists/*
    

# Install Code Server and extensions
ENV CODE_VERSION=4.9.1
RUN curl -fOL https://github.com/coder/code-server/releases/download/v$CODE_VERSION/code-server_${CODE_VERSION}_amd64.deb \
    && dpkg -i code-server_${CODE_VERSION}_amd64.deb \
    && rm -f code-server_${CODE_VERSION}_amd64.deb \
    && code-server --install-extension ms-python.python \
    && code-server --install-extension njpwerner.autodocstring \
    && code-server --install-extension davidanson.vscode-markdownlint


# Copy custom config for jupyter
COPY jupyter_notebook_config.json /etc/jupyter/jupyter_notebook_config.json

# Finally install packages from environment.yaml
COPY environment.yaml environment.yaml

# Install Python packages for proxying XFCE and code-server
RUN mamba install -c conda-forge jupyter-server-proxy jupyter-vscode-proxy websockify \
    && mamba env update --name base --file environment.yaml \
    && rm environment.yaml \
    && mamba clean --all -f -y \
    && fix-permissions "${CONDA_DIR}" \
    && fix-permissions "/home/${NB_USER}"

# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}