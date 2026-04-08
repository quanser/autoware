# QCar2-Autoware Installation Guide
## Prerequisites
Connect to the QCar2 via HDMI + keyboard/mouse or Remote desktop (as per the [QCar2 User Manual](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-qcar/3_user_manuals/qcar2/user_manual_connectivity.pdf)).
Ensure the QCar2 is connected to the internet.

Before installing Autoware to your QCar2, ensure the following steps are completed: 

1. Install [CycloneDDS](./cyclonedds.md)

2. Set up [Quanser Academic Resources](./quanser_academic_resources.md)

## Autoware Universe Docker Installation
The following setup instruction for QCar2 is adapted from [Autoware Foundation](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/docker-installation/)

**NOTE:** The entire build process may take up to 3 hours, it is recommended that the QCar2 is connected to the static power supply. Steps 1-5 (setting up environment) take about 15 minutes, and step 6 (compiling) will take 2.5 hours to complete.
### 1. Clone the Autoware repository (QCar2 branch)
On the QCar, open a terminal and run the following commands:
```bash
cd ~/Documents
git clone -b quanser_qcar2 --single-branch https://github.com/quanser/autoware.git
cd autoware
```
### 2. Setup Dependencies
Run the following setup script to install all required dependencies:

```bash
./setup-dev-env.sh -y --no-nvidia docker
```

### 3. Start the Development Docker Container
```bash
./docker/run.sh --devel --headless --no-nvidia
```

### 4. Set up a Workspace
Create the src directory and clone repositories into it. Note: when entering the container, the default directory is /autoware.
```bash
cd /workspace
mkdir -p src
vcs import src < repositories/autoware.repos
```
### 5. Update system packages and install dependencies
```bash
sudo apt update && sudo apt upgrade
rosdep update
rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO
```
### 6. Build Workspace
```bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

Once the modules are built, you can start the next step and run the example

## [Next - Run the Example](./run_example.md)
