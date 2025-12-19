# Quick Start - Raspberry Pi SLAM Mapping Setup

## Pre-Deployment Checklist

### Hardware Setup
- [ ] Raspberry Pi 4 (4GB+ RAM recommended)
- [ ] RPLiDAR A1M8 (USB or serial connection)
- [ ] Arduino board (UNO/Nano) connected via USB
- [ ] Motor driver L298N wired to GPIO pins
- [ ] 2-4 wheels with motors
- [ ] Power supply setup (separate for Pi and motors)
- [ ] Network connectivity (WiFi or Ethernet)

### Software Prerequisites
```bash
# Ubuntu 22.04 ARM64 on Raspberry Pi
# ROS 2 Humble installed
# Build tools installed
# Arduino libraries configured
```

---

## 1. Copy Package to Raspberry Pi

### From Development Machine (Windows)
```powershell
# Install WinSCP or use SCP from Windows
scp -r c:\Users\Admin\Desktop\car_project_19122025\articubot_one\
  pi@raspberrypi.local:~/ros2_ws/src/articubot_one
```

### Or Copy Manually
1. Navigate to `c:\Users\Admin\Desktop\car_project_19122025\articubot_one`
2. Zip the folder
3. SCP to Pi: `~/ros2_ws/src/articubot_one`
4. Unzip on Pi

### Verify on Raspberry Pi
```bash
ls -la ~/ros2_ws/src/articubot_one
# Should show: CMakeLists.txt, package.xml, description/, launch/, config/
```

---

## 2. Build Package on Raspberry Pi

```bash
# Navigate to workspace
cd ~/ros2_ws

# Build package
colcon build --packages-select articubot_one

# Source setup
source install/setup.bash
```

**Expected Output**:
```
Starting >>> articubot_one
Finished <<< articubot_one [2.5s]
Summary: 1 package finished [2.5s]
```

---

## 3. Test LiDAR Connection

### Check RPLiDAR Connection
```bash
# List USB devices
lsusb
# Look for: CH340 (USB serial) or similar

# Or check serial devices
ls -la /dev/tty*
# Should see: /dev/ttyUSB0 or /dev/ttyACM0
```

### Grant Permissions (if needed)
```bash
sudo chmod 666 /dev/ttyUSB0
# or
sudo usermod -a -G dialout $USER
# Then reboot Pi
```

### Launch RPLiDAR Driver
```bash
# Terminal 1: Start roscore (ROS2 only, skip for humble)
# ROS2 Humble doesn't need roscore

# Terminal 1: Robot State Publisher
ros2 launch articubot_one rsp.launch.py

# Terminal 2: RPLiDAR Driver
ros2 launch articubot_one rplidar.launch.py
```

**Expected Output from Terminal 2**:
```
[rplidar-1] [2025-12-19 10:15:33] RPLIDAR running...
[rplidar-1] [2025-12-19 10:15:33] RPLIDAR S1 detected
```

### Verify in ROS2
```bash
# Terminal 3: Check topics
ros2 topic list
# Should show: /scan, /tf, /tf_static

# View laser scan data
ros2 topic echo /scan | head -20
```

---

## 4. Start SLAM Mapping

### Launch All Components

```bash
# Terminal 1: Robot State Publisher
ros2 launch articubot_one rsp.launch.py

# Terminal 2: RPLiDAR Driver
ros2 launch articubot_one rplidar.launch.py

# Terminal 3: SLAM Toolbox
ros2 launch articubot_one online_async_launch.py
```

### Monitor SLAM Status
```bash
# Terminal 4: Check SLAM topics
ros2 topic list | grep slam
# Should show: /map, /map_metadata, /pose, /particles

# Monitor mapping performance
ros2 service list | grep slam
```

---

## 5. Visualize on Desktop

### From Desktop Machine (Windows)
```bash
# Setup ROS2 environment
set ROS_DOMAIN_ID=0
set ROS_LOCALHOST_ONLY=0

# Connect to Pi
ping raspberrypi.local

# Set ROS_MASTER_URI (for ROS1 only, skip for ROS2)
# ROS2 uses domain ID instead

# Start RViz
rviz2 -d path\to\config\map.rviz

# Or
ros2 launch articubot_one launch_robot.launch.py
```

