ARG BASE_IMAGE_WEBOTS=cyberbotics/webots:R2023b-ubuntu22.04
FROM $BASE_IMAGE_WEBOTS AS base

# dependiences
RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common \
    dbus-x11 \
    wget \
    python3.10 \
    python3-pip \
    && \
    apt-get clean

# python env
RUN pip install matplotlib pillow
RUN pip install gymnasium

# setup zsh
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -t jispwoso -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-syntax-highlighting && \
    chsh -s /bin/zsh

RUN /bin/bash
WORKDIR /root/ws
ENTRYPOINT [ "/bin/zsh" ]

