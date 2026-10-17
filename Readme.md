# 🚀 Smart-Farm – Hệ thống IoT chăm sóc Lan thông minh

> **Tài liệu hướng dẫn cài đặt, vận hành và xử lý sự cố** dành cho người quản lý trang trại Lan.  
> Phiên bản: `1.0.0` · Đối tượng sử dụng: người vận hành và kỹ thuật viên.

## 1. Mô tả dự án

**Smart-Farm** là hệ thống IoT hỗ trợ giám sát điều kiện môi trường và tự động tưới nước cho vườn Lan tại trang trại Lan. Hệ thống thu thập dữ liệu từ cảm biến, xử lý tại bộ điều khiển, điều khiển bơm/van theo ngưỡng cài đặt và gửi dữ liệu lên bảng điều khiển để theo dõi từ xa.

Mục tiêu chính:

- Giảm công việc kiểm tra và tưới thủ công.
- Hạn chế tưới quá nhiều hoặc quá ít nhờ dữ liệu cảm biến.
- Theo dõi lịch sử môi trường, trạng thái thiết bị và cảnh báo.
- Hỗ trợ người vận hành ra quyết định dựa trên dữ liệu.

> **Lưu ý:** Các ngưỡng nhiệt độ, độ ẩm và thời gian tưới trong tài liệu là giá trị minh họa. Hãy điều chỉnh theo giống Lan, giá thể, giai đoạn sinh trưởng và hướng dẫn của kỹ thuật viên.

## 2. Hướng dẫn cài đặt

### 2.1. Thiết bị và phần mềm cần có

- Bộ điều khiển tương thích ESP32 (SoC/vi điều khiển).
- Cảm biến nhiệt độ/độ ẩm không khí, cảm biến độ ẩm đất hoặc giá thể.
- Module relay phù hợp tải, máy bơm hoặc van điện từ.
- Nguồn điện đúng thông số của từng thiết bị; hộp bảo vệ chống ẩm.
- Máy tính có Arduino IDE 2.x, cáp USB dữ liệu và mạng Wi-Fi.
- Thư viện cảm biến và MQTT tương ứng với phần cứng đang dùng.

### 2.2. Lấy mã nguồn

Mở Terminal/PowerShell và chạy:

```bash
git clone https://github.com/trang-trai-lan/smart-farm.git
cd smart-farm
```

> Đường dẫn trên là ví dụ minh họa. Nếu trang trại sử dụng kho mã nguồn riêng, hãy thay bằng URL do người quản trị cung cấp.

### 2.3. Cấu hình

1. Mở thư mục dự án trong Arduino IDE hoặc môi trường phát triển do kỹ thuật viên cung cấp.
2. Cài đặt đúng thư viện theo tệp `README`, `platformio.ini` hoặc hướng dẫn đi kèm dự án.
3. Sao chép `config.example.h` thành `config.h` (nếu dự án có tệp mẫu).
4. Điền thông tin Wi-Fi và máy chủ MQTT trong tệp cấu hình cục bộ.

Ví dụ cấu hình — **thay bằng thông tin thực tế; không đưa mật khẩu thật lên Git**:

```cpp
#define WIFI_SSID       "TEN_WIFI_TRANG_TRAI"
#define WIFI_PASSWORD   "MAT_KHAU_WIFI"
#define MQTT_HOST       "192.168.1.10"
#define MQTT_PORT       1883
```

5. Chọn đúng board và cổng COM trong IDE.
6. Nạp chương trình lên bộ điều khiển, sau đó mở Serial Monitor để kiểm tra kết nối và dữ liệu cảm biến.

### 2.4. Kiểm tra trước khi vận hành

- Đảm bảo dây cảm biến, relay, bơm/van và nguồn điện được lắp đặt đúng sơ đồ.
- Thử từng cảm biến và xác nhận dữ liệu hiển thị hợp lý.
- Thử relay ở chế độ kiểm tra, không để bơm chạy khô.
- Kiểm tra chế độ thủ công và nút dừng khẩn cấp (nếu có).
- Chạy thử một khu vực nhỏ trước khi đưa vào vận hành toàn trang trại.

## 3. Tính năng

- [x] Thu thập nhiệt độ, độ ẩm không khí và độ ẩm giá thể theo chu kỳ.
- [x] Hiển thị trạng thái cảm biến và thiết bị trên màn hình/bảng điều khiển (nếu được cấu hình).
- [x] Điều khiển bơm nước hoặc van điện từ theo ngưỡng.
- [x] Hỗ trợ điều khiển thủ công để kiểm tra và bảo trì.
- [x] Gửi dữ liệu và cảnh báo qua mạng khi có cấu hình máy chủ.
- [x] Ghi nhật ký trạng thái, dữ liệu cảm biến và sự kiện điều khiển.
- [ ] Tích hợp dự báo tưới theo thời tiết — tùy chọn, cần cấu hình dịch vụ tương ứng.

