# Installation

----------------------------------------------------------------------

## RealTime: Embedded

This guide walks through setting up a real-time (RT) embedded environment on a fresh Ubuntu system using Xenomai and an EtherCAT stack.

### 1. Install Xenomai Kernel

Xenomai provides real-time capabilities by adding a co-kernel to Linux.

Steps:
```bash
git clone https://github.com/Advanced-Robotics-Facility/xenomai-ethercat-setup.git
./xenomai-ethercat-setup/xenomai/prepare.sh
```

Configuration options:

* Answer **YES** to "Enable Xenomai Real-Time?"
* Select packages:

   * *1:* Dependencies (required)
   * *2:* Xenomai setup (required)
   * *3/4:* Kernel patch (choose based on your CPU)

Then reboot into the Xenomai-patched kernel:

```bash
sudo reboot
```

After reboot, make sure you're running the Xenomai kernel:

```bash
uname -r
```

### 2. Install EtherCAT Stack

Run the build script:

```bash
./xenomai-ethercat-setup/build_ecat.sh
```

Configuration options:

   * Enter your workspace name (e.g., code_ws) and parent directory
   * Answer **YES** to "Enable Xenomai Real-Time?"
   * Answer **NO** to "Build GUI tools?" (recommended for embedded systems)
   * Select all packages (and dependencies) for a full installation: 1 2 3 4 5 6

This installs both the EtherCAT master and client libraries.

### 3. Configure Network Interface

EtherCAT requires direct control of a network card, bypassing the standard Linux network stack.

Identify your network card:

```bash
lspci -nkv 
```

Edit configuration script:

```bash
vim /usr/local/bin/ec_xeno3.sh
# ETH_DRV - set the correct driver for your network card (e.g., e1000e, igb, etc.)
# Verify REBIND_RT_NICS matches your network card
```

### 4. Start Xenomai Service

Restart the Xenomai service to apply everything:

```bash
systemctl restart xeno.service
```

You can check its status with:

```bash
systemctl status xeno.service
```

### 5. Load Environment

Make sure your environment variables are loaded:

```bash
source ~/.bashrc
```
----------------------------------------------------------------------

## Non-RealTime: Embedded/PC

Same as RealTime Embedded but:

* Skip "Install Xenomai Kernel", "Network Interface Configuration", and "Start Xenomai Service" 
* Answer **NO** to "Enable Xenomai Real-Time?" in *build_ecat.sh*

----------------------------------------------------------------------

## Manual Installation

This section describes how to manually install the EtherCAT Stack on an **Ubuntu system**. The following manual setup replicates the behavior of the provided automated installer script (*build_ecat.sh*).

### 1. Create the Workspace

Run the `create_ws.sh` script to generate the workspace structure:

```bash
git clone https://github.com/Advanced-Robotics-Facility/xenomai-ethercat-setup.git
./xenomai-ethercat-setup/create_ws.sh [name_ws] [dir_ws, default=HOME]
```

Add the generated workspace setup to your shell environment:

```bash
source ~/<workspace_name>/setup.bash 
```

Then use the **src** dir to place all the cloned repositories, and **build** dir (creating the relative package sub-directory (**mkdir package_name**)) for compiling them.

### 2. Dependencies

This section includes all system dependencies required for building and running the stack.

**Basic development tools**

```bash
sudo apt update
sudo apt install -y git gitg git-gui
sudo apt install -y build-essential curl cmake cmake-curses-gui
sudo apt install -y libgtest-dev
```

**Networking tools**

```bash
sudo apt install -y net-tools openssh-server sshpass curl \
                     gnome-terminal terminator
```

**YAML and formatting libraries**

```bash
sudo apt install -y libyaml-cpp-dev libfmt-dev
```

**Mechanism Protocols: Zmq, Protobuf, Msgpack, Boost**

```bash
sudo apt install -y libzmq3-dev protobuf-compiler libmsgpack-dev libboost-system-dev
```

**Eigen3**

```bash
sudo apt install -y libeigen3-dev
```

**MatLogger**

```bash
sudo apt install -y libmatio-dev python3-pip
```

**GUI tools (Qt)**

```bash
sudo apt install -y qt6-tools-dev qt6-declarative-dev \
                     libqt6charts6-dev uuid-dev libtiff-dev qttools5-dev
```

