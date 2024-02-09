FROM ros:noetic-perception

# Utilitaires
RUN apt-get update && \
    apt-get install -y \
    git \
    cmake \
    wget \
    tar \
    libx11-dev \
    xorg-dev \
    libssl-dev \
    build-essential \
    libusb-1.0-0-dev \
    libglu1-mesa-dev
    
# OpenGL
RUN apt-get install -qq --no-install-recommends \
    libglvnd0 \
    libgl1 \
    libglx0 \
    libegl1 \
    libxext6 \
    libx11-6 && \
    rm -rf /var/lib/apt/lists/* 

# Realsense SDK
RUN mkdir /dpds && \
    cd /dpds && \
    wget --no-check-certificate https://github.com/IntelRealSense/librealsense/archive/refs/tags/v2.50.0.tar.gz && \
    tar -xzf v2.50.0.tar.gz && \
    rm v2.50.0.tar.gz && \
    cd librealsense-2.50.0/ && \
    mkdir build && cd build && \
    cmake .. && \
    make && \
    make install

ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES graphics,utility,compute
ENV CERES_VERSION="1.12.0"
ENV CATKIN_WS=/root/catkin_ws

RUN if [ "x$(nproc)" = "x1" ] ; then export USE_PROC=1 ; \
    else export USE_PROC=$(($(nproc)/2)) ; fi && \
    # Install ros dependencies
    apt-get update && apt-get install -y \
    libatlas-base-dev \
    libeigen3-dev \
    libgoogle-glog-dev \
    libsuitesparse-dev \
    python3-catkin-tools \
    ros-${ROS_DISTRO}-cv-bridge \
    ros-${ROS_DISTRO}-image-transport \
    ros-${ROS_DISTRO}-message-filters \
    ros-${ROS_DISTRO}-tf && \
    rm -rf /var/lib/apt/lists/*

# Build and install Ceres
RUN git clone https://ceres-solver.googlesource.com/ceres-solver && \
    cd ceres-solver && \
    git checkout tags/${CERES_VERSION} && \
    mkdir build && cd build && \
    cmake .. && \
    make -j$(USE_PROC) install && \
    rm -rf ../../ceres-solver && \
    mkdir -p $CATKIN_WS/src/VINS-Fusion/

# Get and build ros packages (note that we are using the forked version of VINS-Fusion)
WORKDIR $CATKIN_WS
RUN cd src && \ 
    git clone https://github.com/guisoares9/VINS-Fusion.git && \
    apt-get update && apt-get install -y ros-${ROS_DISTRO}-realsense2-camera && \
    cd .. 
ENV TERM xterm
ENV PYTHONIOENCODING UTF-8
RUN catkin config \
      --extend /opt/ros/$ROS_DISTRO \
      --cmake-args \
        -DCMAKE_BUILD_TYPE=Release && \
    catkin build

# Install Zsh
RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common \
    dbus-x11 && \
    apt-get clean

RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -t jispwoso -p git \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-syntax-highlighting && \
    chsh -s /bin/zsh

RUN echo "source /opt/ros/${ROS_DISTRO}/setup.zsh" >> /root/.zshrc && \
    echo "source /root/catkin_ws/devel/setup.zsh" >> /root/.zshrc && \
    echo "export DISPLAY=:0" >> /root/.zshrc 

# Install SSH
RUN apt-get update && apt-get install -y --no-install-recommends \
    openssh-server net-tools && \
    echo "root:;" | chpasswd && \
    echo "AllowTcpForwarding yes\n" >> /etc/ssh/sshd_config && \
    echo "PermitRootLogin yes\n" >> /etc/ssh/sshd_config && \
    echo "Port 2222\n" >> /etc/ssh/sshd_config && \
    echo "#!/bin/zsh\n" > /entrypoint.sh && \
    chmod +x /entrypoint.sh && \
    sed -i '2a #Start Zsh\n/bin/zsh\n' /entrypoint.sh && \
    sed -i '2a #Start SSH\nservice ssh start\n' /entrypoint.sh && \
    mkdir /var/run/sshd && \
    mkdir /root/.ssh && \
    chmod 700 /root/.ssh

# Install other utilities
RUN apt-get install -y --no-install-recommends \
    tmux \
    htop \
    ros-${ROS_DISTRO}-rqt \
    ros-${ROS_DISTRO}-rqt-common-plugins \
    ros-${ROS_DISTRO}-rviz \
    && \
    apt-get clean

ENTRYPOINT [ "/entrypoint.sh" ]
