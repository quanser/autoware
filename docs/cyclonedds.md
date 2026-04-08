#  QCar2 Cyclone DDS Setup

This page contains instructions for installing and configuring Cyclone DDS for QCar2.

---
## Cyclone DDS Installation

### 1. Create Workspace
On the QCar, open a terminal and create a new ROS 2 workspace for Cyclone DDS. This workspace will contain the Cyclone DDS implementation and related packages.

```bash
mkdir -p ~/cyclonedds_ws/src
cd ~/cyclonedds_ws
cd src
```
### 2. Clone Cyclone DDS Repositories
Clone the necessary Cyclone DDS repositories. The `CYCLONEDDS_BRANCH` variable is used to determine the appropriate branch of Cyclone DDS to use based on the ROS 2 Humble release.

```bash
git clone https://github.com/ros2/rmw_cyclonedds -b humble
CYCLONEDDS_BRANCH=$(curl -s https://raw.githubusercontent.com/ros2/ros2/refs/heads/humble/ros2.repos | grep -A3 "eclipse-cyclonedds/cyclonedds:" | grep version: | awk '{print $2}')
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b "${CYCLONEDDS_BRANCH}"
cd ..
```
### 3. Build and Install Cyclone DDS
```bash
colcon build --packages-select cyclonedds --cmake-args -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_EXAMPLES=ON
sudo cmake --install build/cyclonedds
```
### 4. Build Remaining Workspace
```bash
source /opt/ros/humble/setup.bash
colcon build
```
## Environment Configuration

### 1. Save the following file as ~/cyclonedds.xml.

You may use the following command to create and open the file in vim:
```bash
vim ~/cyclonedds.xml
```
Then, paste the following into the file:    
```bash
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="any">
        <General>
            <Interfaces>
                <NetworkInterface autodetermine="false" name="eth0" priority="default" multicast="default" />
            </Interfaces>
        </General>
        <Discovery>
            <ParticipantIndex>none</ParticipantIndex>
        </Discovery>
        <Internal>
            <SocketReceiveBufferSize min="10MB"/>
            <Watermarks>
                <WhcHigh>500kB</WhcHigh>
            </Watermarks>
        </Internal>
    </Domain>
</CycloneDDS>
```
then save and exit vim by pressing the following keys in order:
1. ==`Esc`==    
2. ==`:wq`==
3. ==`Enter`== 
   
**NOTE:** Network interface (eth0) may need adjustment depending on your setup. You can check your network interfaces using the `ip a` command and adjust the `name` attribute accordingly.

### 2. Add the following to your ~/.bashrc on the QCar2:

```bash
source /home/nvidia/cyclonedds_ws/install/setup.bash
export ROS_DOMAIN_ID=0
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI=file:///$HOME/cyclonedds.xml
```

## Configure DDS setting

You may also further configure DDS settings to better utilize the DDS implementations for Autoware ([source](https://autowarefoundation.github.io/autoware-documentation/main/installation/additional-settings-for-developers/network-configuration/dds-settings/)).

### Enable multicast on lo

Enabling multicast on the loopback interface allows DDS participants on the same machine to communicate using multicast, which is required for proper discovery and communication in some Autoware applications


#### Option 1
You can call the following command to enable multicast on the loopback interface. This command sets up multicast on the loopback interface temporarily. You need to call this command every time you restart your system.
```bash
sudo ip link set lo multicast on
```
#### Option 2 
Alternatively, you can configure/set up a systemd service to enable multicast automatically when the system starts as a permanent solution.

Create a new systemd service file using the following command:

```bash
sudo vim /etc/systemd/system/multicast-lo.service
```
This should open a blank file if it doesn't already exist. Paste the following into the file:

```bash
[Unit]
Description=Enable Multicast on Loopback

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set lo multicast on

[Install]
WantedBy=multi-user.target
```
To save your changes, press ==`Esc`==, then type ==`:wq`==, and then press ==`Enter`==. 
Now you can start the service: 

```bash
# Make it recognized
sudo systemctl daemon-reload

# Make it run on startup
sudo systemctl enable multicast-lo.service

# Start it now
sudo systemctl start multicast-lo.service
```

To validate if the service file is configured correctly, you can check its status with following command:

```bash
sudo systemctl status multicast-lo.service
```
Your terminal should display something similar to the following:
```bash
○ multicast-lo.service - Enable Multicast on Loopback
     Loaded: loaded (/etc/systemd/system/multicast-lo.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Mon 2024-07-08 12:54:17 +03; 4s ago
    Process: 22588 ExecStart=/usr/bin/ip link set lo multicast on (code=exited, status=0/SUCCESS)
   Main PID: 22588 (code=exited, status=0/SUCCESS)
        CPU: 1ms

Tem 08 12:54:17 mfc-leo systemd[1]: Starting Enable Multicast on Loopback...
Tem 08 12:54:17 mfc-leo systemd[1]: multicast-lo.service: Deactivated successfully.
Tem 08 12:54:17 mfc-leo systemd[1]: Finished Enable Multicast on Loopback.
```
Press ==`Q`== to exit this process. Then check lo status with following command:

```bash
ip link show lo
```
Your terminal should display the following:
```bash
1: lo: <LOOPBACK,MULTICAST,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
### Tune system-wide network settings

Set the config file path and enlarge the Linux kernel maximum buffer size before launching Autoware.

#### Option 1
Use the following commands to temporarily set the network settings temporarily. You need to call this command every time you restart your system.
``` bash
# Increase the maximum receive buffer size for network packets
sudo sysctl -w net.core.rmem_max=2147483647  # 2 GiB, default is 208 KiB

# IP fragmentation settings
sudo sysctl -w net.ipv4.ipfrag_time=3  # in seconds, default is 30 s
sudo sysctl -w net.ipv4.ipfrag_high_thresh=134217728  # 128 MiB, default is 256 KiB
```
#### Option 2 
Alternatively, you can set the network settings permanently by creating a sysctl configuration file:

```bash
sudo vim /etc/sysctl.d/10-cyclone-max.conf
```
Paste the following into the file:
```bash
# Increase the maximum receive buffer size for network packets
net.core.rmem_max=2147483647  # 2 GiB, default is 208 KiB

# IP fragmentation settings
net.ipv4.ipfrag_time=3  # in seconds, default is 30 s
net.ipv4.ipfrag_high_thresh=134217728  # 128 MiB, default is 256 KiB
```
Then to save, press ==`Esc`==, then type ==`:wq`==, and then press ==`Enter`==. 

To validate the sysctl settings, run the following command:
```bash
sysctl net.core.rmem_max net.ipv4.ipfrag_time net.ipv4.ipfrag_high_thresh
```
Your terminal should show the following output:

```bash
net.core.rmem_max = 2147483647
net.ipv4.ipfrag_time = 3
net.ipv4.ipfrag_high_thresh = 134217728
```

## [Back to Installation Guide](./install.md)
