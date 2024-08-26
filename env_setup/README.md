# Docker Setup
First lets start by setting up the docker environment. We will be configuring it with Ubuntu 20.04 OS and install KVM along with it.

1. Create a Dockerfile:
   ```
   nano DockerFile
   ```
2. In the text editor, copy and paste this content:

   ```
   FROM ubuntu:20.04
   
   ENV DEBIAN_FRONTEND=noninteractive
   
   #Install CUDA Toolkit
   RUN apt-get update && apt-get install -y --no-install-recommends \
       gnupg2 curl ca-certificates && \
       curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub | apt-key add - && \
       echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda.list && \
       apt-get update && \
       apt-get install -y --no-install-recommends \
       cuda-toolkit-12-2 && \
       rm -rf /var/lib/apt/lists/*
   
   #Set environment variables
   ENV PATH /usr/local/cuda-12.2/bin:${PATH}
   ENV LD_LIBRARY_PATH /usr/local/cuda-12.2/lib64:${LD_LIBRARY_PATH}
   
   #Install NVIDIA Container Toolkit
   RUN distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && \
       curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add - && \
       curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list && \
       apt-get update && \
       apt-get install -y --no-install-recommends nvidia-container-toolkit && \
       rm -rf /var/lib/apt/lists/*
   
   #Install KVM-related packages
   RUN apt-get update && apt-get install -y --no-install-recommends \
       qemu-kvm \
       libvirt-daemon-system \
       libvirt-clients \
       bridge-utils \
       && rm -rf /var/lib/apt/lists/*
   
   # Install sudo
   RUN apt-get update && apt-get install -y --no-install-recommends sudo && rm -rf /var/lib/apt/lists/*
   
   # Create a new user and add to sudo group
   RUN useradd -m docker-user && echo "docker-user:docker-user" | chpasswd && adduser docker-user sudo
   
   # Allow sudo commands without password
   RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
   
   # Switch to the new user
   USER docker-user
   
   # Set working directory
   WORKDIR /home/docker-user
   
   # Set the default command
   CMD ["/bin/bash"]
   ```
   Ctrl+S to save the file and then press Ctrl+X to close it.
3. Now, in the terminal run this to build the docker image from the docker file:
   ```
   docker build -t my-cuda-container .
   ```

4. Run the Docker container with the necessary permissions:
   ```
   docker run --gpus all \
           --device /dev/kvm \
           --group-add $(getent group kvm | cut -d: -f3) \
           -v /var/run/libvirt/libvirt-sock:/var/run/libvirt/libvirt-sock \
           --network host \
           -it my-cuda-container
   ```

# Environment Installation
## Android Software Development Kit (SDK)
### Install Java (JDK 8)
Download a Java Development Kit 8 (v1.8.0) release version from the open-source Java releaser [OpenLogic](https://www.oracle.com/java/technologies/downloads/). Install using your Linux package installer, like `apt` or `rpm`. For example, on a Debian server:

```bash
sudo apt-get update
cd ~ && mkdir install-android/ && cd install-android
wget https://builds.openlogic.com/downloadJDK/openlogic-openjdk/8u412-b08/openlogic-openjdk-8u412-b08-linux-x64-deb.deb
sudo apt install ./openlogic-openjdk-8u412-b08-linux-x64-deb.deb
```

If you already has a java binary previously, you should also do this:

```bash
sudo update-alternatives --config java # select /usr/lib/jvm/openlogic-openjdk-8-hotspot-amd64/bin/java
```

Check whether the installation is successful by `java -version`. You should expect the output shows version 1.8.0. Higher versions makes `sdkmanager` crash.

```bash
java -version
# openjdk version "1.8.0_412-412"
# OpenJDK Runtime Environment (build 1.8.0_412-412-b08)
# OpenJDK 64-Bit Server VM (build 25.412-b08, mixed mode)
```
