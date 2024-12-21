FROM osrf/ros:noetic-desktop-focal

# install ros packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-noetic-desktop-full=1.5.0-1* \
    && rm -rf /var/lib/apt/lists/*

# Install necessary libraries for GUI applications
RUN apt-get update && apt-get install -y \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*


ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y apt-utils curl wget git bash-completion build-essential sudo

# Now create the user
ARG UID=1000
ARG GID=1000
RUN addgroup --gid ${GID} kinova-phanthom
RUN adduser --gecos "ROS User" --disabled-password --uid ${UID} --gid ${GID} kinova-phanthom
RUN usermod -a -G dialout kinova-phanthom
RUN mkdir config && echo "kinova-phanthom ALL=(ALL) NOPASSWD: ALL" > config/99_aptget
RUN cp config/99_aptget /etc/sudoers.d/99_aptget
RUN chmod 0440 /etc/sudoers.d/99_aptget && chown root:root /etc/sudoers.d/99_aptget

RUN apt-get update && apt-get install -y apt-utils curl wget git bash-completion build-essential sudo && rm -rf /var/lib/apt/lists/*

# tools
RUN apt-get update && apt-get -y install vim udev usbutils
RUN apt install -y python3-wstool python3-catkin-tools
RUN apt-get -y install python3-pip  libusb-1.0-0-dev && python3 -m pip install pyserial

# Home 
ENV HOME=/home/kinova-phanthom
RUN mkdir -p ${HOME}/catkin_ws/kinova
RUN mkdir -p ${HOME}/catkin_ws/phanthom

# Copy dependeces for phanthom
RUN mkdir -p ${HOME}/catkin_ws/phanthom/dependences/Phanthom
COPY /Phanthom ${HOME}/catkin_ws/phanthom/dependences/Phanthom
COPY ./Phanthom/99-3dsystems.rules /etc/udev/rules.d/
RUN cd ${HOME}/catkin_ws/phanthom/dependences/Phanthom && cp libPhantomIOLib42.so /usr/lib/
RUN cd ${HOME}/catkin_ws/phanthom/dependences/Phanthom/openhaptics_3.4-0-developer-edition-amd64 && chmod +x install &&./install 

# phanthom dependeces
RUN apt install -y libxml2-dev libraw1394-dev libncurses5-dev qtcreator swig sox espeak cmake-curses-gui cmake-qt-gui git subversion gfortran libcppunit-dev libqt5xmlpatterns5-dev
RUN apt-get install -y freeglut3-dev libncurses5-dev zlib1g-dev
RUN apt-get install -y libncurses5 libncursesw5 #libncurses5:i386
RUN apt-get install -y libfltk1.3-dev fluid 

# Kinova Kortex repository
RUN apt-get update && apt-get install -y python3 python3-pip
RUN python3 -m pip install conan==1.59
RUN conan config set general.revisions_enabled=1
RUN conan profile new default --detect > /dev/null
RUN conan profile update settings.compiler.libcxx=libstdc++11 default
RUN mkdir -p ${HOME}/catkin_ws/kinova/src && cd ${HOME}/catkin_ws/kinova/src && git clone -b noetic-devel https://github.com/Kinovarobotics/ros_kortex.git
RUN cd ${HOME}/catkin_ws/kinova && rosdep update && rosdep install --from-paths src --ignore-src -y

# Catkin make Kinova Kortex
RUN /bin/bash -c 'source /opt/ros/noetic/setup.bash &&\
    cd ${HOME}/catkin_ws/kinova &&\
    catkin_make'

# create workspace for phanthom
RUN cd ${HOME}/catkin_ws/phanthom && mkdir src && catkin init && catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release  
# Clone repository for phanthom
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/cisstNetlib.git
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/cisst.git
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/crtk_msgs.git
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/crtk_python_client.git
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/cisst-ros.git
RUN cd ${HOME}/catkin_ws/phanthom/src && git clone https://github.com/EPVelasco/sawSensablePhantom.git

# Catkin make phanthom
WORKDIR ${HOME}/catkin_ws/phanthom
RUN /bin/bash -c "source /opt/ros/noetic/setup.bash && catkin build -j$(nproc)"

# Install ufw Firewall tools
RUN apt-get update && apt-get install ufw -y 

# bash update
RUN echo "TERM=xterm-256color" >> ~/.bashrc
RUN echo "# COLOR Text" >> ~/.bashrc
RUN echo "PS1='\[\033[01;33m\]\u\[\033[01;33m\]@\[\033[01;33m\]\h\[\033[01;34m\]:\[\033[00m\]\[\033[01;34m\]\w\[\033[00m\]\$ '" >> ~/.bashrc
RUN echo "CLICOLOR=1" >> ~/.bashrc
RUN echo "LSCOLORS=GxFxCxDxBxegedabagaced" >> ~/.bashrc
RUN echo "" >> ~/.bashrc
RUN echo "## ROS" >> ~/.bashrc
RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc

# Establecer variables de entorno para NVIDIA
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility

RUN cd /home/kinova-phanthom/catkin_ws/phanthom/dependences/Phanthom && chmod 777 Touch_Setup

COPY ./Phanthom/start_omni_LAN.sh start_omni_LAN.sh
COPY ./Phanthom/start_omni_USB.sh start_omni_USB.sh
COPY ./Phanthom/Touch_Diagnostic Touch_Diagnostic
RUN chmod +x start_omni_USB.sh start_omni_LAN.sh Touch_Diagnostic
