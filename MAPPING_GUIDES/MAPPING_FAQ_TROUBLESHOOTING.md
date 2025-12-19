# ❓ MAPPING - CÂU HỎI THƯỜNG GẶP & TROUBLESHOOTING

## **Phần 1: Các vấn đề thường gặp**

---

### **❌ VẤN ĐỀ 1: Không thấy `/scan` topic**

**Biểu hiệu**:
```bash
$ ros2 topic list | grep scan
# Không có gì hiện ra hoặc không thấy /scan
```

**Nguyên nhân**:
- LiDAR không kết nối
- Driver LiDAR không chạy
- Cổng USB sai
- Permission bị từ chối

**Giải pháp**:

**Step 1: Kiểm tra kết nối vật lý**
```bash
# SSH vào Pi
lsusb
# Tìm "Silicon Labs" hoặc "Prolific" → là LiDAR
# Nếu không thấy → kết nối USB cable

# Kiểm tra cổng
ls -la /dev/tty*
# LiDAR thường ở /dev/ttyUSB0 hoặc /dev/ttyACM0
```

**Step 2: Chạy LiDAR launch**
```bash
ros2 launch articubot_one rplidar.launch.py

# Xem output:
# [rplidar_node-1] [INFO] RPLIDAR S2 started
# [rplidar_node-1] [INFO] Publishing scan data...
```

**Step 3: Kiểm tra topic**
```bash
# Terminal khác
ros2 topic list | grep scan
# Phải thấy: /scan

# Xem dữ liệu
ros2 topic echo /scan --once | head -20
```

**Nếu vẫn lỗi**:
```bash
# Kiểm tra chi tiết lỗi
ros2 launch articubot_one rplidar.launch.py --log-level DEBUG

# Hoặc kiểm tra quyền cổng USB
sudo usermod -a -G dialout $USER
# Đăng xuất và đăng nhập lại
```

---

### **❌ VẤN ĐỀ 2: SLAM Toolbox không khởi động**

**Biểu hiệu**:
```bash
$ ros2 launch articubot_one online_async_launch.py use_sim_time:=false
[ERROR] ... slam_toolbox not found
```

**Nguyên nhân**:
- Package slam_toolbox không cài đặt
- Workspace không được source

**Giải pháp**:

**Step 1: Cài đặt slam_toolbox**
```bash
sudo apt update
sudo apt install ros-humble-slam-toolbox
# Hoặc cho version khác:
sudo apt install ros-<distro>-slam-toolbox
```

**Step 2: Source workspace**
```bash
cd ~/ros2_ws
source install/setup.bash

# Hoặc thêm vào ~/.bashrc
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

**Step 3: Kiểm tra**
```bash
ros2 pkg find slam_toolbox
# Phải show đường dẫn
```

---

### **❌ VẤN ĐỀ 3: Bản đồ không được cập nhật**

**Biểu hiệu**:
- RViz thấy /map topic nhưng bản đồ trống hoặc không thay đổi khi robot di chuyển

**Nguyên nhân**:
- Robot di chuyển quá nhanh
- LiDAR dữ liệu bị chắn
- Transform frame sai

**Giải pháp**:

**Step 1: Kiểm tra robot di chuyển**
```bash
# Terminal 1: Monitor /scan
ros2 topic hz /scan
# Phải thấy ~8 Hz

ros2 topic echo /scan --once | grep ranges
# Phải thấy giá trị không phải 0 hoặc inf
```

**Step 2: Di chuyển robot**
```bash
# Di chuyển CHẬM (0.1 m/s)
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
    "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"
```

**Step 3: Kiểm tra map update**
```bash
ros2 topic hz /map
# Phải thấy > 0.5 Hz (ít nhất)

ros2 topic echo /map | head -50
# Phải thấy data.data có giá trị khác 0
```

**Step 4: Kiểm tra frame**
```bash
ros2 run tf2_tools view_frames
evince frames.pdf  # Xem cấu trúc frame
```

---

### **❌ VẤN ĐỀ 4: RViz báo "Transform [map] not available"**

**Biểu hiệu**:
```
[map] - Waiting for transform from [base_footprint] to [map]
```

**Nguyên nhân**:
- SLAM Toolbox chưa khởi động xong
- /tf topic không được publish
- Frame name sai

**Giải pháp**:

**Step 1: Kiểm tra SLAM chạy**
```bash
ros2 node list | grep slam
# Phải thấy: /slam_toolbox

