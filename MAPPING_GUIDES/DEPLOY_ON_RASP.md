# 📦 HƯỚNG DẪN ĐƯA SOURCE VÀO RASPBERRY PI & LAUNCH MAPPING

## **PHẦN 1: ĐƯA SOURCE VÀO RASPBERRY PI**

### **Cách 1: SCP (Copy từ Laptop sang Rasp) - ⭐ Khuyến nghị**

#### **Step 1.1: Kiểm tra kết nối SSH**
```bash
# Từ Laptop (Windows PowerShell)
ssh pi@192.168.x.x
# Hoặc
ssh pi@articubot.local

# Nếu kết nối được → OK, nhập password
# Nếu không kết nối → Kiểm tra IP Rasp
```

#### **Step 1.2: Tìm đường dẫn ROS2 workspace trên Rasp**
```bash
# SSH vào Rasp
ssh pi@192.168.x.x

# Kiểm tra workspace
ls -la ~/
# Tìm: ros2_ws, colcon_ws, dev_ws, v.v...

# Hoặc kiểm tra ~/.bashrc
grep "install/setup.bash" ~/.bashrc
# Sẽ hiện: source ~/ros2_ws/install/setup.bash
# → Workspace là ~/ros2_ws
```

#### **Step 1.3: Copy project từ Laptop sang Rasp**

**Trên Laptop (PowerShell):**

```powershell
# Di chuyển đến thư mục project
cd c:\Users\Admin\Desktop\car_project_19122025

# Copy folder articubot_one sang Rasp
# (Đầu tiên backup cái cũ)
scp -r articubot_one pi@192.168.x.x:~/ros2_ws/src/articubot_one_backup
# hoặc
scp -r articubot_one pi@192.168.x.x:~/ros2_ws/src/articubot_one

# Nếu Rasp đã có folder, ghi đè:
# scp -r articubot_one\ pi@192.168.x.x:~/ros2_ws/src/

# Kiểm tra (sau ~1-2 phút)
ssh pi@192.168.x.x
ls ~/ros2_ws/src/articubot_one/
# Phải thấy: CMakeLists.txt, package.xml, launch/, config/, v.v...
```

**Hoặc một lần (copy toàn bộ project):**
```powershell
scp -r articubot_one pi@192.168.x.x:~/ros2_ws/src/
```

---

### **Cách 2: Git Clone (Nếu project ở GitHub)**

Nếu project của bạn ở trên GitHub:

```bash
# SSH vào Rasp
ssh pi@192.168.x.x

# Vào workspace
cd ~/ros2_ws/src

# Clone project
git clone https://github.com/joshnewans/articubot_one.git
# hoặc
git clone https://github.com/<your-username>/articubot_one.git

# Kiểm tra
ls articubot_one/
```

---

### **Cách 3: USB Drive (Nếu Rasp không có internet)**

1. Copy folder `articubot_one` vào USB drive
2. Cắm USB vào Rasp
3. Vào Rasp qua SSH/Console
4. Mount USB drive
5. Copy từ USB vào ~/ros2_ws/src/

---

## **PHẦN 2: COMPILE PROJECT TRÊN RASP**

### **Step 2.1: SSH vào Rasp**
```bash
ssh pi@192.168.x.x
# Hoặc nếu dùng console trực tiếp
```

### **Step 2.2: Vào workspace**
```bash
cd ~/ros2_ws
```

### **Step 2.3: Build project**

**Lần đầu (tốn thời gian 5-10 phút):**
```bash
colcon build --packages-select articubot_one
```

**Lần sau (nhanh hơn):**
```bash
colcon build --packages-select articubot_one --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

**Nếu có lỗi:**
```bash
# Clean và rebuild
rm -rf build install log
colcon build --packages-select articubot_one
```

### **Step 2.4: Kiểm tra build thành công**
```bash
# Phải thấy output:
# Summary: 1 package finished
# [100%] PASSED

# Nếu FAILED → xem error
tail -n 50 log/latest_build/articubot_one/stdout.log
```

### **Step 2.5: Source workspace**
```bash
source ~/ros2_ws/install/setup.bash