### From Ubuntu Machine
```bash
# Export ROS2 domain
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0

# Launch RViz with mapping config
ros2 launch articubot_one launch_robot.launch.py

# Or manually
rviz2 -d ~/ros2_ws/src/articubot_one/config/map.rviz
```

---

## 6. Drive the Robot

### Using Joystick (if available)
```bash
# Terminal 5: Launch joystick
ros2 launch articubot_one joystick.launch.py

# Drive robot using controller
# See joystick.yaml for button mappings
```

### Manual Command
```bash
# Terminal 5: Send velocity commands
ros2 topic pub /cmd_vel geometry_msgs/Twist "{linear: {x: 0.2, y: 0.0, z: 0.0}, angular: {z: 0.1}}"
```

---

## 7. Save and Manage Maps

### Map Files Location
```bash
# Maps are saved in:
~/ros2_ws/src/articubot_one/maps/
# Files: my_map.pgm (image), my_map.yaml (metadata)
```

### Save Current Map
```bash
# Terminal: Use slam_toolbox service
ros2 service call /slam_toolbox/save_map \
  slam_toolbox/srv/SaveMap \
  "name: /tmp/my_map"
```

### Load Existing Map
```bash
# Modify nav2_params.yaml and set:
map_server:
  map: /tmp/my_map.yaml

# Then launch localization mode
ros2 launch articubot_one localization_launch.py
```

---

## 8. Troubleshooting

### LiDAR Not Found
```bash
# Check port
ls /dev/tty*

# Test connection
cat /dev/ttyUSB0

# Check permissions
sudo chmod 666 /dev/ttyUSB0
```

### SLAM Not Generating Map
- Check `/scan` topic publishing (should be 8-10 Hz)
- Verify TF tree: `ros2 run tf2_tools view_frames`
- Ensure robot moving at least 0.5m before loop closure

### Slow Mapping on Raspberry Pi
- Reduce `resolution` in mapper_params_online_async.yaml
- Lower `scan_match_quality` threshold
- Increase `minimum_travel_distance` to skip frames

### Network Issues
- Check Pi IP: `hostname -I`
- Ping Pi: `ping raspberrypi.local`
- Check ROS_DOMAIN_ID matches (default: 0)
- Ensure same WiFi network

---

## 9. Parameter Tuning

### mapper_params_online_async.yaml Key Parameters

```yaml
# Mapping resolution (m) - Lower = larger file, faster
resolution: 0.05  # 5cm grid cells

# Max laser range (m)
max_laser_range: 6.0

# Minimum movement before publishing map
minimum_travel_distance: 0.5

# Loop closure parameters
do_loop_closing: true
loop_search_maximum_distance: 3.0

# Scan matching quality (0.0-1.0)
scan_match_quality: 0.7
```

### For Slow Pi or Large Areas
```yaml
# Reduce resolution for faster mapping
resolution: 0.1  # 10cm grid cells

# Skip more frames
minimum_travel_distance: 1.0

# Reduce loop closure complexity
loop_search_maximum_distance: 2.0
```

---

## 10. Next Steps

1. ✅ Verify LiDAR connection and data
2. ✅ Run SLAM and generate initial map
3. ✅ Drive robot around environment (figure-8 pattern recommended)
4. ✅ Save final map
5. ⏭️ Deploy Nav2 for autonomous navigation (Phase 2)
6. ⏭️ Implement local obstacle avoidance
7. ⏭️ Integrate path planning

---

## Useful Commands Reference

```bash
# List all ROS2 nodes
ros2 node list

# List all topics
ros2 topic list

# Monitor specific topic
ros2 topic echo /scan

# Check TF tree
ros2 run tf2_tools view_frames

# List services
ros2 service list

# Call service
ros2 service call /slam_toolbox/clear_queue std_srvs/Empty

# View launch file dependencies
ros2 launch articubot_one online_async_launch.py --show-args

# Kill all ROS2 nodes
killall -9 ros2
```

---

**Status**: Ready for deployment!

For detailed guides, see:
- `../MAPPING_GUIDES/DEPLOY_ON_RASP.md` - Full deployment guide
- `../MAPPING_GUIDES/HUONG_DAN_MAPPING.md` - Detailed mapping procedures
- `../MAPPING_GUIDES/MAPPING_FAQ_TROUBLESHOOTING.md` - Common issues
