# 🔬 PHÂN TÍCH CODE MAPPING - CHI TIẾT TỪNG DÒNG

## **FILE 1: online_async_launch.py**

**Vị trí**: `launch/online_async_launch.py`

### **Giải thích từng phần:**

```python
import os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, LogInfo
from launch.conditions import UnlessCondition
from launch.substitutions import LaunchConfiguration, PythonExpression
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
from nav2_common.launch import HasNodeParams
```

**Các import này làm gì?**
- `os`: Để làm việc với đường dẫn tệp
- `launch, LaunchDescription`: Để tạo file launch
- `Node`: Để khởi động các ROS2 node
- `get_package_share_directory`: Để tìm package directory
- `HasNodeParams`: Để kiểm tra xem file cấu hình có đúng tham số không

---

```python
def generate_launch_description():
    use_sim_time = LaunchConfiguration('use_sim_time')
    params_file = LaunchConfiguration('params_file')
```

**Giải thích**:
- `generate_launch_description()`: Hàm chính tạo launch description
- `use_sim_time`: Biến kiểm tra có dùng thời gian mô phỏng không
  - `true` = Gazebo simulation
  - `false` = Robot thật
- `params_file`: Đường dẫn đến file cấu hình YAML

---

```python
    default_params_file = os.path.join(
        get_package_share_directory("articubot_one"),
        'config', 'mapper_params_online_async.yaml'
    )
```

**Giải thích**:
- Tìm thư mục package `articubot_one`
- Nối đường dẫn: `.../articubot_one/config/mapper_params_online_async.yaml`
- Đây là tệp cấu hình mặc định cho SLAM

---

```python
    declare_use_sim_time_argument = DeclareLaunchArgument(
        'use_sim_time',
        default_value='true',
        description='Use simulation/Gazebo clock'
    )
```

**Giải thích**:
- Khai báo một argument có thể truyền từ command line
- Tên: `use_sim_time`
- Giá trị mặc định: `'true'` (dùng Gazebo)
- Nếu chạy trên robot thật: `ros2 launch ... use_sim_time:=false`

---

```python
    declare_params_file_cmd = DeclareLaunchArgument(
        'params_file',
        default_value=default_params_file,
        description='Full path to the ROS2 parameters file...'
    )
```

**Giải thích**:
- Khai báo argument cho file cấu hình
- Mặc định dùng `mapper_params_online_async.yaml`
- Có thể ghi đè bằng: `ros2 launch ... params_file:=/path/to/custom.yaml`

---

```python
    has_node_params = HasNodeParams(
        source_file=params_file,
        node_name='slam_toolbox'
    )

    actual_params_file = PythonExpression([
        '"', params_file, '" if ', has_node_params,
        ' else "', default_params_file, '"'
    ])
```

**Giải thích**:
- Kiểm tra file cấu hình có chứa tham số `slam_toolbox` không
- Nếu có → dùng file đó
- Nếu không → dùng file mặc định
- Tương tự như: `actual_file = param_file if has_node_params else default_params_file`

---

```python
    start_async_slam_toolbox_node = Node(
        parameters=[
            actual_params_file,
            {'use_sim_time': use_sim_time}
        ],
        package='slam_toolbox',
        executable='async_slam_toolbox_node',
        name='slam_toolbox',
        output='screen'
    )
```

**Giải thích**:
- Tạo Node để khởi động SLAM Toolbox
- `package='slam_toolbox'`: Tên package chứa executable
- `executable='async_slam_toolbox_node'`: Tên chương trình cần chạy
  - `async` = không đồng bộ (xử lý nhanh hơn)
  - Có thể dùng `online_sync_slam_toolbox_node` để chậm hơn nhưng chính xác hơn
- `name='slam_toolbox'`: Tên của node này (dùng để debug)
- `output='screen'`: In output ra terminal (thay vì file log)
- `parameters`: Danh sách cấu hình:
  - File YAML cấu hình
  - `use_sim_time`: Thời gian mô phỏng hay thời gian thật?

---

```python
    ld = LaunchDescription()
    ld.add_action(declare_use_sim_time_argument)
    ld.add_action(declare_params_file_cmd)
    ld.add_action(log_param_change)
    ld.add_action(start_async_slam_toolbox_node)
    return ld
```

**Giải thích**:
- Tạo `LaunchDescription` rỗng
- Thêm các action (khai báo argument, logging, khởi động node)
- Return description này → ROS2 sẽ thực thi các action

---

## **FILE 2: mapper_params_online_async.yaml**

**Vị trí**: `config/mapper_params_online_async.yaml`

### **Các tham số quan trọng nhất:**

#### **1. FRAME CONFIGURATION** (Cấu hình hệ tọa độ)

```yaml
odom_frame: odom          # Frame từ odometry (bánh xe)
map_frame: map            # Frame bản đồ
base_frame: base_footprint # Frame tâm robot
```

**Giải thích**:
- `odom_frame`: Frame được tính từ chuyển động bánh xe
  - Drift (lệch) theo thời gian
- `map_frame`: Frame cố định toàn cục
  - Do SLAM tính toán
- `base_frame`: Nơi SLAM tính toán vị trí robot
  - Thường ở tâm hoặc chân robot

**Sơ đồ**:
```
map_frame (SLAM tính toán từ toàn cục)
    ↓
odom_frame (Từ bánh xe)
    ↓
base_footprint (Chân robot)
    ↓
base_link (Tâm robot)
```

---

#### **2. SENSOR CONFIGURATION** (Cấu hình cảm biến)

```yaml
scan_topic: /scan           # Topic LiDAR
mode: localization          # localization hoặc mapping
map_file_name: /home/dev/dev_ws/my_map_serial  # Nơi lưu bản đồ
```

**Giải thích**:
- `scan_topic: /scan`: SLAM lắng nghe topic này từ LiDAR
- `mode: localization`: 
  - `localization`: Định vị trên bản đồ cũ (không vẽ mới)
  - `mapping`: Vẽ bản đồ mới
- `map_file_name`: Nơi tìm bản đồ (khi dùng localization) hoặc lưu (khi dùng mapping)

---

#### **3. SCAN MATCHING** (So khớp dữ liệu)

```yaml
minimum_travel_distance: 0.5      # Đi 0.5m mới lấy scan mới
minimum_travel_heading: 0.5       # Quay 0.5 rad (28°) mới lấy scan mới
use_scan_matching: true           # Bật so khớp scan
use_scan_barycenter: true         # Dùng trọng tâm scan (chính xác hơn)
```

**Giải thích**:
- `minimum_travel_distance: 0.5`: 
  - Nếu robot di chuyển < 0.5m → không lấy scan mới
  - Tiết kiệm tài nguyên, tránh dữ liệu thừa
- `minimum_travel_heading: 0.5`:
  - Nếu robot quay < 0.5 rad → không lấy scan mới
  - Tương tự như trên
- `use_scan_matching: true`:
  - Bật: So sánh scan hiện tại với scan trước
  - Tìm ra robot di chuyển bao nhiêu
  - Cập nhật vị trí robot trong bản đồ

---

#### **4. LOOP CLOSURE** (Đóng vòng)

```yaml
do_loop_closing: true                    # Bật loop closing
loop_search_maximum_distance: 3.0         # Tìm vòng trong 3m
loop_match_minimum_chain_size: 10        # Cần ít nhất 10 scan để loop
loop_match_minimum_response_coarse: 0.35 # Độ tin cậy tối thiểu (coarse)
loop_match_minimum_response_fine: 0.45   # Độ tin cậy tối thiểu (fine)
```

**Giải thích**:
- **Loop Closing**: Khi robot quay lại chỗ nó đã đến trước đó
- `do_loop_closing: true`: Bật chức năng này
- `loop_search_maximum_distance: 3.0`:
  - SLAM tìm scan cũ trong bán kính 3m
  - Nếu tìm thấy → kiểm tra xem có khớp không
- `loop_match_minimum_chain_size: 10`:
  - Cần ít nhất 10 scan liên tiếp khớp → công nhận loop
  - Tránh false positive
- Confidence:
  - Nếu SLAM tìm thấy điểm khớp với độ tin cậy > 0.45 → xác nhận loop

**Ví dụ**:
```
Robot vẽ hình U:

Start ─→─┐
         │
         └─→ (đi qua lại chỗ cũ → loop closing kích hoạt!)
```

---

#### **5. RESOLUTION & RANGE** (Độ phân giải & Tầm)

```yaml
resolution: 0.05              # Mỗi ô = 5cm x 5cm
max_laser_range: 20.0         # LiDAR tầm xa 20m
```

**Giải thích**:
- `resolution: 0.05`:
  - Bản đồ chia thành ô 5cm × 5cm
  - Ô nhỏ hơn → chi tiết hơn, nhưng tốn bộ nhớ
  - Ô lớn hơn → kém chi tiết, nhưng nhanh hơn
  - Chọn 5cm là tốt cho robot này
- `max_laser_range: 20.0`:
  - Laser tầm xa 20m
  - Đừng lưu điểm > 20m → tiết kiệm bộ nhớ

---

#### **6. SOLVER** (Thuật toán giải quyết)

```yaml
solver_plugin: solver_plugins::CeresSolver
ceres_linear_solver: SPARSE_NORMAL_CHOLESKY
ceres_preconditioner: SCHUR_JACOBI
ceres_trust_strategy: LEVENBERG_MARQUARDT
ceres_loss_function: None
```

**Giải thích** (chỉ cần biết):
- `CeresSolver`: Thuật toán tối ưu bản đồ
- Các tham số khác: Cách Ceres tính toán (không cần thay đổi)
- Mục đích: Giảm lỗi, làm bản đồ chính xác hơn

---

## **FILE 3: Luồng dữ liệu Chi tiết**

### **Khi robot di chuyển:**