## 4. Sơ đồ hệ thống

![Sơ đồ hệ thống Smart-Farm: cảm biến gửi dữ liệu đến bộ điều khiển ESP32, sau đó điều khiển bơm/van và đồng bộ dữ liệu qua Wi-Fi lên máy chủ/bảng điều khiển.](docs/smart-farm-architecture.svg)

**Luồng hoạt động tổng quát:** Cảm biến → Bộ điều khiển (SoC) → quyết định điều khiển → relay → bơm/van. Dữ liệu và nhật ký có thể được gửi qua Wi-Fi đến máy chủ hoặc ứng dụng giám sát.

### 4.1. Chu kỳ Fetch–Decode–Execute khi nhận lệnh tưới

> **Fetch (Lấy lệnh):** CPU của bộ điều khiển lấy lệnh kế tiếp từ bộ nhớ chương trình, ví dụ lệnh kiểm tra điều kiện tưới hoặc cập nhật trạng thái relay.  
>
> **Decode (Giải mã):** CPU giải mã lệnh để xác định thao tác cần thực hiện, chẳng hạn đọc giá trị cảm biến, so sánh với ngưỡng hoặc đặt đầu ra GPIO.  
>
> **Execute (Thực thi):** CPU thực hiện thao tác: nếu điều kiện tưới hợp lệ, chương trình điều khiển chân GPIO để kích hoạt relay và bật bơm/van; nếu không, bơm tiếp tục tắt. CPU sau đó chuyển sang lệnh tiếp theo.
>
> Chu kỳ Fetch–Decode–Execute là mô hình hoạt động của CPU khi thực thi lệnh chương trình; quyết định tưới thực tế còn phụ thuộc vào logic điều khiển, cảm biến và các điều kiện an toàn được lập trình.

## 5. DIKW: Từ dữ liệu nhiệt độ đến hành động tưới

DIKW mô tả quá trình chuyển từ **Data (Dữ liệu)** → **Information (Thông tin)** → **Knowledge (Tri thức)** → **Wisdom (Trí tuệ/Quyết định)**.

```text
                  /\
                 /  \
                / WISDOM \       Quyết định: tưới, chờ hoặc cảnh báo
               /----------\
              / KNOWLEDGE  \     Áp dụng quy tắc và ngưỡng đã được kỹ thuật viên
             /--------------\    xác nhận cho giống Lan và giá thể
            /  INFORMATION   \   Diễn giải dữ liệu cùng thời gian, độ ẩm giá thể
           /------------------\
          /       DATA         \  Số đo cảm biến có dấu thời gian
         /______________________\
```

Ví dụ minh họa:

1. **Data – Dữ liệu:** cảm biến ghi nhiệt độ `33,5°C`, độ ẩm giá thể `35%`, thời gian `09:30`.
2. **Information – Thông tin:** hệ thống nhận thấy nhiệt độ đang cao và độ ẩm giá thể đang thấp so với ngưỡng cấu hình.
3. **Knowledge – Tri thức:** áp dụng quy tắc tưới đã được người phụ trách phê duyệt; xác nhận cảm biến hợp lệ, không ở chế độ bảo trì và bơm không báo lỗi.
4. **Wisdom – Quyết định:** nếu đủ điều kiện an toàn, tưới trong khoảng thời gian giới hạn rồi đo lại; nếu dữ liệu bất thường hoặc có lỗi, không tự động bật bơm và gửi cảnh báo.

**Không dùng nhiệt độ đơn lẻ làm căn cứ duy nhất để tưới.** Cần kết hợp độ ẩm giá thể, lịch tưới, trạng thái thiết bị và yêu cầu chăm sóc của từng giống Lan.

## 6. Bảng thông số kỹ thuật

| Hạng mục | Thông số tham khảo | Ghi chú |
|---|---|---|
| Bộ điều khiển | ESP32 hoặc thiết bị tương đương | Xác nhận đúng model thực tế |
| Nguồn bộ điều khiển | Theo thông số bo mạch | Thường cấp qua USB hoặc bộ nguồn phù hợp |
| Cảm biến nhiệt độ/độ ẩm không khí | Ví dụ: DHT22 | Dải đo phụ thuộc cảm biến |
| Cảm biến độ ẩm giá thể | Loại điện dung | Cần hiệu chuẩn theo giá thể thực tế |
| Giao tiếp | GPIO, I²C/1-Wire tùy cảm biến, Wi-Fi | Phụ thuộc cấu hình phần cứng |
| Điều khiển tải | Relay/driver phù hợp | Chọn theo điện áp và dòng tải |
| Thiết bị chấp hành | Bơm nước hoặc van điện từ | Không vận hành khi thiếu nước |
| Kết nối máy chủ | MQTT/HTTP tùy triển khai | Cần cấu hình mạng và bảo mật |
| Lưu trữ | Bộ nhớ thiết bị và/hoặc máy chủ | Theo dõi dung lượng nhật ký |
| Chu kỳ lấy mẫu | Ví dụ: 30–60 giây | Điều chỉnh theo yêu cầu vận hành |

