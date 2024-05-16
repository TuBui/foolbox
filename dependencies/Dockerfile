FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04
ENV DEBIAN_FRONTEND noninteractive
# rm nvidia repo if necessary
RUN rm -f /etc/apt/sources.list.d/cuda.list /etc/apt/sources.list.d/nvidia-ml.list

# base packages
RUN apt-get update -y && apt-get install -y \
    ca-certificates \
    software-properties-common \
    sudo \
    git \
    wget \
    curl \
    bzip2 \
    # next 2 lines are packages required for python
    build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev \
    libgtk2.0-dev \
    libopenblas-dev \
    g++ \
    cmake \
    sshfs \
    vim nano \
    && apt-get -y purge manpages-dev \
    && apt-get clean && rm -rf /var/lib/apt/lists/*


# imagemagik and opencv dependencies
RUN apt-get update -y && apt-get install -y \
    libmagickwand-dev \
    libgl1-mesa-glx \
    && rm -rf /var/lib/apt/lists/*

# custom user
ARG NB_USER="appuser"
ARG NB_UID="1000"
ARG NB_GID="1000"


SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    HOME="/app/${NB_USER}"
RUN useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    mkdir -p "${HOME}" && \
    chown "${NB_USER}:${NB_GID}" "${HOME}"
RUN adduser ${NB_USER} sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER ${NB_UID}

# conda
WORKDIR "${HOME}"
ARG PYTHON_VER="3.9.18"

ENV PYENV_ROOT=${HOME}/.pyenv
ENV PATH=${HOME}/.pyenv/shims:${HOME}/.pyenv/bin:${PATH}
RUN git clone  -b 'v2.3.28' --single-branch --depth 1 https://github.com/pyenv/pyenv.git ~/.pyenv && \
  cd ~/.pyenv && src/configure && make -C src && cd .. \
  && env PYTHON_CONFIGURE_OPTS='--enable-optimizations --with-lto' PYTHON_CFLAGS='-march=native -mtune=native' pyenv install ${PYTHON_VER} && pyenv global ${PYTHON_VER}

RUN pip install torch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 --index-url https://download.pytorch.org/whl/cu118

# imagenet c
RUN git clone https://github.com/TuBui/robustness.git && cd robustness/ImageNet-C/imagenet_c/ && pip install -e . 

# pip packages
ADD ./requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt


