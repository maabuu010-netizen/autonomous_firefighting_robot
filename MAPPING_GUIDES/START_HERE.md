# 🚀 MAPPING GUIDES - BẮT ĐẦU TỪ ĐÂY

Chào mừng! Bạn đang ở đúng chỗ để học Mapping trên robot Articubot One.

---

## **📂 Cấu trúc thư mục**

```
MAPPING_GUIDES/
├── START_HERE.md                          ← Bạn đang đọc file này!
├── INDEX_HUONG_DAN.md                     ← 📋 Danh sách & chỉ dẫn
├── README_MAPPING.md                      ← 📖 Giới thiệu tất cả
├── HUONG_DAN_MAPPING.md                   ← 📚 Khái niệm chi tiết
├── MAPPING_THUC_HANH.md                   ← ⭐⭐⭐ THỰC HÀNH CHÍNH
├── PHAN_TICH_CODE_MAPPING.md              ← 🔬 Phân tích code
├── MAPPING_FAQ_TROUBLESHOOTING.md         ← ❓ FAQ & Lỗi
├── MAPPING_DIAGRAMS.md                    ← 📊 Sơ đồ & hình vẽ
└── QUICK_REFERENCE.md                     ← ⚡ Copy-paste nhanh
```

---

## **🎯 Bạn là loại nào?**

### **🔰 "Tôi là newbie, chưa biết gì"**
```
➡️  Bước 1: Đọc README_MAPPING.md (5 phút)
➡️  Bước 2: Đọc HUONG_DAN_MAPPING.md (20 phút)
➡️  Bước 3: Đọc MAPPING_THUC_HANH.md (30 phút)
➡️  Bước 4: Theo hướng dẫn và thực hành
➡️  Bước 5: Nếu lỗi → Tra MAPPING_FAQ_TROUBLESHOOTING.md
```

### **⚡ "Tôi vội, chỉ cần chạy nhanh"**
```
➡️  Mở: QUICK_REFERENCE.md
➡️  Copy-paste 5 lệnh Terminal
➡️  Chạy!
➡️  Nếu lỗi → Tra FAQ
```

### **🔬 "Tôi muốn hiểu sâu code"**
```
➡️  Đọc: PHAN_TICH_CODE_MAPPING.md (45 phút)
➡️  Sửa: mapper_params_online_async.yaml
➡️  Test: Mapping với cấu hình mới
```

---

## **⏱️ Nhanh chóng (5 phút)**

### **Khởi động Mapping**

Mở **5 Terminal riêng** và chạy lần lượt:

**Terminal 1: Robot State Publisher**
```bash
cd ~/ros2_ws && source install/setup.bash
ros2 launch articubot_one rsp.launch.py use_sim_time:=false use_ros2_control:=false
```

**Terminal 2: LiDAR**
```bash
cd ~/ros2_ws && source install/setup.bash
ros2 launch articubot_one rplidar.launch.py
```

**Terminal 3: SLAM Toolbox**
```bash
cd ~/ros2_ws && source install/setup.bash
ros2 launch articubot_one online_async_launch.py use_sim_time:=false
```

**Terminal 4: RViz**
```bash
cd ~/ros2_ws
rviz2 -d src/articubot_one/config/map.rviz
```

**Terminal 5: Điều khiển (Joystick hoặc Command)**
```bash
# Joystick
ros2 launch articubot_one joystick.launch.py use_sim_time:=false

# Hoặc dùng lệnh:
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0}}"
```

✅ Di chuyển robot từ từ → RViz sẽ hiển thị bản đồ!

---

## **📚 Chi tiết từng File**

| File | Nội dung | Khi dùng |
|------|---------|---------|
| **INDEX_HUONG_DAN.md** | Danh sách & lộ trình | Lần đầu, muốn overview |
| **README_MAPPING.md** | Giới thiệu & hướng dẫn | Muốn biết nên đọc file nào |
| **HUONG_DAN_MAPPING.md** | Khái niệm Mapping | Muốn hiểu từng thành phần |
| **MAPPING_THUC_HANH.md** | 🌟 Thực hành chi tiết | **CHÍNH YẾU - Làm theo file này** |
| **PHAN_TICH_CODE_MAPPING.md** | Phân tích code & tham số | Muốn tối ưu hoặc học sâu |
| **MAPPING_FAQ_TROUBLESHOOTING.md** | ❓ 10 FAQ + 6 vấn đề | **GẶP LỖI - Tra cứu nhanh** |
| **MAPPING_DIAGRAMS.md** | Sơ đồ & hình vẽ ASCII | Muốn hình dung trực quan |
| **QUICK_REFERENCE.md** | Commands & cheat sheet | Đã biết, cần copy-paste |