ps aux | grep async_slam
# Phải thấy: async_slam_toolbox_node
```

**Step 2: Kiểm tra transform**
```bash
ros2 topic list | grep /tf
# Phải thấy: /tf, /tf_static

ros2 topic echo /tf --once
# Phải thấy transforms từ map frame
```

**Step 3: Kiểm tra RViz setting**
```
RViz → Global Options → Fixed Frame = "map"
# (Không phải "base_link" hoặc "odom")
```

**Step 4: Xem frame tree**
```bash
ros2 run tf2_tools view_frames
# Nếu map frame biệt lập → SLAM lỗi
```

---

### **❌ VẤN ĐỀ 5: Bản đồ bị lỗi, méo**

**Biểu hiệu**:
- Bản đồ có hình dạng kỳ lạ, méo mó
- Các bức tường không thẳng
- Bản đồ có "bóng ma" (nhân đôi)

**Nguyên nhân**:
- Robot di chuyển quá nhanh
- LiDAR bị rung lắc
- Cảm biến IMU bị lỗi
- Loop closing sai

**Giải pháp**:

**Tốc độ robot**:
```bash
# Giảm tốc độ xuống 0.1 m/s
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
    "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"
```

**Tăng thời gian giữa scan**:
```yaml
# Sửa mapper_params_online_async.yaml
minimum_travel_distance: 1.0  # Tăng từ 0.5 lên 1.0
minimum_travel_heading: 1.0   # Tăng từ 0.5 lên 1.0
```

**Tắt loop closing**:
```yaml
do_loop_closing: false  # Tạm tắt
# Khởi động lại SLAM
```

**Bắt đầu lại mapping**:
```bash
# Dừng SLAM (Ctrl+C)
# Di chuyển robot trở lại vị trí ban đầu
# Khởi động SLAM lại
ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

---

### **❌ VẤN ĐỀ 6: Bản đồ không đóng vòng (loop closing không hoạt động)**

**Biểu hiệu**:
- Robot đi về chỗ cũ nhưng bản đồ "nhân đôi"
- Loop không tự khớp lại

**Nguyên nhân**:
- Loop search distance quá nhỏ
- Robot di chuyển quá nhanh (không đủ scan để match)
- Môi trường lặp lại (VD: dãy kệ giống nhau)

**Giải pháp**:

**Tăng loop search distance**:
```yaml
mapper_params_online_async.yaml:
loop_search_maximum_distance: 5.0  # Tăng từ 3.0 lên 5.0
```

**Tăng minimum chain size** (cần nhiều scan hơn):
```yaml
loop_match_minimum_chain_size: 20  # Tăng từ 10 lên 20
```

**Giảm tin cậy** (loop dễ kích hoạt hơn):
```yaml
loop_match_minimum_response_coarse: 0.25  # Giảm từ 0.35
loop_match_minimum_response_fine: 0.35    # Giảm từ 0.45
```

**Cách test loop closing**:
```bash
# Vẽ hình U hoặc vòng tròn
# Di chuyển chậm (0.1 m/s)
# Quay vòng tại chỗ cũ
# Theo dõi RViz → bản đồ sẽ "nhét lại" khi loop khớp
```

---

## **Phần 2: Các câu hỏi thường gặp**

---

### **❓ CÂU HỎI 1: Chế độ "localization" vs "mapping" là gì?**

**Trả lời**:

**Mapping Mode** (Vẽ bản đồ):
```yaml
mode: mapping
```
- Robot vẽ bản đồ mới từ đầu
- SLAM Toolbox không load bản đồ cũ
- Tất cả dữ liệu mới được thêm vào bản đồ
- Lúc đầu bản đồ trống, rồi dần được lấp đầy

**Localization Mode** (Định vị):
```yaml
mode: localization
```
- Robot load bản đồ cũ (từ `map_file_name`)
- SLAM chỉ cập nhật vị trí robot trên bản đồ
- Không vẽ bản đồ mới
- Dùng để điều hướng trên bản đồ đã biết