# Hoặc thêm vào ~/.bashrc (tự động)
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## **PHẦN 3: LAUNCH MAPPING**

### **Cách A: 5 Terminal (Khuyến nghị)**

Mở 5 cửa sổ terminal (SSH riêng hoặc Console + SSH):

#### **Terminal 1: Robot State Publisher**
```bash
ssh pi@192.168.x.x
# hoặc console trực tiếp

cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one rsp.launch.py use_sim_time:=false use_ros2_control:=false
```

**Output kỳ vọng:**
```
[robot_state_publisher-1] [INFO] Robot Description loaded from URDF file
[robot_state_publisher-1] [INFO] Publishing TF frames
```

---

#### **Terminal 2: LiDAR**
```bash
ssh pi@192.168.x.x

cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one rplidar.launch.py
```

**Output kỳ vọng:**
```
[rplidar_node-1] [INFO] RPLIDAR S2 E1 S/N: ...
[rplidar_node-1] [INFO] RPLIDAR is running at 8Hz
[rplidar_node-1] [INFO] Publishing laser scans to /scan
```

---

#### **Terminal 3: SLAM Toolbox (CHÍNH)**
```bash
ssh pi@192.168.x.x

cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

**Output kỳ vọng:**
```
[async_slam_toolbox_node-1] [INFO] SLAM: Starting solver ...
[async_slam_toolbox_node-1] [INFO] SLAM: Processing first scan ...
[async_slam_toolbox_node-1] [INFO] SLAM: Publishing map ...
```

---

#### **Terminal 4: RViz (Từ Laptop)**
```bash
# Từ Laptop Windows/Linux/Mac (không phải Rasp)
export ROS_DOMAIN_ID=0  # Phải giống Rasp
export ROS_LOCALHOST_ONLY=0  # Cho phép kết nối qua network

rviz2 -d ~/path/to/map.rviz
# hoặc
rviz2
# Rồi add Map, LaserScan, TF layers
```

**Nếu không thấy topics:**
```bash
# Kiểm tra ROS_DOMAIN_ID (phải giống nhau)
echo $ROS_DOMAIN_ID
# Nếu trống → set ROS_DOMAIN_ID=0

# Kiểm tra localhost
echo $ROS_LOCALHOST_ONLY
# Nếu 1 → set ROS_LOCALHOST_ONLY=0
```

---

#### **Terminal 5: Điều khiển Robot**

**Option A: Joystick (nếu có)**
```bash
ssh pi@192.168.x.x

cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one joystick.launch.py use_sim_time:=false
```

**Option B: Manual Command**
```bash
ssh pi@192.168.x.x

cd ~/ros2_ws
source install/setup.bash

# Đi thẳng
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"

# Quay vòng
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}}"

# Dừng
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"
```

---

### **Cách B: Launch All (1 Terminal - Lần đầu phức tạp)**

Nếu muốn tất cả cùng 1 Terminal:

```bash
ssh pi@192.168.x.x

cd ~/ros2_ws
source install/setup.bash

# Khởi động tất cả:
# (Nhưng Terminal sẽ bị block, không thể nhập lệnh khác)
ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

⚠️ **Vấn đề**: Bạn không thể điều khiển robot vì Terminal bị block
💡 **Giải pháp**: Dùng `&` để chạy background (nếu là bash script)

---

## **PHẦN 4: KIỂM TRA MAPPING HOẠT động**

### **Step 1: Kiểm tra Topics (Terminal mới)**
```bash
ssh pi@192.168.x.x

# Xem tất cả topics
ros2 topic list

# Phải thấy:
# /map
# /map_metadata
# /scan
# /tf
# /tf_static
# /cmd_vel (nếu joystick chạy)
```

### **Step 2: Kiểm tra LiDAR data**
```bash
# Xem dữ liệu scan
ros2 topic echo /scan --once | head -30

# Phải thấy:
# ranges: [3.5, 3.4, 3.3, ..., 2.1, 2.0]
# angle_increment: 0.00157...
# angle_min: -3.14...
```