```
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 1: LiDAR quét                                     │
│  - Phát 2000+ tia laser xung quanh robot                │
│  - Laser phản hồi từ vật thể                            │
│  - Tính khoảng cách: distance = time × speed of light   │
│  - Gửi data tới topic /scan                             │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 2: SLAM Toolbox nhận dữ liệu                      │
│  - Lắng nghe topic /scan                                │
│  - Kiểm tra: robot di chuyển đủ (distance, heading)?    │
│  - Nếu không → bỏ qua scan này                          │
│  - Nếu có → tiếp tục                                    │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 3: Scan Matching (So khớp scan)                   │
│  - Lấy scan hiện tại vs scan trước                      │
│  - Tìm điểm giống nhau (góc, khoảng cách)               │
│  - Tính robot di chuyển bao nhiêu (dx, dy, dθ)          │
│  - Cập nhật vị trí robot trong bản đồ                   │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 4: Thêm dữ liệu vào bản đồ                        │
│  - Chuyển đổi laser scan → 2D grid (map)                │
│  - Các ô có chướng ngại vật → màu đen                   │
│  - Các ô trống → màu xám/trắng                          │
│  - Cộng cộng dữ liệu vào bản đồ cũ                      │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 5: Loop Detection (Tìm vòng)                      │
│  - Kiểm tra: Có scan cũ (> 3m trước) khớp không?       │
│  - Nếu có → kích hoạt loop closing                      │
│  - Sửa lỗi tích lũy từ odometry                         │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 6: Tối ưu bản đồ (Solver)                         │
│  - Sử dụng CeresSolver để giảm lỗi                      │
│  - Điều chỉnh các scan để khớp nhau                     │
│  - Bản đồ chính xác hơn                                 │
└─────────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────┐
│  BƯỚC 7: Publish kết quả                                │
│  - Publish map frame → /map, /map_metadata              │
│  - Publish transform: map → base_footprint              │
│  - RViz nhận dữ liệu và vẽ bản đồ                       │
└─────────────────────────────────────────────────────────┘
```

---

## **FILE 4: Các Topic ROS2 quan trọng**

### **Subscriber (SLAM lắng nghe)**:

```
/scan
├─ sensor_msgs/msg/LaserScan
├─ Từ: LiDAR (rplidar_node)
├─ Tần số: ~8 Hz (RPLiDAR S2)
└─ Chứa: angle_min, angle_max, ranges[], intensities[]
```

**Ví dụ dữ liệu**:
```yaml
ranges: [3.5, 3.4, 3.3, ..., 2.1, 2.0]  # 2000 giá trị khoảng cách
angle_increment: 0.00157 rad  # Khoảng giữa các tia
angle_min: -3.14 rad
angle_max: 3.14 rad
```

---

### **Publisher (SLAM phát hành)**:

```
/map
├─ nav_msgs/msg/OccupancyGrid
├─ Từ: SLAM Toolbox
├─ Tần số: ~1 Hz
└─ Chứa: Grid 2D với giá trị occupancy (-1, 0-100)
      -1 = unknown, 0 = free, 100 = occupied

/map_metadata
├─ nav_msgs/msg/MapMetaData
├─ Chứa: resolution, width, height, origin

/tf
├─ geometry_msgs/msg/TransformStamped
├─ Chứa: map → odom transform
└─ Cho ROS biết vị trí robot trong bản đồ
```

---

## **FILE 5: Gỡ lỗi - Xem dữ liệu**

### **Kiểm tra SLAM chạy**:

```bash
# Liệt kê tất cả topics
ros2 topic list | grep -E "(map|slam)"

# Output mong đợi:
# /map
# /map_metadata
# /slam_toolbox/dynamic_map
# /slam_toolbox/scan_matched_points
```

### **Xem tốc độ map publish**:

```bash
ros2 topic hz /map

# Output: ~1 Hz (1 bản đồ/giây)
```

### **Xem chi tiết map**:

```bash
ros2 topic echo /map_metadata

# Output:
# map_load_time: ...
# resolution: 0.05
# width: 2000
# height: 2000
# origin:
#   position: {x: -50.0, y: -50.0, z: 0.0}
```

### **Kiểm tra transform**:

```bash
# Xem tất cả transform
ros2 run tf2_tools view_frames

# Xem transform cụ thể
ros2 run tf2_ros tf2_monitor map base_footprint

# Output:
# "map" TO "base_footprint"
# Translation: [-1.234, 2.345, 0.000]
# Rotation: ...
```

---

## **Tóm tắt**

| Thành phần | Chức năng | File |
|-----------|----------|------|
| **Launch file** | Khởi động SLAM node | `online_async_launch.py` |
| **Config YAML** | Cấu hình tham số SLAM | `mapper_params_online_async.yaml` |
| **Input: /scan** | Dữ liệu LiDAR | Từ rplidar_node |
| **Process** | So khớp, loop closing, tối ưu | async_slam_toolbox_node |
| **Output: /map** | Bản đồ 2D | Để RViz hiển thị |
| **Output: /tf** | Transform map→robot | Cho ROS biết vị trí |

---

**Bây giờ bạn đã hiểu mapping code hoạt động như thế nào! 🚀**
