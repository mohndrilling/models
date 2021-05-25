FROM tensorflow/tensorflow:2.4.1-gpu

ARG DEBIAN_FRONTEND=noninteractive

# Install apt dependencies
RUN apt-get update && apt-get install -y \
    git \
    gpg-agent \
    python3-cairocffi \
    protobuf-compiler \
    python3-pil \
    python3-lxml \
    python3-tk \
    wget \
    vim \
    sudo \
    tmux \
    nano \
    libgl1-mesa-glx \
    htop \
    iputils-ping


# Install gcloud and gsutil commands
# https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu
RUN export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)" && \
    echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update -y && apt-get install google-cloud-sdk -y


# Add new user to avoid running as root
ARG USER=tf
ARG PW='tf'
RUN useradd -m ${USER} && echo "${USER}:${PW}" | chpasswd
RUN adduser ${USER} sudo
USER tf

# Copy this version of of the model garden into the image
COPY --chown=tf . /home/tf/models

# Compile protobuf configs
RUN (cd /home/tf/models/research/ && protoc object_detection/protos/*.proto --python_out=.)
WORKDIR /home/tf/models/research/

RUN cp object_detection/packages/tf2/setup.py ./
ENV PATH="/home/tf/.local/bin:${PATH}"

RUN python -m pip install -U pip
RUN python -m pip install .

RUN python -m pip install boto3

ENV TF_CPP_MIN_LOG_LEVEL 3
