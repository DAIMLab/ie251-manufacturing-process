# Ubuntu 20.04 + ROS Noetic remote desktop, with every lab-0 dependency installed.
# Students connect with TurboVNC Viewer or a browser (noVNC) and build the lab-0
# workspace (catkin_ws) themselves.
#
# Published without a VNC password (docker-compose.yml adds it on each VM):
#   docker build -t daimlab/noetic-desktop . && docker push daimlab/noetic-desktop
FROM osrf/ros:noetic-desktop-full
ARG DEBIAN_FRONTEND=noninteractive
ARG TURBOVNC_VERSION=3.3.1
ARG VIRTUALGL_VERSION=3.1.5
ARG NOVNC_VERSION=1.7.0

# Remote desktop: XFCE, TurboVNC (VNC server + noVNC web server), VirtualGL (GPU OpenGL)
RUN apt-get update && apt-get install -y --no-install-recommends \
        sudo git curl ca-certificates \
        xfce4 xfce4-terminal dbus-x11 \
 && curl -fsSL -o /tmp/turbovnc.deb \
        https://github.com/TurboVNC/turbovnc/releases/download/${TURBOVNC_VERSION}/turbovnc_${TURBOVNC_VERSION}_amd64.deb \
 && curl -fsSL -o /tmp/virtualgl.deb \
        https://github.com/VirtualGL/virtualgl/releases/download/${VIRTUALGL_VERSION}/virtualgl_${VIRTUALGL_VERSION}_amd64.deb \
 && apt-get install -y --no-install-recommends /tmp/turbovnc.deb /tmp/virtualgl.deb \
 && rm -rf /tmp/*.deb /var/lib/apt/lists/*

# lab-0 apt packages: GUI apps (WSL step 5), build tools (ROS step 6),
# Doosan robot dependencies (workspace step 3)
RUN apt-get update && apt-get install -y \
        gedit nautilus x11-apps \
        python3-rosdep python3-rosinstall python3-rosinstall-generator \
        python3-wstool build-essential \
        'ros-noetic-rqt*' 'ros-noetic-moveit*' ros-noetic-gazebo-ros-control \
        ros-noetic-joint-state-controller ros-noetic-effort-controllers \
        ros-noetic-position-controllers ros-noetic-ros-controllers ros-noetic-ros-control \
        ros-noetic-joint-state-publisher-gui ros-noetic-joint-state-publisher \
 && rm -rf /var/lib/apt/lists/*

# noVNC browser client, served by TurboVNC's web server behind nginx.
# defaults.json is noVNC's own settings file.
ADD https://github.com/novnc/noVNC.git#v${NOVNC_VERSION} /opt/noVNC
COPY novnc-defaults.json /opt/noVNC/defaults.json

RUN groupadd -g 1000 daim \
 && useradd -m -u 1000 -g 1000 -s /bin/bash daim \
 && echo "daim:daim" | chpasswd \
 && echo "daim ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/daim \
 && chmod 0440 /etc/sudoers.d/daim \
 && install -d -o daim -g daim /opt/vnc

# NVIDIA runtime mounts the EGL libraries only with the "graphics" capability.
# Without DISABLE_ROS1_EOL_WARNINGS, RViz opens an end-of-life dialog on every start.
ENV NVIDIA_DRIVER_CAPABILITIES=all \
    DISABLE_ROS1_EOL_WARNINGS=1 \
    VGL_DISPLAY=egl \
    PATH=/opt/TurboVNC/bin:/opt/VirtualGL/bin:${PATH}

USER daim
WORKDIR /home/daim

# lab-0 ROS steps 5 and 7. The base image already ran `rosdep init`.
# Noetic is end-of-life, so rosdep skips it unless told otherwise.
RUN rosdep update --include-eol-distros \
 && echo 'source /opt/ros/noetic/setup.bash' >> ~/.bashrc

# The marker skips Ubuntu's sudo tip in new terminals. The tip calls `groups`,
# which warns about the GPU device's host group that has no name in the container.
RUN touch ~/.sudo_as_admin_successful

# Display :1 serves VNC on 5901 (TurboVNC Viewer over TCP, noVNC over WebSocket)
# and the noVNC page on 5801 (reached through nginx).
# -vgl runs the desktop under vglrun, so every OpenGL app renders on the GPU.
# TurboVNC Viewer follows the server's order and takes TLSVnc (encrypted);
# noVNC cannot do TLS and falls back to VNC.
EXPOSE 5801 5901
CMD ["vncserver", ":1", "-fg", "-wm", "xfce", "-vgl", "-novnc", "/opt/noVNC", \
     "-securitytypes", "TLSVnc,VNC", "-rfbauth", "/opt/vnc/passwd"]
