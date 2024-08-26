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
