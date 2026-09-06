===============================================================================
4. Module limitation (dtb - kernel 4.9)
===============================================================================

.. meta::
   :description: Technical specifications and hardware capabilities of Banana Pi BPI-M4 (RTD1395)
   :keywords: Banana Pi, BPI-M4, RTD1395, Embedded Linux, GPIO, I2C, UART, Edge AI

.. contents:: **Mục lục chi tiết**
   :depth: 2
   :local:
   :backlinks: entry

---

Tổng quan Kiến trúc & Tài nguyên Hệ thống
=========================================

Bo mạch **Banana Pi BPI-M4** dựa trên vi xử lý Realtek RTD1395 (Quad-core ARM Cortex-A53 64-bit) kết hợp với **2GB RAM** (trên bản Linux Community/Armbian). Hệ thống cung cấp khả năng tính toán đơn/đa luồng mạnh mẽ, đáp ứng hoàn hảo cho các tác vụ xử lý tại biên (Edge Computing).

.. note::
   Tài liệu này tập trung vào **tính năng kỹ thuật thuần túy** và **khả năng vận hành phần cứng** của hệ thống, bỏ qua các cấu hình phần cứng không ổn định (như Hardware SPI nội bộ).

.. image:: https://docs.banana-pi.org/en/BPI-M4/bpi-m4_top.jpg
   :alt: Banana Pi BPI-M4 Hardware Top View
   :align: center
   :width: 800px

---

1. Khả năng Tính toán & Xử lý Phần mềm (Computing Capabilities)
================================================================

Xử lý Đa nhiệm & Môi trường Đóng gói
------------------------------------
* **Xử lý đa nhiệm Linux (Multi-tasking):**
  Cho phép thực thi đồng thời nhiều dịch vụ nền (*Python, C/C++, Node.js, Go*) nhờ kiến trúc 4 nhân 64-bit và bộ nhớ RAM 2GB khả dụng.
* **Môi trường Containerization:**
  Hỗ trợ khởi chạy các môi trường đóng gói isolated (*Docker, Podman*) trực tiếp trên board mà không gây tràn bộ nhớ SWAP.

Thực thi AI/ML trên CPU (CPU Inference)
---------------------------------------

Cho phép chạy trực tiếp các Framework Machine Learning và Computer Vision mà không cần NPU phần cứng:

.. code-block:: text

   +-------------------------------------------------------------------+
   |                    AI/ML Frameworks on CPU                        |
   +----------------------------------+--------------------------------+
   | OpenCV / ONNX Runtime C++        | TensorFlow Lite C++ API        |
   | Scikit-Learn / SciPy             | MediaPipe (CPU-optimized)      |
   +----------------------------------+--------------------------------+

* **Các dạng bài toán xử lý được:**

  #. *Time-series Data:* Phân tích chuỗi thời gian, phát hiện bất thường (Anomaly Detection).
  #. *Classification:* Phân loại dữ liệu cảm biến đa chiều.
  #. *Lightweight Computer Vision:* Phát hiện vật thể (Object Detection - YOLOv8-nano/MobileNet) ở tốc độ khung hình phù hợp.

Quản trị Dữ liệu & Mạng
-----------------------

* **Local Database:** Đọc/Ghi/Truy vấn dữ liệu tần suất cao bằng *SQLite, InfluxDB, Redis, PostgreSQL*.
* **Networking Services:** Routing, VPN Server, MQTT Broker (*Mosquitto*), Web Server (*Nginx/Apache*), SSH Management.

---

2. Khả năng Giao tiếp qua Cổng USB (USB Subsystem)
===================================================

Các cổng USB Host của BPI-M4 hoạt động độc lập với sơ đồ Device Tree nội bộ, cung cấp khả năng mở rộng phần cứng linh hoạt và tức thì:

.. list-table:: **Khả năng mở rộng ngoại vi qua USB Host**
   :widths: 30 70
   :header-rows: 1
   :stub-columns: 0

   * - Thiết bị USB Ngoại vi
     - Tính năng phần cứng đạt được
   * - **USB Camera (UVC Standard)**
     - Bắt luồng Video/Hình ảnh tĩnh chuẩn RGB/YUV trực tiếp vào bộ nhớ RAM.
   * - **USB-to-SPI / I2C / RS485 / RS232**
     - Khôi phục **100% khả năng giao tiếp SPI** (*via FT232H / CH341A*) bypass hoàn toàn lỗi DTB.
   * - **USB AI Accelerator**
     - Bổ sung bộ tăng tốc Neural Network chuyên dụng (*Google Coral Edge TPU*) qua đường USB.
   * - **USB Audio Adapter**
     - Ghi âm đầu vào (Microphone) và xuất tín hiệu âm thanh (Audio Output).
   * - **USB Storage (SSD/Flash)**
     - Mở rộng dung lượng bộ nhớ lưu trữ tốc độ cao, làm bộ nhớ SWAP hoặc chứa DB lớn.

