# Hệ thống Nhà thông minh & Vườn Sinh thái Thích ứng (IoT Smart Home & Eco-Garden)

Dự án môn học IoT triển khai mô hình **IoT Gateway** với kiến trúc **1 Node phần cứng tinh gọn**, kết hợp mạch **ESP32-S3 (YoloUNO)** chạy firmware **Tasmota**, **PC Gateway (Python + AI/LLM)** và nền tảng đám mây **CoreIOT**.

---

## 1. Thành viên & Phân công Nhiệm vụ

| Thành viên | Vai trò chính | Mã tiêu chí phụ trách | Thư mục làm việc |
| :--- | :--- | :--- | :--- |
| **Hy** | Phần cứng (YoloUNO S3, Tasmota, OTA) & GitHub | `T01`, `T07`, `T14` | `hy-hardware/` |
| **Huy** | PC Gateway (Python MQTT), AI, Trợ lý ảo LLM & Báo cáo | `T08`, `T09`, `N01` | `huy-gateway/`, `report/` |
| **Hưng** | Đám mây CoreIOT (Dashboard, RuleChain, Cảnh báo) & Video Demo | `T11`, `T12`, `T13` | `hung-coreiot/` |

---

## 2. Kiến trúc Hệ thống & Phần cứng

Hệ thống sử dụng **01 mạch ESP32-S3 (YoloUNO)** gắn trên **Mạch mở rộng OhStem** (chạy firmware Tasmota bản ESP32-S3), kết nối 100% bằng giắc cắm từ bộ **OhStem AIoT Kit**:

* **Cảm biến I2C DHT20 (`T01`):** Đo nhiệt độ và độ ẩm không khí.
* **USB đóng ngắt 2 kênh (hoặc Relay rời) (`T07`):** Đóng/ngắt trực tiếp **Quạt mini** (Kênh 1 - làm mát phòng) và **Máy bơm mini USB** (Kênh 2 - tưới cây).
* **Cảm biến độ ẩm đất:** Theo dõi độ ẩm đất phục vụ tưới cây tự động.
* **Cảm biến PIR & 4 LED RGB *(tùy chọn)*:** Hỗ trợ phát hiện chuyển động và hiển thị trạng thái cảnh báo.

**Luồng dữ liệu:**
`YoloUNO S3 (Tasmota)` $\xrightarrow{\text{MQTT (LAN)}}$ `PC Gateway (Python + AI/LLM)` $\xrightarrow{\text{MQTT / HTTP}}$ `CoreIOT (Dashboard, RuleChain, Alert)`

---

## 3. Cấu trúc Thư mục

```text
.
├── hy-hardware/              # Phần cứng & Tasmota (Hy)
│   ├── tasmota_config.md     # Cấu hình GPIO (DHT20, Relay/USB 2 kênh, Độ ẩm đất) & MQTT
│   └── ota_firmware/         # File firmware .bin / hướng dẫn demo cập nhật OTA (T14)
├── huy-gateway/              # PC Gateway & AI/LLM (Huy)
│   ├── mqtt_bridge.py        # Cầu nối MQTT 2 chiều Tasmota <-> CoreIOT + Mock Data (T08)
│   ├── ai_detector.py        # Thuật toán AI phát hiện cháy sớm / bất thường (T09)
│   ├── llm_assistant.py      # Trợ lý ảo LLM điều khiển bằng tiếng Việt (N01)
│   ├── requirements.txt      # Danh sách thư viện Python
│   └── .env.example          # File mẫu biến môi trường (Token CoreIOT, LLM API Key)
├── hung-coreiot/             # Cloud CoreIOT & Video Demo (Hưng)
│   ├── dashboard.json        # File export Dashboard giám sát & điều khiển (T11)
│   ├── rulechain.json        # File export RuleChain tự động hóa & cảnh báo (T12, T13)
│   └── demo_notes.md         # Kịch bản 5 cảnh quay & link video demo
├── report/                   # Mã nguồn LaTeX và hình ảnh báo cáo PDF cuối kỳ
└── README.md                 # Tài liệu tổng quan dự án
```

---

## 4. Chuẩn Giao tiếp Dữ liệu (JSON Data Contract)

**1. Gói tin chuẩn hóa từ YoloUNO S3 (Tasmota) về PC Gateway:**
```json
{
  "device_id": "YoloUNO_S3_Node",
  "temperature": 32.5,
  "humidity": 70.1,
  "soil_moisture": 25.0,
  "pir_motion": true,
  "fan_status": "OFF",
  "pump_status": "OFF"
}
```
*(Tasmota mặc định gửi thông số DHT20 dưới khóa `"DHT20"` hoặc `"AHT20"`, script Python Gateway sẽ tự động chuẩn hóa về định dạng trên).*

**2. Gói tin từ PC Gateway (sau khi AI phân tích) đẩy lên CoreIOT:**
```json
{
  "temperature": 32.5,
  "humidity": 70.1,
  "soil_moisture": 25.0,
  "pir_motion": true,
  "fan_status": "ON",
  "pump_status": "OFF",
  "AI_event": "Fire_Warning",
  "AI_confidence": 0.95
}
```

---

## 5. Hướng dẫn Cài đặt & Chạy nhanh

1. **Phần cứng (Node):** Nạp firmware Tasmota (ESP32-S3) lên YoloUNO, cấu hình chân GPIO theo `hy-hardware/tasmota_config.md` và trỏ MQTT Host về địa chỉ IP máy tính chạy PC Gateway.
2. **PC Gateway:**
   ```bash
   cd huy-gateway
   pip install -r requirements.txt
   cp .env.example .env   # Điền CoreIOT Access Token & LLM API Key vào file .env
   python mqtt_bridge.py
   ```
3. **CoreIOT:** Import `hung-coreiot/dashboard.json` và `hung-coreiot/rulechain.json` lên tài khoản CoreIOT của nhóm.

---

## 6. Quy trình Nộp file trên GitHub (Cho thành viên)

1. Mở repo trên GitHub, chọn **đúng branch riêng của mình** ở góc trên bên trái.
2. Mở đúng thư mục cá nhân (`hy-hardware/`, `huy-gateway/`, hoặc `hung-coreiot/`).
3. Chọn **Add file** $\rightarrow$ **Upload files**, kéo thả file cần nộp, ghi chú commit (VD: `[Huy][T08] Up code MQTT bridge`) và bấm **Commit changes**.

> **Lưu ý bảo mật:** Kho lưu trữ để chế độ **Public** để giảng viên chấm điểm. Tuyệt đối **không** đẩy lên các file chứa mật khẩu Wi-Fi, Access Token CoreIOT thật hoặc API Key của LLM (như file `.env`).
