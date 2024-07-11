# Docker Setup
First lets start by setting up the docker environment. We will be configuring it with Ubuntu 20.04 OS and install KVM along with it.

1. Create a Dockerfile:
   `nano DockerFile`
2. In the text editor, copy and paste this content:

   ```
   #Use Ubuntu 20.04 as a base image
   FROM ubuntu:20.04

   # Install necessary packages for KVM
   RUN apt-get update && apt-get install -y \
       qemu-kvm \
       libvirt-daemon-system \
       libvirt-clients \
       bridge-utils \
       && apt-get clean

   # Set the working directory
   WORKDIR /root
   
   # Keep the user as root
   USER root
   ```
   Ctrl+S to save the file and then press Ctrl+X to close it.
4. Now, in the terminal run this:
   `docker build -t ubuntu-kvm-docker .`