### **Step 3: Kiểm tra Map**
```bash
# Xem metadata bản đồ
ros2 topic echo /map_metadata

# Phải thấy:
# resolution: 0.05
# width: 2000
# height: 2000
```

### **Step 4: Kiểm tra Transform**
```bash
# Xem frame tree
ros2 run tf2_tools view_frames
# (Tạo file frames.pdf)

# Hoặc kiểm tra transform cụ thể
ros2 run tf2_ros tf2_monitor map base_footprint
```

---

## **PHẦN 5: LƯỚI BẢNG ĐỒ**

Khi mapping xong:

### **Lưu bản đồ**
```bash
# Service call
ros2 service call /slam_toolbox/save_map slam_toolbox/srv/SaveMap "{name: {data: '/home/pi/my_map'}}"

# Output:
# result:
#   save_map: true
```

### **Kiểm tra file**
```bash
ls -la ~/my_map.*

# Phải thấy:
# my_map.pgm   (bản đồ ảnh)
# my_map.yaml  (metadata)
```

---

## **PHẦN 6: TROUBLESHOOTING**

### **❌ Lỗi: "Could not find package"**
```bash
# Nguyên nhân: Package chưa build
# Giải pháp:
cd ~/ros2_ws
colcon build --packages-select articubot_one
source install/setup.bash
```

### **❌ Lỗi: "Failed to find node 'async_slam_toolbox_node'"**
```bash
# Nguyên nhân: slam_toolbox chưa cài
# Giải pháp:
sudo apt update
sudo apt install ros-humble-slam-toolbox
# (Thay humble bằng ROS2 version của bạn)
```

### **❌ Lỗi: "Cannot connect to LiDAR"**
```bash
# Nguyên nhân: LiDAR không kết nối hoặc cổng sai
# Giải pháp:
lsusb  # Kiểm tra USB device
ls -la /dev/ttyUSB*  # Kiểm tra cổng serial

# Hoặc restart LiDAR:
sudo systemctl restart usb  # (Nếu có)
```

### **❌ Lỗi: "ROS2 not found"**
```bash
# Nguyên nhân: ~/.bashrc chưa source ROS2
# Giải pháp:
source ~/.bashrc
# hoặc
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

---

## **PHẦN 7: WORKFLOW HOÀN CHỈNH**

```
1️⃣  Laptop: Copy source sang Rasp (SCP)
    scp -r articubot_one pi@192.168.x.x:~/ros2_ws/src/

2️⃣  Rasp: Build project
    cd ~/ros2_ws
    colcon build --packages-select articubot_one
    source install/setup.bash

3️⃣  Rasp: Khởi động 3 Terminal (RSP, LiDAR, SLAM)
    Terminal 1: ros2 launch ... rsp.launch.py
    Terminal 2: ros2 launch ... rplidar.launch.py
    Terminal 3: ros2 launch ... online_async_launch.py

4️⃣  Laptop: Khởi động RViz
    rviz2 -d map.rviz

5️⃣  Rasp: Điều khiển robot (Terminal 5)
    ros2 topic pub /cmd_vel ...

6️⃣  Mapping xảy ra (RViz hiển thị bản đồ)

7️⃣  Lưu bản đồ
    ros2 service call /slam_toolbox/save_map ...
```

---

## **⚡ QUICK SUMMARY**

### **Tóm tắt 7 bước:**

1. **Copy**: `scp -r articubot_one pi@192.168.x.x:~/ros2_ws/src/`
2. **Build**: `colcon build --packages-select articubot_one`
3. **Source**: `source install/setup.bash`
4. **Terminal 1**: `ros2 launch ... rsp.launch.py`
5. **Terminal 2**: `ros2 launch ... rplidar.launch.py`
6. **Terminal 3**: `ros2 launch ... online_async_launch.py`
7. **Laptop**: `rviz2` + điều khiển robot = **MAPPING!**

---

**Bây giờ bạn đã biết cách đưa source vào Rasp và launch mapping! 🚀**