**Lựa chọn**:
- Lần đầu: Dùng `mapping`
- Sau khi lưu bản đồ: Đổi sang `localization`

---

### **❓ CÂU HỎI 2: Tại sao phải lưu bản đồ?**

**Trả lời**:
- Mapping tốn tài nguyên (CPU, RAM)
- Nếu tắt SLAM rồi bật lại → bản đồ cũ bị mất
- Bản đồ được lưu có thể dùng cho:
  - Định vị (`localization`)
  - Navigation (điều hướng tự động)
  - Các lần mapping tiếp theo

**Ví dụ**:
```
Lần 1: Mapping khu vực → Lưu bản đồ
Lần 2: Load bản đồ cũ → Tiếp tục mapping (nếu cần)
       hoặc Định vị + Điều hướng
```

---

### **❓ CÂU HỎI 3: Resolution 0.05m là gì? Giảm/tăng nó có sao không?**

**Trả lời**:
- `resolution: 0.05` = Mỗi ô trên bản đồ = 5cm × 5cm

| Resolution | Kích thước ô | Ưu điểm | Nhược điểm |
|-----------|----------|--------|-----------|
| 0.02m | 2cm × 2cm | Rất chi tiết | Tốn bộ nhớ, chậm |
| 0.05m | 5cm × 5cm | **Cân bằng tốt** | - |
| 0.1m | 10cm × 10cm | Nhanh, tiết kiệm | Kém chi tiết |
| 0.2m | 20cm × 20cm | Rất nhanh | Rất kém chi tiết |

**Khuyến nghị**: Giữ `0.05` cho đa số robot
- Nếu bộ nhớ Pi hạn chế → thử `0.1`
- Nếu cần chi tiết cao → thử `0.02`

---

### **❓ CÂU HỎI 4: Odometry drift là gì?**

**Trả lời**:
- **Odometry**: Ước tính vị trí từ chuyển động bánh xe
- **Drift**: Lỗi tích lũy theo thời gian

**Ví dụ**:
```
Robot quay vòng 10 lần:
- Odometry: Vị trí sai 1m (drift)
- SLAM: Vị trí chính xác (sửa drift bằng camera/laser)
```

**SLAM giải quyết drift** bằng:
1. Scan matching: So sánh laser data
2. Loop closing: Khi quay lại chỗ cũ → fix sai số

---

### **❓ CÂU HỎI 5: Tại sao SLAM cần quay vòng?**

**Trả lời**:
- LiDAR chỉ quét 360° xung quanh robot (nằm ngang)
- Nếu robot chỉ đi thẳng:
  - Dữ liệu trên-dưới robot = trống
  - Bản đồ không hoàn chỉnh
- Quay vòng → LiDAR quét từ mọi góc
- Kết quả: Bản đồ đầy đủ, chính xác

**Cách mapping tốt**:
```
Hình vẽ chiều robot di chuyển:

1. Quay vòng ở điểm bắt đầu (360°)
2. Đi hình xoắn qua các khu vực
3. Quay vòng ở các điểm quan trọng
4. Quay lại chỗ cũ để loop closing
```

---

### **❓ CÂU HỎI 6: SLAM Toolbox async vs sync có khác gì?**

**Trả lời**:

| Async (Bất đồng bộ) | Sync (Đồng bộ) |
|---------|---------|
| `async_slam_toolbox_node` | `online_sync_slam_toolbox_node` |
| **Nhanh**: Không chờ optimize hoàn thành | **Chậm**: Optimize xong mới xử lý scan mới |
| **Thích hợp**: Real-time mapping | **Thích hợp**: Chính xác tuyệt đối |
| Có thể bỏ frame nếu CPU bị bottleneck | Không bỏ frame, đảm bảo xử lý tất cả |
| Kém chính xác hơn (một chút) | Chính xác hơn (nhưng chậm) |

**Khuyến nghị**: Dùng **Async** cho robot Pi (tài nguyên hạn chế)

---

### **❓ CÂU HỎI 7: Cần calibrate LiDAR không?**

**Trả lời**:
- RPLiDAR thường **không cần calibrate** (nó calibrate tự động)
- Nhưng cần:
  1. **Lắp thẳng**: LiDAR phải nằm ngang, không lệch
  2. **Sạch**: Vệ sinh kính LiDAR
  3. **Thử**: Quay vòng xem scan có cân bằng không