---

3. Khả năng Điều khiển qua Giao tiếp I2C
========================================

Bus I2C tích hợp (*SDA, SCL*) trên SoC RTD1395 vận hành ổn định trên Linux kernel với các tính năng:

* **Đọc dữ liệu từ cảm biến (Sensor Polling):**
  Thu thập dữ liệu môi trường (*Nhiệt độ, Độ ẩm, Áp suất, Gia tốc, Từ trường, Quang phổ*) với tốc độ lên tới **400kHz (Fast-mode I2C)**.
* **Xuất dữ liệu hiển thị (Display Output):**
  Truyền khung hình/ký tự ra các loại màn hình hiển thị (*OLED SSD1306, SH1106, LCD 1602/2004*).
* **Đồng bộ Thời gian thực (Real-time Clock):**
  Duy trì cấu hình thời gian chuẩn xác qua IC RTC ngoại vi (*DS3231*) ngay cả khi ngắt kết nối Internet/Nguồn điện.

---

4. Khả năng Điều khiển qua Giao tiếp UART (Serial)
==================================================

Chuẩn giao tiếp chuỗi bất đồng bộ qua các cổng tty phần cứng (``/dev/ttyS*`` hoặc ``/dev/ttyUSB*``):

* **Stream Dữ liệu Chuỗi (Text & Binary):**
  Truyền nhận dữ liệu dạng gói với tốc độ Baudrate tùy chỉnh linh hoạt từ ``9600`` đến ``115200+``.
* **Giao tiếp Thiết bị Đo đạc Chuyên dụng:**
  Đọc trực tiếp dữ liệu định vị (*GPS NEO-6M/M8N*), chỉ số môi trường (*Bụi mịn PMS5003/Khí CO2*), dữ liệu thẻ từ (*RFID/NFC RC522*).
* **Truyền thông Không dây Tầm xa (RF/LoRa):**
  Gửi/Nhận dữ liệu mã hóa qua các module thu phát không dây tầm xa (*LoRa E32*).

---

5. Khả năng Điều khiển GPIO & Bit-Banging SPI
=============================================

Tập hợp các chân Digital Input/Output trên hàng Header 40-pin cung cấp khả năng điều khiển phần cứng cấp thấp:

Digital Output (3.3V Logic Level)
---------------------------------
* Bật/Tắt trạng thái Logic (HIGH/LOW) để kích hoạt Relay, Op-amp, Còi buzzer, Đèn báo.
* Điều hướng xung **Software PWM** để chỉnh độ sáng LED hoặc định vị góc quay động cơ Servo.

Digital Input
-------------
* Phát hiện sự thay đổi trạng thái tín hiệu (*Interrupt / Polling*) từ Nút nhấn, Công tắc hành trình, Cảm biến chuyển động (PIR).

Software SPI (Bit-banging SPI)
------------------------------
Kỹ thuật giả lập tín hiệu bus dữ liệu phần cứng bằng phần mềm thông qua các chân GPIO tiêu chuẩn:

.. code-block:: python

   # Minh họa thuật toán Bit-banging SPI bằng GPIO tiêu chuẩn
   def bitbang_spi_send(data_byte, pin_clk, pin_mosi):
       for bit in range(8):
           # Gán giá trị bit dữ liệu lên chân MOSI
           gpio_write(pin_mosi, (data_byte >> (7 - bit)) & 0x01)
           # Kéo xung Clock HIGH -> LOW để dịch dữ liệu
           gpio_write(pin_clk, HIGH)
           gpio_write(pin_clk, LOW)

.. warning::
   Phương pháp **Bit-banging SPI** cho phép giao tiếp thành công 100% với các màn hình TFT (như ST7735) và mọi module SPI mà **không cần bộ điều khiển Hardware SPI hay tùy chỉnh Device Tree**.

---

.. important::
   Tất cả các tính năng trên đã được xác minh khả năng vận hành 100% trên nền tảng **Armbian / Linux Community Kernel** của bo mạch Banana Pi BPI-M4.

