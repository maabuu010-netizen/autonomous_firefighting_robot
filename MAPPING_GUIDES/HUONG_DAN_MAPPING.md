# 📚 HƯỚNG DẪN MAPPING (LẬP BẢN ĐỒ) - ARTICUBOT_ONE

## 🎯 Tổng quan về Mapping

**Mapping** là quá trình robot tạo ra một bản đồ 3D của môi trường xung quanh dựa trên dữ liệu từ LiDAR.

```
LiDAR phát tia laser → Nhận tia phản hồi → Xử lý dữ liệu → Tạo bản đồ
```

---

## 🔧 Thành phần chính trong Mapping

### 1. **SLAM Toolbox** (Simultaneous Localization and Mapping)
- **File launch**: `online_async_launch.py`
- **Công dụng**: Đồng thời lập bản đồ và định vị robot
- **Loại**: Async SLAM (xử lý không đồng bộ, nhanh hơn)

### 2. **LiDAR**
- **Topic**: `/scan` (dữ liệu laser từ RPLiDAR)
- **Công dụng**: Cung cấp dữ liệu khoảng cách xung quanh robot

### 3. **Odometry** (Đo quãng đường)
- **Topic**: `/odom`
- **Công dụng**: Ước tính vị trí robot từ chuyển động bánh xe

### 4. **Coordinate Frames** (Hệ tọa độ)
```
map (bản đồ)
  ↓
odom (quãng đường)
  ↓
base_footprint (điểm chân robot)
  ↓
base_link (trung tâm robot)
```

---

## 📋 Các bước Mapping chi tiết

### **BƯỚC 1: Chuẩn bị Robot**

✅ Bật Raspberry Pi
✅ Kết nối SSH đến Raspberry Pi
✅ Kiểm tra LiDAR chạy tốt:
```bash
# Kiểm tra topic /scan
ros2 topic list | grep scan
ros2 topic echo /scan  # xem dữ liệu LiDAR
```

---

### **BƯỚC 2: Khởi động Robot State Publisher**

```bash
ros2 launch articubot_one rsp.launch.py use_sim_time:=false use_ros2_control:=false
```

**Công dụng**: 
- Công bố mô hình URDF của robot
- Tạo các transform giữa các khớp

**Kiểm tra**:
```bash
ros2 topic list | grep robot_description
```

---

### **BƯỚC 3: Khởi động SLAM Toolbox**

```bash
ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

**Những gì xảy ra**:

1. **async_slam_toolbox_node** khởi động
2. Lắng nghe topic `/scan` từ LiDAR
3. Bắt đầu tạo bản đồ trong thời gian thực
4. Công bố frame **map** → **odom**

**File cấu hình**: `mapper_params_online_async.yaml`

---

### **BƯỚC 4: Khởi động RViz để Hình dung**

```bash
rviz2 -d /home/pi/ros2_ws/src/articubot_one/config/map.rviz
```

**Cách xem**:
1. Thêm **Map** layer → chọn topic `/map`
2. Thêm **LaserScan** layer → chọn topic `/scan`
3. Thêm **TF** layer → xem các frame

---

### **BƯỚC 5: Điều khiển Robot di chuyển**

Khởi động joystick hoặc điều khiển bằng command:

```bash
# Lựa chọn 1: Dùng joystick
ros2 launch articubot_one joystick.launch.py use_sim_time:=false

# Lựa chọn 2: Điều khiển bằng lệnh
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "linear: {x: 0.2, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}"
```

**Quan trọng**: Di chuyển robot từ từ, để SLAM có thời gian xử lý dữ liệu!

---

## 🔍 Các tham số quan trọng trong Mapping

### **Từ `mapper_params_online_async.yaml`**

| Tham số | Giá trị | Ý nghĩa |
|---------|--------|---------|
| `solver_plugin` | CeresSolver | Thuật toán giải quyết bản đồ |
| `odom_frame` | odom | Frame odometry |
| `map_frame` | map | Frame bản đồ |
| `base_frame` | base_footprint | Frame trung tâm robot |
| `scan_topic` | /scan | Topic dữ liệu laser |
| `mode` | localization | Chế độ (mapping hoặc localization) |
| `resolution` | 0.05 | Độ phân giải bản đồ (5cm mỗi ô) |
| `max_laser_range` | 20.0 | Tầm xa tối đa laser (20m) |
| `minimum_travel_distance` | 0.5 | Khoảng cách di chuyển trước khi scan mới (0.5m) |
| `minimum_travel_heading` | 0.5 | Góc xoay trước khi scan mới (0.5 rad) |
| `do_loop_closing` | true | Đóng vòng khi phát hiện nơi cũ |

---

## 📊 Luồng xử lý dữ liệu Mapping

```
LiDAR (/scan)
   ↓
SLAM Toolbox
   ↓
┌─────────────────────┐
│  Khớp Scan Matching │  → Tìm điểm giống nhau giữa các scan
└─────────────────────┘
   ↓
┌──────────────────┐
│  Loop Detection  │  → Phát hiện nơi robot đã đến trước đó
└──────────────────┘
   ↓
