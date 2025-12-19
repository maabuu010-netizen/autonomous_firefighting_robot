# 🚀 HƯỚNG DẪN THỰC HÀNH - MAPPING TRÊN ROBOT THẬT

## **PHẦN 1: CHUẨN BỊ (Chỉ làm lần đầu)**

### Bước 1.1: SSH vào Raspberry Pi

```bash
# Từ máy tính của bạn
ssh pi@192.168.x.x  # Thay IP của Raspberry Pi
# Hoặc nếu biết hostname
ssh pi@articubot.local
```

### Bước 1.2: Kiểm tra ROS2 và các package

```bash
# Kiểm tra ROS2 đã cài đặt
ros2 --version

# Kiểm tra SLAM Toolbox đã cài
ros2 pkg find slam_toolbox

# Kiểm tra workspace
cd ~/ros2_ws
source install/setup.bash  # Activate workspace
```

### Bước 1.3: Kiểm tra LiDAR kết nối

```bash
# Mở terminal mới
ros2 launch articubot_one rplidar.launch.py

# Terminal khác: Kiểm tra dữ liệu
ros2 topic list
# Phải thấy: /scan

# Xem dữ liệu LiDAR
ros2 topic echo /scan
```

**Nếu thấy dữ liệu ranges có giá trị → LiDAR OK** ✅

---

## **PHẦN 2: MAPPING BƯỚC-BY-BƯỚC**

### **Terminal 1: Robot State Publisher** (Khởi động trước)

```bash
cd ~/ros2_ws
source install/setup.bash

# Khởi động RSP - công bố mô hình robot
ros2 launch articubot_one rsp.launch.py \
    use_sim_time:=false \
    use_ros2_control:=false
```

**Output kỳ vọng**:
```
[robot_state_publisher-1] [INFO] ... Using robot URDF from ...
[robot_state_publisher-1] [INFO] ... Publishing transforms in the background using service ...
```

---

### **Terminal 2: Khởi động LiDAR** (Khởi động thứ 2)

```bash
cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one rplidar.launch.py
```

**Output kỳ vọng**:
```
[rplidar_node-1] [INFO] RPLIDAR S2 is running at 8Hz
[rplidar_node-1] [INFO] Publishing scan data...
```

**Kiểm tra trong Terminal khác**:
```bash
ros2 topic echo /scan --once  # Xem 1 scan
# Phải thấy:
# ranges: [...]  # Danh sách khoảng cách
# angle_min: -3.14...
# angle_max: 3.14...
```

---

### **Terminal 3: SLAM Toolbox** (Khởi động thứ 3)

```bash
cd ~/ros2_ws
source install/setup.bash

ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

**Output kỳ vọng**:
```
[async_slam_toolbox_node-1] [INFO] ... Starting SLAM ...
[async_slam_toolbox_node-1] [INFO] ... Receiving scan data ...
[async_slam_toolbox_node-1] [INFO] ... Map with score: ...
```

**Kiểm tra**:
```bash
# Terminal khác
ros2 topic list | grep -E "(map|slam)"
# Phải thấy: /map, /map_metadata, /slam_toolbox/...
```

---

### **Terminal 4: RViz** (Để xem bản đồ)

```bash
cd ~/ros2_ws

# Option 1: Nếu có file config
rviz2 -d src/articubot_one/config/map.rviz

# Option 2: Nếu không có, tạo mới
rviz2
```

**Cấu hình RViz**:

1. **Left Panel → Add → Map**
   - Topic: `/map`
   - Color Scheme: `costmap`
   
2. **Add → LaserScan**
   - Topic: `/scan`
   - Color Transformer: `Intensity`

3. **Add → TF** (để xem frame)
   - Show Names ✓

4. **Global Options → Fixed Frame**: Chọn `map`

---

### **Terminal 5: Điều khiển Robot**

**Cách 1: Dùng Joystick**
```bash
ros2 launch articubot_one joystick.launch.py use_sim_time:=false

# Lúc này cầm joystick và di chuyển robot
```

**Cách 2: Dùng Command (test)**
```bash
# Đi thẳng
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"

# Quay vòng
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.3}}"

