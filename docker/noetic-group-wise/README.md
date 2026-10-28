# Prerequisites

- Ubuntu host with an GPU
- NVIDIA Container Toolkit for NVIDIA GPUs
- **VM** instance on a cloud (container instances such as Runpod Pods cannot run Docker)
- Inbound port 5901 (VNC)
- [RealVNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on your PC

# Must Install

## Docker

```bash
# Repository
sudo apt-get update && sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Install Docker Engine with the compose plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## NVIDIA Toolkit

```bash
# Repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install and register with Docker
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## Port Mapping

some cloud services like [vast.ai](http://vast.ai) maps the inbound port to the random available port of the public IP.

```
# For vast.ai, look up env variables
root@ubuntu:~# env | grep PORT
VAST_TCP_PORT_22=40028
VAST_TCP_PORT_5901=40023
VAST_UDP_PORT_741641=40016
```



# Quick Start

```
# 1. Clone this repository
git clone https://github.com/DAIMLab/ie251-manufacturing-process.git
cd ie251-manufacturing-process/docker/noetic-group-wise

# 2. Launch docker
docker compose up -d

# 3. Connect with RealVNC Viewer (password: daim), and accept the unencrypted-connection warning
<instance IP>::<port>
```
