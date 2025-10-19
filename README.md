## Livox MID-360 - Quick Start Guide

This guide walks through connecting a Livox MID-360 LiDAR, recording its point cloud, and replaying the data inside RViz. Follow the steps in order; every command assumes a freshly opened terminal unless stated otherwise.

---

### 1. Requirements
- Ubuntu 24.04 with ROS 2 **Jazzy** already installed.
- Livox MID-360 sensor powered and wired directly to your PCâ€s Ethernet port.
- Packages:
  ```bash
  sudo apt update
  sudo apt install cmake python3-colcon-common-extensions ros-jazzy-pcl-ros
  ```

---

### 2. Prepare the Workspace
```bash
mkdir -p ~/livox_ws/src
cd ~/livox_ws/src
git clone https://github.com/atinfinity/livox_ros_driver2.git
```

Install the Livox SDK2 (required by the driver):
```bash
cd ~
git clone https://github.com/atinfinity/Livox-SDK2.git
cd Livox-SDK2
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j
sudo make install
sudo ldconfig
```

Build the ROS workspace:
```bash
cd ~/livox_ws
rosdep install --from-paths src --ignore-src -r -y --rosdistro jazzy
colcon build --symlink-install
```

---

### 3. Configure the Network
1. List existing NetworkManager profiles:
   ```bash
   nmcli connection show
   ```
2. Apply a static IP to the wired profile (adjust interface name if different):
   ```bash
   sudo nmcli connection modify "Wired connection 1" \
       connection.interface-name enp1s0 \
       ipv4.method manual \
       ipv4.addresses 192.168.1.5/24 \
       ipv4.gateway "" \
       ipv4.dns "" \
       connection.autoconnect yes

   sudo nmcli connection down "Wired connection 1"
   sudo nmcli connection up "Wired connection 1"
   ```
3. Confirm the address:
   ```bash
   ip a show enp1s0
   ```
4. Ping the MID-360 (default IP is `192.168.1.1XX`, where `XX` are the final digits of the serial number):
   ```bash
   ping 192.168.1.127
   ```

---

### 4. Update the Sensor Configuration
Edit `src/livox_ros_driver2/config/MID360_config.json` so the IP matches your sensor, for example:
```json
{
  "lidar_configs" : [
    {
      "ip" : "192.168.1.127",
      "pcl_data_type" : 1,
      "pattern_mode" : 0,
      "frame_id" : "livox_frame",
      ...
    }
  ]
}
```

Adjust any other fields (publish frame, extrinsics) as needed.

---

### 5. Source ROS and the Workspace
Run these commands in every terminal where you use ROS tools:
```bash
source /opt/ros/jazzy/setup.bash
source ~/livox_ws/install/setup.bash
```

---

### 6. Run the Driver and Visualize
Terminal 1:
```bash
ros2 launch livox_ros_driver2 rviz_MID360_launch.py
```
This launches `livox_ros_driver2_node` and RViz. RViz subscribes to `/livox/lidar` and displays the PointCloud2 stream.

Optional checks (Terminal 2 after sourcing ROS):
```bash
ros2 topic list
ros2 topic info /livox/lidar --verbose
ros2 topic echo /livox/lidar --qos-reliability best_effort
```

---

### 7. Record Point Clouds (Manual Method)
In a second sourced terminal, record the point cloud topic:
```bash
ros2 bag record --topics /livox/lidar --storage sqlite3 --output mid360_capture
```

Press `Ctrl+C` to stop recording once you have enough data.

---

### 9. Replay a Recording
Terminal 1:
```bash
ros2 bag play mid360_capture
```

Terminal 2:
```bash
rviz2 -d ~/livox_ws/src/livox_ros_driver2/config/display_point_cloud.rviz
```
RViz will show the previously recorded `/livox/lidar` data.

---

### 10. Troubleshooting
- **No points in RViz**: check the sensor IP, ensure `ping` succeeds, and verify `/livox/lidar` appears in `ros2 topic list`.
- **Bag is empty**: confirm `ros2 bag record` prints `Subscribed to topic [/livox/lidar]` and run `ros2 bag info <bag>` after recording.
- **QoS warnings**: use `ros2 topic echo /livox/lidar --qos-profile sensor_data` to match the sensorâ€s QoS.
- **Multiple sensors**: set `"multi_topic": 1` in the launch parameters or script options and add more entries to `lidar_configs`.


## Author

Sandesh Athawale<br>
Currently based in Tokyo, Japan 🇯🇵 and working as a System engineer.
