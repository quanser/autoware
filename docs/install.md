# QCar2-Autoware Installation Guide
## Prerequisites
You can connect to the QCar2 via HDMI + keyboard/mouse or Remote desktop (per [QCar2 User Manual](https://github.com/quanser/Quanser_Academic_Resources/blob/dev-qcar/3_user_manuals/qcar2/user_manual_connectivity.pdf)).
Ensure the QCar2 has internet access.

Before installing Autoware to your QCar2, the following prerequisites need to be installed. 

1. Install [CycloneDDS](./cyclonedds.md)

2. Set up [Quanser Academic Resources](./quanser_academic_resources.md)

## Autoware Universe Docker Installation
The following setup instruction for QCar2 is distilled from [Autoware Foundation](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/docker-installation/)
### 1. Clone this Autoware fork in the QCar2 branch
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

### 3. Start the development Docker container
```bash
./docker/run.sh --devel --headless --no-nvidia
```

### 4. Set up a work splace. 
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
### 6. Build workspace
```bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
**NOTE:** The entire build process may take up to 3 hours, it is recommended that the QCar2 is conneceted to the static power supply.
Once the modules are built, you can start the next step and run the example

## [Next - Run the Example](./run_example.md)
