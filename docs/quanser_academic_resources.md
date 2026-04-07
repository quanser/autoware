# Quanser Academic Resources Setup Guide

### 1. Prepare the Documents Directory

Navigate to your Documents folder:
```bash
cd ~/Documents
```
If a previous Quanser folder exists, you can remove it:
```bash
rm -rf Quanser
```
or rename it for backup:
```bash
mv Quanser Quanser_old
```
### 2. Download the Repository
Clone the QCar branch from Quanser Academic Resources:
```bash
git clone -b dev-qcar --single-branch https://github.com/quanser/Quanser_Academic_Resources.git Quanser
```
This will create a Quanser folderand download all required files into it.

### 3. Run the Setup Script

Navigate to the setup directory:
```bash
cd ~/Documents/Quanser/1_setup
```
Run the setup script:
```bash
chmod +x updatebashrc_qcar2.sh
./updatebashrc_qcar2.sh
```
### 4. Setup QCar2 ROS2 Workspace
Go to the QCar ROS2 workspace:
```bash
cd ~/Documents/Quanser/5_research/sdcs/qcar2/ros2
```
Build QCar2 packages. Make sure you have already installed CycloneDDS before buidling the ROS2 packages.
```bash
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
```
From here you can run QCar2 hardware nodes.

## [Back to Installation Guide](./install.md)