---

## **🎓 3 Cách tiếp cận**

### **Cách 1: Học kỹ (Khuyến nghị cho lần đầu)**
⏱️ **Thời gian**: 1-2 giờ
```
1. README_MAPPING.md (5 phút)
2. HUONG_DAN_MAPPING.md (20 phút)
3. MAPPING_DIAGRAMS.md (10 phút)
4. MAPPING_THUC_HANH.md (30 phút - theo từng bước)
5. Thực hành mapping (30 phút)
6. Nếu lỗi → Tra MAPPING_FAQ_TROUBLESHOOTING.md
```

### **Cách 2: Copy-paste nhanh**
⏱️ **Thời gian**: 10 phút
```
1. Mở QUICK_REFERENCE.md
2. Copy 5 Terminal commands
3. Chạy
4. Mapping!
```

### **Cách 3: Deep dive (Cho người muốn expert)**
⏱️ **Thời gian**: 2-3 giờ
```
1. PHAN_TICH_CODE_MAPPING.md (45 phút)
2. Sửa mapper_params_online_async.yaml
3. Testing & tối ưu (1 giờ)
4. Advanced mapping techniques
```

---

## **✅ Checklist trước khi bắt đầu**

- [ ] Robot kết nối, Raspberry Pi bật
- [ ] LiDAR kết nối vào USB
- [ ] Có thể SSH vào Raspberry Pi
- [ ] ROS2 workspace đã compile
- [ ] SLAM Toolbox đã cài (`ros2 pkg find slam_toolbox`)
- [ ] Mở sẵn 5 Terminal
- [ ] Đọc xong README_MAPPING.md
- [ ] **Sẵn sàng!**

---

## **🚀 Quick Start (3 bước)**

### **Bước 1: Đọc (10 phút)**
Đọc qua **README_MAPPING.md** để hiểu sơ bộ

### **Bước 2: Làm (45 phút)**
Làm theo **MAPPING_THUC_HANH.md** từng bước

### **Bước 3: Debug (Nếu cần)**
Tra cứu **MAPPING_FAQ_TROUBLESHOOTING.md**

✅ **Xong!**

---

## **💡 Mẹo**

- 📖 **Không hiểu?** → Đọc HUONG_DAN_MAPPING.md
- 📊 **Muốn hình dung?** → Xem MAPPING_DIAGRAMS.md
- ❌ **Gặp lỗi?** → Tra FAQ
- ⚡ **Cần lệnh nhanh?** → Dùng QUICK_REFERENCE.md
- 🔬 **Muốn sửa cấu hình?** → Đọc PHAN_TICH_CODE_MAPPING.md

---

## **📞 Nếu gặp vấn đề**

**90% vấn đề được giải quyết ở đây:**
1. Tra **MAPPING_FAQ_TROUBLESHOOTING.md**
2. Chạy lệnh debug từ **QUICK_REFERENCE.md**
3. Đọc lại **MAPPING_THUC_HANH.md**

---

## **🎯 Mục tiêu sau khi học xong**

✅ Hiểu Mapping là gì
✅ Biết các thành phần hoạt động như thế nào
✅ Có thể chạy Mapping trên robot thực
✅ Lưu bản đồ thành công
✅ Tối ưu cấu hình
✅ Gỡ lỗi khi gặp vấn đề

---

## **🔄 Sau Mapping là gì?**

Khi bạn đã mapping xong:
1. **Lưu bản đồ** → Dùng cho lần tới
2. **Chuyển Localization** → Robot định vị trong bản đồ
3. **Chạy Navigation** → Robot tự động đi đến goal

---

## **📌 Bạn nên bắt đầu bằng file nào?**

### **Nếu bạn:**
- ✅ Lần đầu → **README_MAPPING.md**
- ✅ Muốn chạy ngay → **QUICK_REFERENCE.md**
- ✅ Muốn hiểu khái niệm → **HUONG_DAN_MAPPING.md**
- ✅ Muốn thực hành → **MAPPING_THUC_HANH.md** ⭐
- ✅ Gặp lỗi → **MAPPING_FAQ_TROUBLESHOOTING.md**
- ✅ Muốn sâu hơn → **PHAN_TICH_CODE_MAPPING.md**

---

## **Sẵn sàng chưa? 🚀**

**Chọn một file ở trên và bắt đầu!**

Nếu bạn chưa biết chọn gì → **Đọc `README_MAPPING.md`**

Nếu bạn muốn chạy ngay → **Xem `QUICK_REFERENCE.md`**

Nếu bạn muốn học kỹ → **Đọc `MAPPING_THUC_HANH.md`**

---

**Chúc bạn mapping thành công! 🎉**
