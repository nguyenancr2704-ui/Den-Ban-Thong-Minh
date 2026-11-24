# 💡 Đèn Bàn Thông Minh (Smart Desk Lamp) — ESP32

Hệ thống đèn bàn học thông minh sử dụng vi điều khiển **ESP32**, tự động điều tiết độ sáng theo ánh sáng môi trường, tự động bật/tắt khi phát hiện người ngồi, đồng bộ thời gian thực qua Internet (NTP) và tích hợp **Web Server** điều khiển trực tiếp màn hình OLED cùng cấu hình Wi-Fi không cần nạp lại code.

## ✨ Tính năng nổi bật

- **Tự động bù sáng thông minh (Adaptive Brightness):** Cảm biến **BH1750** đo độ rọi môi trường liên tục (`0 – 500 lx`) và điều chỉnh độ rộng xung **PWM (5000 Hz)** xuất ra LED để duy trì ánh sáng dễ chịu cho mắt (`PWM = 255` khi phòng tối và giảm dần về `PWM = 10` khi môi trường đủ sáng).
- **Nhận diện hiện diện & chống tắt nhầm:** Cảm biến siêu âm **HC-SR04** phát hiện người ngồi trong phạm vi **60 cm**. Khi rời khỏi bàn, hệ thống giữ trạng thái chờ trong **5 giây** trước khi tắt hẳn đèn.
- **Điều khiển cảm ứng 1 chạm (2× TTP223):**
  - **Cảm ứng 1 (`GPIO 17`):** Chuyển tuần tự 3 chế độ hoạt động `AUTO` → `ON` → `OFF`.
  - **Cảm ứng 2 (`GPIO 16`):** Bật/tắt nhanh điểm phát Wi-Fi cấu hình (`SmartLamp_Setup`) bất kỳ lúc nào.
- **Tích hợp Web Server & Captive Portal (`PROGMEM`):**
  - Tự động phát Wi-Fi **`SmartLamp_Setup`** (`http://192.168.4.1`) khi chưa có mạng hoặc mất kết nối quá 15 giây.
  - Truy cập nhanh qua tên miền nội bộ **`http://denban.local`** (mDNS) khi đã kết nối Wi-Fi.
  - Giao diện điều khiển phong cách Ukiyo-e với 3 chủ đề màu (*Indigo*, *Wave*, *Morning*), hỗ trợ điều khiển chế độ đèn, quét Wi-Fi 2.4GHz và tùy chỉnh màn hình OLED theo thời gian thực.
- **Lưu cấu hình vĩnh viễn (NVS Flash):** Toàn bộ thông tin Wi-Fi và thiết lập hiển thị OLED được lưu vào bộ nhớ không bay hơi (`Preferences`), tự động khôi phục sau khi mất điện.

---

## 📋 Thông số kỹ thuật

| Hạng mục | Thông số chi tiết |
| :--- | :--- |
| **Nguồn cấp** | Adapter AC/DC 12V – 3A (36W); hạ áp xuống 5V qua mạch Buck cho ESP32 và cảm biến |
| **Vi điều khiển** | ESP32 (mức logic 3.3V, hỗ trợ Wi-Fi 2.4GHz) |
| **Cảm biến ánh sáng** | BH1750 — dải đo `1 – 65535 lx`, giao tiếp I²C |
| **Cảm biến hiện diện** | HC-SR04 — dải đo `2 – 400 cm`, ngưỡng kích hoạt mặc định `60 cm`, trễ tắt `5 giây` |
| **Màn hình hiển thị** | OLED 0.96 inch SSD1306 (`128×64` pixel), giao tiếp I²C (địa chỉ `0x3C`) |
| **Nút điều khiển** | 2 module cảm ứng điện dung TTP223 (mức kích hoạt `HIGH`) |
| **Điều khiển độ sáng** | Điều chế độ rộng xung (PWM) tần số `5000 Hz`, độ phân giải 8-bit (`0 – 255`) |
| **Độ rọi mục tiêu** | `300 – 500 lx` tại mặt bàn |