### 3. Clone, Build, and Install Packages

Create the build folders:

```bash
cd build/
mkdir -p SOEM cppzmq MatLogger2 ecat-master-advr ecat-client-advr
```

Clone the required packages with the correct branches:

```bash
git clone -b xeno3 https://github.com/alessiomargan/SOEM
git clone -b v4.7.1 https://github.com/zeromq/cppzmq
git clone -b master https://github.com/ADVRHumanoids/MatLogger2
git clone -b feature/cia402_rev https://github.com/Advanced-Robotics-Facility/ecat-master-advr
git clone -b feature/novanta https://github.com/Advanced-Robotics-Facility/ecat-client-advr
```

**Note:** *ecat-master-advr* is a private repository. Ask to Advanced-Robotics-Facility for the access.

**SOEM**

```bash
cd build/SOEM
cmake ../../src/SOEM 
      -DCMAKE_INSTALL_PREFIX=<path-to-install-folder> \
      -DCMAKE_USE_XENOMAI=ON
make -j$(nproc) install
```

**Note:** *-DCMAKE_USE_XENOMAI*=OFF if Non-RealTime

**cppzmq**

```bash
cd build/cppzmq
cmake ../../src/cppzmq 
      -DCMAKE_BUILD_TYPE="Release" \
      -DCMAKE_INSTALL_PREFIX=<path-to-install-folder> \
      -DCPPZMQ_BUILD_TESTS=OFF
make -j$(nproc) install
```

**MatLogger2**

```bash
cd build/MatLogger2 
cmake ../../src/MatLogger2
      -DCMAKE_BUILD_TYPE="Release" \
      -DCMAKE_INSTALL_PREFIX=<path-to-install-folder> \
      -DCOMPILE_PY_BINDINGS=OFF
make -j$(nproc) install
```

**ecat-master-advr** 

```bash
cd build/ecat-master-advr
cmake ../../src/ecat-master-advr 
      -DCMAKE_BUILD_TYPE="Release" \
      -DCMAKE_INSTALL_PREFIX=<path-to-install-folder>  \
      -DBUILD_UDP_SRV=ON \
      -DENABLE_XENO=ON   
make -j$(nproc) install
```

**Note:** *-DENABLE_XENO*=OFF if Non-RealTime

**ecat-client-advr**

```bash
cd build/ecat-client-advr
cmake ../../src/ecat-client-advr 
      -DCMAKE_BUILD_TYPE="Release" \
      -DCMAKE_INSTALL_PREFIX=<path-to-install-folder>  \
      -DCMAKE_USE_XENOMAI=ON \ 
      -DENABLE_XENO=ON \ 
      -DCOMPILE_GUI=OFF                              
make -j$(nproc) install
```

**Note1:** *-DENABLE_XENO*=OFF, *-DCMAKE_USE_XENOMAI*=OFF if Non-RealTime

**Note2:** *-DCOMPILE_GUI*=ON if you need the GUI

### 4. Environmental Variable

Export environment variable EC_CFG:

```bash
export EC_CFG=~/<workspace_name>/src/ecat-client-advr/config/ec_cfg.yaml
```

-----------------------------------

## Docker

To install Docker launch the `prepare.sh` file and just choose "install docker" option:

```bash
git clone https://github.com/Advanced-Robotics-Facility/xenomai-ethercat-setup.git
./xenomai-ethercat-setup/xenomai/prepare.sh
```

To build and run everything, you have two paths:

### Docker Compose

```bash
# Xenomai + EtherCAT Stack
cd xenomai-ethercat-setup/docker
docker build -f Dockerfile.xenomai -t xenomai:latest .
DOCKER_BUILDKIT=1 docker compose up --build
docker exec -it ecat-container /bin/bash
```

### Manual Procedure

```bash
# Xenomai Docker Image
cd xenomai-ethercat-setup/docker
docker build -f Dockerfile.xenomai -t xenomai:latest .

# EtherCAT Stack Docker Image inheriting from Xenomai
cd ..
DOCKER_BUILDKIT=1 docker build --ssh default -f docker/Dockerfile.ethercat -t ecat-dev:latest .
docker run --rm -it --name ecat-container --privileged --network host -v /dev:/dev ecat-dev:latest
```

To start a new shell:

```bash
docker exec -it ecat-container /bin/bash
```