## 7. Vận hành hằng ngày

1. Kiểm tra nguồn điện, kết nối mạng và trạng thái cảm biến.
2. Đối chiếu số đo với điều kiện thực tế tại vườn; kiểm tra cảm biến nếu số liệu bất thường.
3. Xác nhận bơm/van chỉ hoạt động khi có yêu cầu tưới hợp lệ.
4. Theo dõi cảnh báo, nhật ký và lịch sử tưới.
5. Cuối ngày, kiểm tra rò rỉ, nguồn nước và tình trạng thiết bị.

## 8. Lưu ý và xử lý sự cố

### RAM đầy do dữ liệu nhật ký

Nếu RAM tăng cao hoặc hệ thống chậm do nhật ký được giữ trong bộ nhớ:

1. **Đưa hệ thống về trạng thái an toàn:** dừng các tác vụ không thiết yếu. Nếu trạng thái điều khiển không còn đáng tin cậy, ngắt lệnh tưới tự động theo quy trình an toàn của trang trại.
2. **Không xóa dữ liệu tùy tiện:** nếu nhật ký chưa được lưu bền vững, xuất hoặc đồng bộ nhật ký cần thiết trước khi dọn dẹp, nếu còn khả năng.
3. **Giảm tải nhật ký:** giới hạn kích thước bộ đệm, chỉ giữ số dòng gần nhất trong RAM, dùng vòng đệm (ring buffer), và ghi mức `INFO` thay vì ghi dữ liệu lặp liên tục.
4. **Lưu nhật ký đúng nơi:** ghi theo lô vào Flash/SD hoặc gửi đến máy chủ; kiểm tra giới hạn ghi của bộ nhớ và tránh ghi Flash quá thường xuyên.
5. **Khởi động lại có kiểm soát:** chỉ khởi động lại khi trạng thái bơm/van đã an toàn; sau khi khởi động, kiểm tra cảm biến, kết nối và nhật ký trước khi bật lại tự động.
6. **Phòng ngừa:** theo dõi dung lượng heap/RAM, đặt ngưỡng cảnh báo, xoay vòng tệp log và kiểm tra rò rỉ bộ nhớ định kỳ.

### Các sự cố thường gặp

- **Không có dữ liệu cảm biến:** kiểm tra dây, nguồn, chân GPIO và cấu hình cảm biến.
- **Không kết nối Wi-Fi/MQTT:** kiểm tra nguồn phát Wi-Fi, SSID, mật khẩu, địa chỉ máy chủ và cổng.
- **Bơm không chạy:** kiểm tra nguồn bơm, cầu chì, relay/driver, mực nước và chế độ bảo vệ.
- **Bơm chạy liên tục:** chuyển sang chế độ an toàn/thủ công, ngắt nguồn tải theo quy trình, kiểm tra cảm biến và ngưỡng điều khiển.
- **Số đo bất thường:** so sánh với thiết bị đo độc lập, vệ sinh và hiệu chuẩn cảm biến.

> **An toàn điện:** Không chạm vào dây điện hoặc module relay khi đang cấp điện. Việc đấu nối tải điện lưới phải do người có chuyên môn thực hiện. Không vận hành bơm khi thiếu nước hoặc khi có rò rỉ.

## 9. Bảo mật và bảo trì

- Không lưu mật khẩu Wi-Fi, khóa API hoặc thông tin xác thực thật trong kho mã nguồn công khai.
- Đổi mật khẩu mặc định và giới hạn quyền truy cập bảng điều khiển.
- Sao lưu cấu hình, cập nhật firmware có kiểm soát và giữ bản firmware có thể khôi phục.
- Kiểm tra cảm biến, dây dẫn, van, bơm và hộp chống ẩm theo lịch bảo trì.
- Ghi lại thời gian, nguyên nhân và cách xử lý mỗi sự cố.

## 10. Hỗ trợ

Khi báo lỗi cho kỹ thuật viên, cung cấp: thời điểm xảy ra lỗi, mã lỗi/cảnh báo, số đo cảm biến, trạng thái bơm/van, phiên bản firmware và đoạn nhật ký liên quan. **Không gửi mật khẩu hoặc khóa truy cập** trong báo cáo.

---

*Tài liệu Smart-Farm dành cho trang trại Lan. Cấu hình thực tế phải được xác nhận theo phần cứng đã lắp đặt và quy trình vận hành tại trang trại.*