┌─────────────────┐
│  Map Optimization │ → Sửa lỗi và tối ưu bản đồ
└─────────────────┘
   ↓
Map Frame → RViz
   ↓
RViz hiển thị bản đồ + LiDAR scans
```

---

## 💾 Lưu bản đồ

### **Tự động lưu** (nếu cấu hình)

```bash
# File bản đồ sẽ được lưu tại:
# /home/pi/dev_ws/my_map_serial.pgm  (ảnh bản đồ)
# /home/pi/dev_ws/my_map_serial.yaml (metadata)
```

### **Lưu bản đồ thủ công**

```bash
# Sau khi mapping xong, lưu bản đồ
ros2 service call /slam_toolbox/save_map slam_toolbox/srv/SaveMap "{name: {data: '/home/pi/my_robot_map'}}"
```

---

## 🎮 Mô phỏng Mapping trong Gazebo

Nếu bạn muốn test mapping mà không cần robot thật:

```bash
# Terminal 1: Khởi động Gazebo
ros2 launch articubot_one launch_sim.launch.py

# Terminal 2: Khởi động SLAM
ros2 launch articubot_one online_async_launch.py use_sim_time:=true

# Terminal 3: RViz
rviz2 -d /path/to/map.rviz

# Terminal 4: Điều khiển robot di chuyển
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "linear: {x: 0.2, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.5}"
```

---

## ⚠️ Các vấn đề thường gặp và cách khắc phục

### **1. Bản đồ bị méo, không chính xác**
- **Nguyên nhân**: Robot di chuyển quá nhanh, SLAM không kịp xử lý
- **Giải pháp**: Giảm tốc độ robot, di chuyển từ từ

### **2. Bản đồ có "khoảng trắng"**
- **Nguyên nhân**: LiDAR bị chắn, hoặc có vật thể di động
- **Giải pháp**: Di chuyển nhiều lần qua khu vực đó

### **3. Transform errors (TF errors)**
- **Nguyên nhân**: Frame không được broadcast đúng
- **Kiểm tra**: 
```bash
ros2 run tf2_tools view_frames
# Xem file frame.pdf để kiểm tra
```

### **4. Loop closing không hoạt động**
- **Nguyên nhân**: Tham số `loop_search_maximum_distance` quá nhỏ
- **Giải pháp**: Tăng giá trị hoặc vẽ đường đi có vòng lặp rõ ràng

---

## 📝 Kiểm tra dữ liệu Mapping

```bash
# Xem tất cả topics
ros2 topic list

# Xem mức đỉnh của /map
ros2 topic hz /map

# Xem độc lập của transform
ros2 run tf2_ros tf2_monitor map base_footprint
```

---

## 🎯 Quy trình Complete Mapping

```
┌──────────────────────────────────────────────────────┐
│  1. Chuẩn bị Hardware                               │
│     - Bật Robot, LiDAR, Raspberry Pi                │
└──────────────────────────────────────────────────────┘
           ↓
┌──────────────────────────────────────────────────────┐
│  2. Khởi động ROS Nodes                             │
│     - Robot State Publisher (rsp.launch.py)         │
│     - SLAM Toolbox (online_async_launch.py)         │
│     - RViz (hiển thị bản đồ)                        │
└──────────────────────────────────────────────────────┘
           ↓
┌──────────────────────────────────────────────────────┐
│  3. Di chuyển Robot                                 │
│     - Joystick hoặc cmd_vel commands                │
│     - Đi theo hình xoắn/S để quét toàn bộ khu vực  │
│     - Quay vòng để mapping đầy đủ (360°)            │
└──────────────────────────────────────────────────────┘
           ↓
┌──────────────────────────────────────────────────────┐
│  4. Kiểm tra Bản đồ                                 │
│     - RViz hiển thị bản đồ đầy đủ?                  │
│     - Loop closing hoạt động?                       │
│     - Không có lỗi tf?                              │
└──────────────────────────────────────────────────────┘
           ↓
┌──────────────────────────────────────────────────────┐
│  5. Lưu Bản đồ                                      │
│     - Sử dụng service call hoặc tự động             │
│     - Kiểm tra file .pgm và .yaml                   │
└──────────────────────────────────────────────────────┘
```

---

## 💡 Mẹo để Mapping tốt hơn

✅ **Di chuyển chậm**: Cho phép SLAM xử lý dữ liệu
✅ **Di chuyển trên đường hình xoắn**: Quét toàn bộ khu vực
✅ **Quay vòng**: Đảm bảo mapping 360° quanh robot
✅ **Tránh các bề mặt phản xạ**: Giảm nhiễu laser
✅ **Sạch khu vực**:Tránh đồ vật di động
✅ **Di chuyển nhiều lần**: Qua các khu vực phức tạp để độ chính xác

---

## 📚 Tài liệu tham khảo

- SLAM Toolbox: https://github.com/SteveMacenski/slam_toolbox
- ROS2 Navigation: https://navigation.ros.org/
- Cartographer (SLAM mạnh mẽ khác): https://google-cartographer-ros.readthedocs.io/

---

**Bây giờ bạn đã hiểu được Mapping hoạt động như thế nào! Hãy thử từng bước trên robot của bạn.** 🚀
