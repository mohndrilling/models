# FROM tensorflow/tensorflow:2.4.1-gpu
# FROM tensorflow/tensorflow:2.6.0-gpu
FROM tensorflow/tensorflow:2.8.0-gpu


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
    iputils-ping \
    google-perftools


# ### enable user
# ARG USERNAME=dev
# ARG USER_UID=1000
# ARG USER_GID=$USER_UID
# RUN groupadd --gid $USER_GID $USERNAME
# RUN useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
#     # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
#     && apt-get update \
#     && apt-get install -y sudo \
#     && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
#     && chmod 0440 /etc/sudoers.d/$USERNAME
# USER dev

RUN adduser --disabled-password --gecos '' dev
# RUN adduser dev sudo
# RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
RUN usermod -aG sudo dev


# RUN sudo apt-get update
# USER dev
# RUN sudo apt-get update
# Install gcloud and gsutil commands
# https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu
# RUN export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)" && \
#     echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
#     curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
#     apt-get update -y && apt-get install google-cloud-sdk -y


# Add new user to avoid running as root
# ARG USER=tf
# ARG PW='tf'
# RUN useradd -m ${USER} && echo "${USER}:${PW}" | chpasswd
# USER root
# RUN sudo adduser ${USER} sudo
# USER tf

# Copy this version of of the model garden into the image
COPY --chown=dev . /home/dev/models

# Compile protobuf configs
RUN (cd /home/dev/models/research/ && protoc object_detection/protos/*.proto --python_out=.)
WORKDIR /home/dev/models/research/

RUN cp object_detection/packages/tf2/setup.py ./
ENV PATH="/home/dev/.local/bin:${PATH}"

RUN python -m pip install -U pip
RUN python -m pip install .

RUN python -m pip install boto3 labelbox tflite_model_maker tensorflow-datasets
RUN python -m pip install -U numpy

# Install edge tpu compiler
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
RUN apt-get update
RUN apt-get install edgetpu-compiler


USER dev
WORKDIR /labelbox
ENV TF_CPP_MIN_LOG_LEVEL 3
