# Quick Start

## NVIDIA Toolkit

Needs the NVIDIA driver (`nvidia-smi` works) and Docker on the host. Use a full driver, not the
`nvidia-headless` variant: VirtualGL needs the driver's OpenGL/EGL libraries.

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

# Check: prints the GPU table from inside a container
sudo docker run --rm --gpus all ubuntu nvidia-smi
```
