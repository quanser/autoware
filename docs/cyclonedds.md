#  QCar2 Cyclone DDS Setup

This section install and configures Cyclone DDS for QCar2.

---
## Cyclone DDS Installation
### 1. Create Workspace

```bash
mkdir -p ~/cyclonedds_ws/src
cd ~/cyclonedds_ws
cd src
```
### 2. Clone Cyclone DDS Repositories
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
**NOTE:** Network interface (eth0) may need adjustment depending on your setup.

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

You can call the following command to enable multicast on the loopback interface.
```bash
sudo ip link set lo multicast on
```
or use the following steps for a **permanent solution**:
```bash
sudo vim /etc/systemd/system/multicast-lo.service
```
Paste the following into the file:
```bash
[Unit]
Description=Enable Multicast on Loopback

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set lo multicast on

[Install]
WantedBy=multi-user.target
```
Press following in order to insert and save with vim:

1. `A`
2. `Ctrl + Shift + V`
3. `Esc`
4. type `:wq`
5. `Enter`

Now you can start the service: 
```bash
# Make it recognized
sudo systemctl daemon-reload

# Make it run on startup
sudo systemctl enable multicast-lo.service

# Start it now
sudo systemctl start multicast-lo.service
```
Validate:
```bash
you@pc:~$ sudo systemctl status multicast-lo.service
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
```bash
you@pc:~$ ip link show lo
1: lo: <LOOPBACK,MULTICAST,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

### Tune system-wide network settings

Set the config file path and enlarge the Linux kernel maximum buffer size before launching Autoware.
``` bash
# Increase the maximum receive buffer size for network packets
sudo sysctl -w net.core.rmem_max=2147483647  # 2 GiB, default is 208 KiB

# IP fragmentation settings
sudo sysctl -w net.ipv4.ipfrag_time=3  # in seconds, default is 30 s
sudo sysctl -w net.ipv4.ipfrag_high_thresh=134217728  # 128 MiB, default is 256 KiB
```
To make it permanent,
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

Validate the sysctl settings
```bash
user@pc$ sysctl net.core.rmem_max net.ipv4.ipfrag_time net.ipv4.ipfrag_high_thresh
net.core.rmem_max = 2147483647
net.ipv4.ipfrag_time = 3
net.ipv4.ipfrag_high_thresh = 134217728
```
## [Back to Installation Guide](./install.md)
