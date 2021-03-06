FROM create_ros_melodic
LABEL maintainer="Emiliano Borghi"

ENV USER="create"

USER root

# Setup sources.list for Gazebo
RUN echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -sc) main" > /etc/apt/sources.list.d/gazebo-stable.list

# Setup keys for Gazebo
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D2486D2DD83DB69272AFE98867170598AF249743

# Install Gazebo 9
RUN apt-get update && \
    apt-get install -y \
    gazebo9 \
    libgazebo9-dev

# Install dependencies
# https://unix.stackexchange.com/a/391112
COPY packages.txt packages.txt
RUN apt-get update && \
    xargs -a packages.txt apt-get install -y

USER $USER

# Install custom cartographer
ENV CARTO_WS="/home/${USER}/catkin_ws"
RUN mkdir ${CARTO_WS}
WORKDIR ${CARTO_WS}
RUN wstool init src && \
    wstool merge -t src \
        https://raw.githubusercontent.com/ekumenlabs/cartographer_ros/master/cartographer_ros.rosinstall && \
    wstool update -t src
RUN src/cartographer/scripts/install_proto3.sh --local
RUN src/cartographer/scripts/install_abseil.sh
RUN rosdep install --from-paths src --ignore-src --rosdistro=${ROS1_DISTRO} -y -r --os=ubuntu:bionic
# Change again to root in order to install in /opt
USER root
RUN /bin/bash -c "source /opt/ros/${ROS1_DISTRO}/setup.bash && \
    catkin_make_isolated --install --use-ninja --install-space /opt/ros/${ROS1_DISTRO}"
RUN rm -rf ${CARTO_WS}

USER $USER
