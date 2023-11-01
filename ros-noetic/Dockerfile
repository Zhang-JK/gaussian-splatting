ARG BASE_IMAGE_ROS=ros:noetic
FROM $BASE_IMAGE_ROS AS base

# dependiences
RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common \
    dbus-x11 \
    wget \
    && \
    apt-get clean

# setup zsh
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -t jispwoso -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-syntax-highlighting && \
    chsh -s /bin/zsh

RUN /bin/bash
WORKDIR /root/ws
ENTRYPOINT [ "/bin/zsh" ]

