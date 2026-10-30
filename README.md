# 🌦️ ESP32 Weather Station - Lấy dữ liệu thời tiết từ OpenWeatherMap API

Dự án sử dụng vi điều khiển **ESP32** kết nối Wi-Fi để gửi yêu cầu **HTTP GET** đến máy chủ [OpenWeatherMap](https://openweathermap.org/). Dữ liệu thời tiết trả về dưới định dạng **JSON** sẽ được bóc tách (parse) để lấy các thông số thực tế như: Nhiệt độ, Áp suất khí quyển, Độ ẩm, Tốc độ gió và hiển thị trực tiếp lên màn hình Serial Monitor.

🎥 **Video tham khảo & Demo thực tế:** [Xem video trên Google Drive tại đây](https://drive.google.com/file/d/10eO2XUFdIVUjTyKBRSjY8WS1y2xQYXGu/view?usp=sharing)

---

## 🛠️ Môi trường phát triển & Cài đặt thư viện

Dự án này được lập trình cho mạch **ESP32** bằng **Visual Studio Code (PlatformIO)**. Tuy nhiên, bạn hoàn toàn có thể chạy mã nguồn này trên **Arduino IDE** bằng cách cài đặt đúng thư viện theo hướng dẫn bên dưới:

### 1. Nếu sử dụng VS Code + PlatformIO (Môi trường gốc của dự án)
* Mở trực tiếp thư mục dự án bằng phần mềm VS Code có cài sẵn tiện ích **PlatformIO IDE**.
* Mã nguồn chính nằm tại đường dẫn `src/main.cpp`.
* Các thư viện cần thiết sẽ được PlatformIO tự động tải về thông qua cấu hình trong file `platformio.ini`:
  ```ini
  [env:esp32doit-devkit-v1]
  platform = espressif32
  board = esp32doit-devkit-v1
  framework = arduino
  monitor_speed = 115200
  lib_deps =
      arduino-libraries/Arduino_JSON
  ```

### 2. Nếu sử dụng Arduino IDE
Nếu bạn dùng Arduino IDE, hãy tạo một Sketch mới (`.ino`), sao chép toàn bộ code trong file `src/main.cpp` dán vào và thực hiện cài đặt:
* **Cài đặt gói Board ESP32:** 
  * Vào **Tools** $\rightarrow$ **Board** $\rightarrow$ **Boards Manager...**, tìm từ khóa **`esp32`** (của *Espressif Systems*) và nhấn **Install**.
  * Hai thư viện `<WiFi.h>` và `<HTTPClient.h>` đã được tích hợp sẵn bên trong gói cài đặt board ESP32 này, không cần tải ngoài.
* **Cài đặt thêm thư viện xử lý JSON (Bắt buộc):**
  * Vào **Sketch** $\rightarrow$ **Include Library** $\rightarrow$ **Manage Libraries...** (hoặc nhấn biểu tượng **Library Manager** ở thanh công cụ bên trái).
  * Gõ vào ô tìm kiếm từ khóa: **`Arduino_JSON`** (Tác giả: **Arduino**).
  * Nhấn **Install** để cài đặt.
  * *(⚠️ Lưu ý: Chọn đúng thư viện tên là **`Arduino_JSON`** có dấu gạch dưới của Arduino, tránh nhầm lẫn với thư viện `ArduinoJson` viết liền của Benoit Blanchon vì cú pháp giải mã trong code khác nhau).*

---

## 🔑 Hướng dẫn lấy API Key và Tra cứu địa điểm

Để dự án hoạt động, bạn cần có một mã **API Key** miễn phí từ OpenWeatherMap:

1. **Đăng ký tài khoản:** Truy cập trang chủ [OpenWeatherMap](https://openweathermap.org/) và tạo một tài khoản miễn phí.
2. **Lấy API Key:** 
   * Đăng nhập và truy cập trực tiếp vào trang quản lý khóa cá nhân tại: [https://home.openweathermap.org/api_keys](https://home.openweathermap.org/api_keys).
   * Sao chép chuỗi **Key** mặc định hoặc tạo một Key mới để điền vào mã nguồn. *(Lưu ý: API Key mới tạo có thể cần từ 15 phút đến 2 giờ để hệ thống kích hoạt).*
3. **Tra cứu tên Thành phố (`city`) và Mã quốc gia (`countryCode`):**
   * Truy cập công cụ tìm kiếm địa điểm tại: [https://openweathermap.org/find](https://openweathermap.org/find).
   * Nhập tên thành phố không dấu để kiểm tra định dạng chuẩn trên hệ thống (ví dụ: `Da Nang, VN`, `Hanoi, VN`, `Tokyo, JP`).

---

## ⚙️ Cấu hình mã nguồn (`src/main.cpp`)

Mở file `src/main.cpp` (hoặc file `.ino` trên Arduino IDE) và cập nhật các thông tin cá nhân của bạn tại các dòng sau:

```cpp
// 1. Nhập tên và mật khẩu Wi-Fi
const char *ssid = "TEN_WIFI_CUA_BAN";
const char *password = "MAT_KHAU_WIFI";

// 2. Dán API Key lấy từ OpenWeatherMap
String openWeatherMapApiKey = "NHAP_API_KEY_CUA_BAN_VAO_DAY";

// 3. Nhập tên thành phố và mã quốc gia (Chuẩn ISO 3166-1 alpha-2)
String city = "DaNang";
String countryCode = "VN";
```

### ⚠️ Lưu ý quan trọng khi cấu hình:
* **Xử lý khoảng trắng trong URL (Tránh lỗi `HTTP 400 Bad Request`):** Giao thức HTTP không cho phép gửi trực tiếp ký tự khoảng trắng trong đường dẫn URL. Nếu tên thành phố có dấu cách, hãy viết liền (ví dụ: `"DaNang"`) hoặc thay khoảng trắng bằng mã **`%20`** (ví dụ: `"Da%20Nang"`, `"Ho%20Chi%20Minh"`).
* **Đổi đơn vị nhiệt độ sang độ C (Celsius):** Mặc định OpenWeatherMap trả về nhiệt độ theo độ **Kelvin (K)**. Để nhận trực tiếp giá trị **độ C**, hãy thêm tham số `&units=metric` vào cuối biến `serverPath`:
  ```cpp
  String serverPath = "[http://api.openweathermap.org/data/2.5/weather?q=](http://api.openweathermap.org/data/2.5/weather?q=)" + city + "," + countryCode + "&APPID=" + openWeatherMapApiKey + "&units=metric";
  ```
* **Chu kỳ lấy dữ liệu (`timerDelay`):** Mặc định biến `timerDelay = 10000` (10 giây/lần) để phục vụ mục đích kiểm thử. Khi chạy thực tế, nên chỉnh lên `600000` (10 phút/lần) để tránh vượt quá giới hạn gọi API miễn phí của máy chủ.

---

## 🌐 Kiểm tra nhanh API trên trình duyệt máy tính

Trước khi nạp code vào mạch ESP32, bạn có thể kiểm tra trước chuỗi JSON trả về bằng cách dán đường dẫn sau lên thanh địa chỉ của trình duyệt web (Chrome, Edge,...):

```text
[http://api.openweathermap.org/data/2.5/weather?q=DaNang,VN&APPID=YOUR_API_KEY](http://api.openweathermap.org/data/2.5/weather?q=DaNang,VN&APPID=YOUR_API_KEY)
```
*(Thay `YOUR_API_KEY` bằng mã API Key thực tế của bạn. Khuyên dùng thêm tiện ích mở rộng **JSON Formatter** trên trình duyệt để quan sát cấu trúc cây JSON rõ ràng hơn).*

---

## 🖥️ Kết quả hiển thị mẫu (Serial Monitor)

Mở Serial Monitor ở tốc độ Baudrate **`115200`**, kết quả trả về khi mạch kết nối và bóc tách dữ liệu thành công:

```text
Connecting
....
Connected to WiFi network with IP Address: 192.168.1.105
Timer set to 10 seconds (timerDelay variable), it will take 10 seconds before publishing the first reading.
HTTP Response code: 200
{"coord":{"lon":108.2208,"lat":16.0678},"weather":[{"id":804,"main":"Clouds","description":"overcast clouds","icon":"04n"}],"base":"stations","main":{"temp":299.35,"feels_like":299.35,"temp_min":299.35,"temp_max":299.35,"pressure":1017,"humidity":86,"sea_level":1017,"grnd_level":1012},"visibility":9676,"wind":{"speed":1.27,"deg":111,"gust":2.05},"clouds":{"all":97},"dt":1793282087,"sys":{"country":"VN","sunrise":1793227371,"sunset":1793269121},"timezone":25200,"id":1583992,"name":"Turan","cod":200}
Temperature: 299.35
Pressure: 1017
Humidity: 86
Wind Speed: 1.27
```

* **`HTTP Response code: 200`**: Yêu cầu hợp lệ, máy chủ phản hồi thành công.
* **`Temperature: 299.35`**: Nhiệt độ hiện tại tính theo độ K ($299.35 - 273.15 = 26.2^\circ\text{C}$).
* **`Pressure: 1017`**: Áp suất khí quyển ($1017\text{ hPa}$).
* **`Humidity: 86`**: Độ ẩm không khí ($86\%$).
* **`Wind Speed: 1.27`**: Tốc độ gió ($1.27\text{ m/s}$).

---

## 📁 Cấu trúc thư mục dự án (PlatformIO)

```text
Get_API_Thoi_Tiet/
├── include/           # Chứa các file thư viện header (.h) tự định nghĩa
├── lib/               # Chứa các thư viện ngoài dùng riêng cho dự án
├── src/
│   └── main.cpp       # Mã nguồn chính của chương trình
├── test/              # Chứa các file kiểm thử (Unit Test)
├── .gitignore         # Cấu hình bỏ qua thư mục biên dịch (.pio) khi đẩy lên Git
├── platformio.ini     # File cấu hình board mạch, tốc độ nạp và thư viện PlatformIO
└── README.md          # Tài liệu hướng dẫn sử dụng dự án
```

---

## 📚 Tài liệu & Liên kết tham khảo

* **Video thực hiện & tham khảo:** [Xem trên Google Drive](https://drive.google.com/file/d/10eO2XUFdIVUjTyKBRSjY8WS1y2xQYXGu/view?usp=sharing)
* **Bài viết hướng dẫn gốc (Random Nerd Tutorials - Rui Santos):** [ESP32 HTTP GET with Arduino IDE (OpenWeatherMap.org and ThingSpeak)](https://randomnerdtutorials.com/esp32-http-get-open-weather-map-thingspeak-arduino/#:~:text=Go%20to%20this%20link%3A%20https%3A%2F%2Fhome.openweathermap.org%2Fapi_keys%20and%20get%20your,you%E2%80%99ll%20need%20to%20pull%20information%20from%20the%20site.)
* **Cổng dữ liệu thời tiết:** [OpenWeatherMap Official Website](https://openweathermap.org/)
* **Tài liệu API thời tiết hiện tại:** [Current Weather Data API Doc](https://openweathermap.org/current)