---

## 🔌 Sơ đồ đấu nối phần cứng (Pinout)

| Linh kiện | Chân linh kiện | Chân ESP32 | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Mạch kích LED (MOSFET)** | `PWM / SIG` | `GPIO 23` | Điều khiển dải LED 12V |
| **OLED 0.96" SSD1306** | `SDA` / `SCL` | `GPIO 21` / `GPIO 22` | Nguồn `3.3V`, địa chỉ `0x3C` |
| **Cảm biến BH1750** | `SDA` / `SCL` | `GPIO 21` / `GPIO 22` | Dùng chung bus I²C với OLED, nguồn `3.3V` |
| **Cảm biến HC-SR04** | `TRIG` / `ECHO` | `GPIO 18` / `GPIO 19` | Nguồn `5V` (nên chia áp chân Echo về `3.3V`) |
| **Cảm ứng TTP223 (Mode)** | `I/O` | `GPIO 17` | Nguồn `3.3V`, chuyển chế độ `AUTO/ON/OFF` |
| **Cảm ứng TTP223 (Wi-Fi)** | `I/O` | `GPIO 16` | Nguồn `3.3V`, bật/tắt Wi-Fi cấu hình |

---

## 🛠️ Cài đặt & Nạp chương trình

### 1. Thư viện yêu cầu (Arduino IDE)
Cài đặt các thư viện sau qua **Library Manager**:
- `BH1750` (by Christopher Laws)
- `Adafruit GFX Library`
- `Adafruit SSD1306`

*(Các thư viện `WiFi`, `WebServer`, `DNSServer`, `ESPmDNS`, `Preferences`, `Wire` đã tích hợp sẵn trong gói **esp32 by Espressif Systems**).*

### 2. Cấu hình Board & Nạp code
1. Mở file `.ino` trong Arduino IDE.
2. Chọn **Tools → Board → ESP32 Dev Module**, chọn đúng cổng **COM**.
3. Chọn **Upload Speed: `115200`** (hoặc `921600`) và nhấn **Upload**.
4. Nếu mạch không tự vào chế độ nạp, nhấn giữ nút **BOOT** khi thấy dòng `Connecting...` và thả ra khi bắt đầu `Writing...`.

---

## 📱 Hướng dẫn sử dụng

1. **Kết nối Wi-Fi lần đầu:**
   - Khi mới nạp code (hoặc khi chạm **cảm ứng TTP223 số 2 — `GPIO 16`**), màn hình OLED hiển thị `== CAI DAT WIFI ==`.
   - Dùng điện thoại kết nối vào mạng Wi-Fi **`SmartLamp_Setup`** (không mật khẩu).
   - Truy cập **`http://192.168.4.1`** trên trình duyệt, chọn mạng Wi-Fi 2.4GHz, nhập mật khẩu và nhấn **Lưu cấu hình Wi-Fi**.
2. **Tùy chỉnh giao diện OLED & điều khiển từ xa:**
   - Sau khi đèn kết nối thành công vào Wi-Fi nhà và đồng bộ giờ Việt Nam (GMT+7), kết nối điện thoại cùng mạng Wi-Fi đó và mở trình duyệt tại **`http://denban.local`** (hoặc địa chỉ IP của ESP32 in trên Serial Monitor).
   - Tại đây có thể bật/tắt từng thành phần trên OLED (ngày tháng, đồng hồ lớn, Lux, khoảng cách), lật ngược màn hình `180°` hoặc chuyển chế độ đèn trực tiếp.

---

## 📂 Cấu trúc thư mục gợi ý

```text
smart-desk-lamp-esp32/
├── firmware/
│   └── smart_desk_lamp.ino        # Code chính cho ESP32 (tích hợp Web Server PROGMEM)
├── web-preview/
│   └── den-ban-thong-minh.html    # Giao diện Web độc lập dùng để xem trước / mô phỏng
└── README.md                      # Tài liệu hướng dẫn dự án
```
