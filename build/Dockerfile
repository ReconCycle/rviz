# use ubuntu 20.04 because we want to use ROS noetic
ARG TARGET="gpu"
FROM nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04 as gpu
# FROM ubuntu:20.04 as cpu
ARG UID=1000
ARG GID=1000

# FROM ros:noetic
LABEL maintainer "Sebastian Ruiz <sebastian.ruiz@uni-goettingen.de>"

SHELL ["/bin/bash","-c"]

################################################################
## BEGIN: ros:core
################################################################

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends tzdata && \
    rm -rf /var/lib/apt/lists/*

# install packages
RUN apt update && apt install -q -y --no-install-recommends \
    dirmngr \
    gnupg2 \
    && rm -rf /var/lib/apt/lists/*

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# setup sources.list
RUN echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros1-latest.list

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO noetic

################################################################
## END: ros:core
## BEGIN: python3
################################################################

RUN apt update && apt install --no-install-recommends -y \
    software-properties-common \
    build-essential
RUN add-apt-repository universe

RUN apt update --fix-missing && apt install --no-install-recommends -y \
    wget \
    bzip2 \
    ca-certificates \
    zlib1g-dev \
    libncurses5-dev \
    libgdbm-dev \
    libglib2.0-0 \
    libxext6 \
    libsm6 \
    libxrender1 \
    libffi-dev \
    git \
    libusb-1.0-0-dev \
    qt5-default \
    qtbase5-dev \
    qtchooser \
    qt5-qmake \
    qtbase5-dev-tools \
    # libcurl4-openssl-dev \
    # curl \
    vim \
    # python3.9 \
    && rm -rf /var/lib/apt/lists/*

################################################################
## END: python3
## BEGIN: ros:noetic
################################################################

# install ros packages
RUN apt update && apt install --no-install-recommends -y \
    ros-noetic-ros-core=1.5.0-1* \
    ros-noetic-ros-base=1.5.0-1* \
    ros-noetic-desktop

RUN apt update && apt install --no-install-recommends -y \
    python3-rosdep \
    python3-rosinstall \
    python3-vcstools \
    python3-catkin-tools \
    ros-noetic-cv-bridge \
    ros-noetic-tf \
    ros-noetic-wfov-camera-msgs \
    ros-noetic-image-exposure-msgs \
    ros-noetic-camera-info-manager \
    ros-noetic-image-transport \
    ros-noetic-image-transport-plugins \
    ros-noetic-roslint \
    ros-noetic-image-proc \
    ros-noetic-robot-state-publisher \
    ros-noetic-diagnostic-updater \
    ros-noetic-xacro

RUN apt update && apt install --no-install-recommends -y \
    python3 \
    python3-pip


################################################################
## END: ros:noetic
## BEGIN: user setup
################################################################

ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_FRONTEND teletype

# https://stackoverflow.com/questions/27701930/how-to-add-users-to-docker-container
RUN groupadd -g $GID docker
RUN useradd -rm -d /home/docker -s /bin/bash -g docker -G sudo -u $UID docker -p "$(openssl passwd -1 docker)"

WORKDIR /home/docker

################################################################
## END: user setup
## BEGIN: ROS
################################################################

# bootstrap rosdep
RUN rosdep init && \
  rosdep update --rosdistro $ROS_DISTRO

# Create local catkin workspace
ENV CATKIN_WS=/home/docker/catkin_ws
ENV ROS_PYTHON_VERSION=3
WORKDIR $CATKIN_WS

# Initialize local catkin workspace
RUN source /opt/ros/${ROS_DISTRO}/setup.bash \
    && cd $CATKIN_WS \
    && rosdep install -y --from-paths . --ignore-src --rosdistro ${ROS_DISTRO}

RUN pip3 install rospkg
RUN pip3 install rospy-message-converter

RUN apt update && apt install --no-install-recommends -y \
    bash-completion \
    ros-noetic-image-pipeline \
    python3-tk

# apt cache fix
RUN rm /etc/apt/apt.conf.d/docker-clean &&\
    apt-get update

RUN touch /home/docker/.sudo_as_admin_successful

################################################################
## END: ROS
## BEGIN: Workspace
################################################################

# enable case-insensitive tab completion
RUN echo 'set completion-ignore-case On' >> /etc/inputrc

WORKDIR $CATKIN_WS

COPY .bashrc /home/docker/.bashrc

# entrypoint stuff
COPY ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

USER docker
