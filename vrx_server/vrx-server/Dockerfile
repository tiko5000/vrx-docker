ARG BASEIMG=ubuntu:focal
FROM $BASEIMG

# Set ROS distribution
ARG DIST=noetic

# Set Gazebo verison
ARG GAZ=gazebo11

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt update \
 && apt install -y \
    tzdata \
 && ln -fs /usr/share/zoneinfo/America/Los_Angeles /etc/localtime \
 && dpkg-reconfigure --frontend noninteractive tzdata \
 && apt clean

# Tools useful during development.
 RUN apt update \
  && apt install -y \
         build-essential \
         cmake \
         cppcheck \
         curl \
         git \
         language-pack-en \
         libeigen3-dev \
         libgles2-mesa-dev \
         lsb-release \
         gdb \
         mercurial \
         pkg-config \
         psmisc \
         python3-dbg \
         python3-pip \
         python3-venv \
         qtbase5-dev \
         ruby \
         software-properties-common \
         sudo \
         vim \
         wget \
  && apt clean

# Get ROS and Gazebo
RUN wget -P /tmp/ http://packages.osrfoundation.org/gazebo.key
RUN wget -P /tmp/ http://packages.ros.org/ros.key
RUN /bin/sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' \
 && apt-key add /tmp/ros.key \
 && /bin/sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list' \
 && apt-key add /tmp/gazebo.key \
 && apt update \
 && apt install -y \
    ${GAZ} \
    lib${GAZ}-dev \
    ros-${DIST}-compressed-image-transport \
    ros-${DIST}-ros-base \
    ros-${DIST}-gazebo-plugins \
    ros-${DIST}-gazebo-ros \
    ros-${DIST}-hector-gazebo-plugins \
    ros-${DIST}-joy \
    ros-${DIST}-joy-teleop \
    ros-${DIST}-key-teleop \
    ros-${DIST}-robot-localization \
    ros-${DIST}-robot-state-publisher \
    ros-${DIST}-rviz \
    ros-${DIST}-teleop-tools \
    ros-${DIST}-teleop-twist-keyboard \
    ros-${DIST}-velodyne-simulator \
    ros-${DIST}-xacro \
 && apt clean

# Removed
#    python-rosdep \
# && rosdep init \
# RUN rosdep update

# Set USER and GROUP
ARG USER=developer
ARG GROUP=developer
ARG USERID=1000
ARG GROUPID=1000

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`.

RUN curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.4/fixuid-0.4-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - && \
    chown root:root /usr/local/bin/fixuid && \
    chmod 4755 /usr/local/bin/fixuid && \
    mkdir -p /etc/fixuid && \
    printf "user: $USER\ngroup: $GROUP\n" > /etc/fixuid/config.yml

RUN addgroup --gid $GROUPID $USER && \
    useradd --no-log-init --create-home --uid $USERID --gid $GROUPID --home-dir /home/$USER --shell /bin/sh -c "" $USER

RUN adduser $USER sudo \
 && echo "$USER ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USER

# Commands below run as the developer user.
USER $USER:$GROUP

# When running a container start in the developer's home folder.
WORKDIR /home/$USER

# Create workspace
RUN mkdir -p vrx_ws/src

# Copy the VRX repository from the local file system
# We can't use the USER:GROUP variables until Docker adds support to --chown
# COPY --chown=developer:developer . vrx_ws/src/vrx/
RUN git clone -b 1.6.2 https://github.com/osrf/vrx.git 
RUN mv ./vrx ./vrx_ws/src

# Compile the VRX project.
RUN /bin/bash -c ". /opt/ros/${DIST}/setup.bash && cd vrx_ws && catkin_make"

# Source all the needed environment files.
RUN /bin/sh -c 'echo ". /opt/ros/${DIST}/setup.bash" >> ~/.bashrc' \
 && /bin/sh -c 'echo ". /usr/share/gazebo/setup.sh" >> ~/.bashrc' \
 && /bin/sh -c 'echo ". ~/vrx_ws/devel/setup.sh" >> ~/.bashrc'
## END OF SECTION BASED ON vrx/docker/Dockerfile

# Expose port used to communiate with gzserver
EXPOSE 11345

# Stuff for nvidia-docker
LABEL com.nvidia.volumes.needed="nvidia_driver"
ENV PATH /usr/local/nvidia/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

# Set encoding to use unicode characters
# RUN sudo locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8

# Takes too long to do, probably not needed
# Get gazebo models early since it is big
# RUN wget -P /tmp/ https://bitbucket.org/osrf/gazebo_models/get/default.tar.gz \
#  && mkdir -p $HOME/.gazebo/models \
#  && tar -xvf /tmp/default.tar.gz -C $HOME/.gazebo/models --strip 1 \
#  && rm /tmp/default.tar.gz

# setup entrypoint
COPY ./vrx_entrypoint.sh /
COPY ./run_vrx_trial.sh /
COPY ./play_vrx_log.sh /
COPY ./gz_utils.sh /

ENTRYPOINT ["/vrx_entrypoint.sh"]