**Cách kiểm tra**:
```bash
# Xem laser scan trong RViz
# Nếu scan bị nghiêng → LiDAR lệch
# → Điều chỉnh độ cao/góc lắp LiDAR
```

---

### **❓ CÂU HỎI 8: Bản đồ .pgm là gì? Làm sao mở?**

**Trả lời**:
- `.pgm` = Portable Gray Map (ảnh grayscale)
- Bản đồ lưu ở dạng hình ảnh
- Màu sắc:
  - **Đen** (0) = Có chướng ngại vật
  - **Xám** (128) = Chưa biết
  - **Trắng** (255) = Trống

**Cách mở**:
```bash
# Xem bằng image viewer
eog ~/my_awesome_map.pgm  # GNOME
feh ~/my_awesome_map.pgm  # hoặc
feh ~/my_awesome_map.pgm

# Hoặc mở bằng Python
python3
>>> from PIL import Image
>>> img = Image.open("my_awesome_map.pgm")
>>> img.show()
```

**File .yaml chứa metadata**:
```yaml
# my_awesome_map.yaml
image: my_awesome_map.pgm
resolution: 0.05
origin: [-50.0, -50.0, 0.0]
occupied_thresh: 0.65
free_thresh: 0.196
```

---

### **❓ CÂU HỎI 9: Làm sao lưu bản đồ thủ công?**

**Trả lời**:

**Cách 1: Service call**
```bash
ros2 service call /slam_toolbox/save_map \
    slam_toolbox/srv/SaveMap \
    "{name: {data: '/home/pi/my_map_v2'}}"

# Output:
# result:
#   save_map: true
```

**Cách 2: Gọi từ code Python**
```python
import rclpy
from slam_toolbox.srv import SaveMap

node = rclpy.create_node('save_map_client')
client = node.create_client(SaveMap, '/slam_toolbox/save_map')

request = SaveMap.Request()
request.name.data = '/home/pi/my_map_v3'

future = client.call_async(request)
```

**Cách 3: Tự động lưu**
```yaml
# mapper_params_online_async.yaml
# SLAM sẽ tự động lưu vào đây khi tắt
map_file_name: /home/pi/auto_saved_map
```

---

### **❓ CÂU HỎI 10: RViz bị lag, bản đồ update chậm sao?**

**Trả lời**:

**Nguyên nhân**:
1. Bản đồ quá lớn
2. LiDAR scan tần số cao (tốn CPU)
3. RViz render quá nhiều layer

**Giải pháp**:

**Giảm kích thước bản đồ**:
```yaml
max_laser_range: 15.0  # Giảm từ 20.0
```

**Tăng khoảng cách giữa scan**:
```yaml
minimum_travel_distance: 1.0  # Tăng từ 0.5
minimum_travel_heading: 1.0
```

**Tắt layer không cần** trong RViz:
- Tắt LaserScan (thay bằng Map)
- Tắt TF (chỉ bật khi debug)

**Giảm quality map render**:
```
RViz → Map → Scheme: "costmap" thay vì "raw"
```

---

## **Phần 3: Checklist Mapping**

Trước khi bắt đầu mapping, kiểm tra:

- [ ] LiDAR kết nối vật lý
- [ ] SSH hoặc kết nối console với Pi
- [ ] ROS2 environment sourced (`source ~/ros2_ws/install/setup.bash`)
- [ ] slam_toolbox cài đặt (`ros2 pkg find slam_toolbox`)
- [ ] RPLiDAR launch file chạy được
- [ ] `/scan` topic xuất hiện
- [ ] RSP launch file chạy được
- [ ] Frame tree hoàn chỉnh (`ros2 run tf2_tools view_frames`)
- [ ] RViz mở được
- [ ] RViz có `/map` layer
- [ ] RViz có `/scan` layer
- [ ] Global frame set = "map"
- [ ] Robot không bị lỗi controller
- [ ] Vùng mapping sạch sẽ (không đồ vật di động)

✅ Tất cả OK? **Bắt đầu mapping!**

---

**Chúc bạn mapping thành công! 🎉 Nếu có vấn đề nào khác, hãy kiểm tra các bước trên!**
