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
### Install SDK Manager

Download the Android SDK for Linux from the [official website](https://developer.android.com/studio/index.html#downloads). For your convenience, you can also directly download the [installation package](https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip).

```bash
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
```

Now specify the android installation path and unzip the installation package to that path. It's recommended to use `/home/<username>/.android` as the default installation path.

```bash
export ANDROID_HOME="/home/docker-user/.android" # recommended: /home/<username>/.android
mkdir -p $ANDROID_HOME
unzip sdk-tools-linux-4333796.zip -d $ANDROID_HOME
```

Make sure you have `unzip` installed. For example, use `sudo apt install unzip -y` to install on Debian servers. To check whether the unzip is successful:

```bash
ls $ANDROID_HOME
# tools
```

### SDK Emulator

Prior to install the SDK emulators, set the environment variables:

```bash
echo "export ANDROID_HOME=$ANDROID_HOME" >> ~/.bashrc
echo 'export SDK=$ANDROID_HOME' >> ~/.bashrc
echo 'export ANDROID_SDK_ROOT=$ANDROID_HOME' >> ~/.bashrc
echo 'export PATH=$SDK/emulator:$SDK/tools:$SDK/tools/bin:$SDK/platform-tools:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Now you should be able to locate the `sdkmanager` binary:

```bash
which sdkmanager
# .../tools/bin/sdkmanager
```

Then install the Android emulator 28 (other versions should also work, but the offline data we provided is in version 28):

```bash
yes | sdkmanager "platform-tools" "platforms;android-28" "emulator"
yes | sdkmanager "system-images;android-28;google_apis;x86_64"
yes | sdkmanager "build-tools;28.0.0"
```

Now you should be able to view the version of the emulator:

```bash
emulator -version
# INFO    | Storing crashdata in: /tmp/android-<username>/emu-crash-34.2.14.db, detection is enabled for process: 16670
# INFO    | Android emulator version 34.2.14.0 (build_id 11834374) (CL:N/A)
# INFO    | Storing crashdata in: /tmp/android-<username>/emu-crash-34.2.14.db, detection is enabled for process: 16670
# INFO    | Duplicate loglines will be removed, if you wish to see each individual line launch with the -log-nofilter flag.
# ...
```
### Install Conda and gdown
```
wget https://repo.anaconda.com/archive/Anaconda3-2023.03-Linux-x86_64.sh -O anaconda.sh
sudo bash anaconda.sh -b -p /opt/anaconda
rm anaconda.sh
export PATH="/opt/anaconda/bin:$PATH"
conda init
source ~/.bashrc
```

Now, we create a new conda environment named 'digirl' configured for python 3.10:
```
conda create --name digirl python=3.10
conda activate digirl
```
Now, in this environment we install gdown which allows us to download files from google drive:
```
pip3 install gdown
```