# Đi ngoảnh
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.2}}"
```

---

## **PHẦN 3: QUAN SÁT QUÁTRÌNH MAPPING**

### **Trong RViz, bạn sẽ thấy**:

1. **Map** (màu đen/xám): Bản đồ đang được tạo
2. **Scan** (màu đỏ/vàng): Dữ liệu LiDAR hiện tại
3. **Robot frame**: Vị trí robot hiện tại

### **Dấu hiệu Mapping đúng**:

✅ Map có những vùng đen (chướng ngại vật)
✅ Map có vùng xám (khu vực trống)
✅ Scan data khớp với các bức tường/vật thể
✅ Khi quay vòng → bản đồ được "khép lại"
✅ Khi đi qua chỗ cũ → bản đồ không "nhân đôi"

---

## **PHẦN 4: CÁC TÓM TẮT NGẮN**

### **Nếu bản đồ bị lỗi**:

**Dấu hiệu**: Map méo, nhân đôi, hoặc không khép vòng

**Giải pháp**:
```bash
# Dừng SLAM Toolbox (Ctrl+C ở Terminal 3)
# Di chuyển robot trở lại điểm bắt đầu
# Khởi động SLAM lại
# Mapping lại từ đầu
```

---

### **Nếu LiDAR không hoạt động**:

**Dấu hiệu**: Không thấy topic `/scan` hoặc ranges toàn 0

**Giải pháp**:
```bash
# 1. Kiểm tra kết nối vật lý (USB cable)
# 2. Khởi động lại rplidar.launch.py
# 3. Kiểm tra cổng USB:
lsusb  # Phải thấy lidar device
```

---

### **Nếu Transform (TF) bị lỗi**:

**Dấu hiệu**: RViz báo "Transform [map] not available"

**Giải pháp**:
```bash
# Xem cấu trúc frame
ros2 run tf2_tools view_frames
evince frames.pdf  # xem file PDF

# Nếu thấy map frame bị disconnect
# → Kiểm tra SLAM Toolbox có chạy không (Terminal 3)
```

---

## **PHẦN 5: LƯU BẢN ĐỒ**

### **Tự động lưu** (nếu cấu hình):

Kiểm tra file `mapper_params_online_async.yaml`:
```yaml
map_file_name: /home/pi/my_map  # Đây là nơi lưu
```

Bản đồ sẽ được lưu tự động (hoặc khi tắt SLAM).

### **Lưu thủ công** (Khuyến nghị):

```bash
# Sau khi mapping xong
cd ~/ros2_ws

# Lưu bản đồ
ros2 service call /slam_toolbox/save_map \
    slam_toolbox/srv/SaveMap \
    "{name: {data: '/home/pi/my_awesome_map'}}"

# Output sẽ hiện:
# saved_map: true
```

**Kiểm tra file được lưu**:
```bash
ls -la ~/my_awesome_map.*
# Phải thấy:
# my_awesome_map.pgm   (ảnh bản đồ)
# my_awesome_map.yaml  (metadata)
```

---

## **PHẦN 6: SỬ DỤNG BẢN ĐỒ ĐÃ LƯU**

### **Bước 1: Chỉnh sửa config SLAM**

```bash
nano src/articubot_one/config/mapper_params_online_async.yaml
```

Tìm và sửa:
```yaml
# OLD:
map_file_name: /home/pi/dev_ws/my_map_serial
mode: localization

# NEW:
map_file_name: /home/pi/my_awesome_map
mode: localization  # hoặc 'mapping' nếu muốn tiếp tục mapping
```

### **Bước 2: Khởi động lại SLAM**

```bash
# Dừng SLAM cũ (Ctrl+C)
# Khởi động lại
ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

---

## **PHẦN 7: KIỂM CHỨNG MAPPING THÀNH CÔNG**

Danh sách kiểm tra cuối cùng:

- [ ] LiDAR hoạt động → thấy `/scan` topic
- [ ] RSP hoạt động → thấy transform
- [ ] SLAM Toolbox chạy → thấy `/map` topic
- [ ] RViz hiển thị bản đồ
- [ ] Robot di chuyển → bản đồ cập nhật
- [ ] Khi đi qua chỗ cũ → bản đồ không nhân đôi
- [ ] Loop closing hoạt động (nếu vẽ vòng)
- [ ] Bản đồ được lưu thành công

✅ **Nếu tất cả đều OK → BẠN ĐÃ MAPPING THÀNH CÔNG!**

---

## **🎓 Giải thích những gì xảy ra:**

### **Bước 1-3**: Khởi động các thành phần cơ bản
- **RSP**: Nói cho ROS2 biết hình dáng robot
- **LiDAR**: Cung cấp dữ liệu quét
- **SLAM**: Xử lý dữ liệu + tạo bản đồ

### **Bước 4**: Hiển thị bản đồ
- **RViz**: Vẽ bản đồ để bạn nhìn thấy

### **Bước 5**: Cung cấp dữ liệu cho SLAM
- **cmd_vel**: Điều khiển robot
- Robot di chuyển → SLAM nhận dữ liệu mới
- SLAM so sánh dữ liệu → cập nhật bản đồ

### **Bước 6**: Lưu kết quả
- Bản đồ được lưu dưới dạng ảnh (PGM) + metadata (YAML)

---

## **💡 Lời khuyên từ kinh nghiệm**

1. **Di chuyển chậm** (0.1-0.2 m/s) → SLAM có thời gian xử lý
2. **Đi hình xoắn/S** → quét toàn bộ khu vực
3. **Quay vòng ít nhất 1-2 lần** → đảm bảo 360° coverage
4. **Nếu có vòng lặp** (đi trở lại chỗ cũ):
   - Di chuyển chậm hơn nữa
   - Quay vòng tại chỗ cũ → loop closing sẽ kích hoạt
5. **Lưu bản đồ khi hoàn thành** → không mất dữ liệu

---

**Chúc bạn mapping thành công! 🎉**
