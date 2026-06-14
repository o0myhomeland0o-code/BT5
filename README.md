# BT5 : TRIỂN KHAI HỆ THỐNG GIÁM SÁT REALTIME VÀ CẢNH BÁO TỰ ĐỘNG QUA TELEGRAM BOT BẰNG DOCKER COMPOSE  
* **Họ tên:** Phạm Duy Tiến Minh
* **MSSV:** K225480106048 
* **Lớp:** K58KTP  

---

## CHƯƠNG I: PHẦN LÝ THUYẾT HỆ THỐNG ĐỒNG BỘ (THEORETICAL FOUNDATION)

### 1. Bản chất của Công nghệ Ảo hóa Container - Docker là gì?
**Docker** là một nền tảng mã nguồn mở mang tính cách mạng trong việc đóng gói, phân phối và triển khai ứng dụng dưới dạng các môi trường biệt lập được gọi là **Container**. 

Trước khi Docker xuất hiện, kiến trúc ảo hóa truyền thống phụ thuộc hoàn toàn vào **Máy ảo (Virtual Machines - VM)**. Mỗi một VM đòi hỏi phải gánh một Hệ điều hành khách (Guest OS) hoàn chỉnh chạy trên nền của Hypervisor, dẫn đến việc lãng phí một lượng lớn tài nguyên RAM, CPU, dung lượng lưu trữ, và tốc độ khởi động rất chậm (vài phút).

Docker giải quyết triệt để vấn đề này nhờ công nghệ ảo hóa ở cấp độ hệ điều hành (OS-level virtualization). Thay vì cài đặt một OS mới, Docker **chia sẻ chung nhân (Kernel)** của Hệ điều hành máy chủ (Host OS) thông qua các tính năng cốt lõi của Linux như `Namespaces` (cách ly tiến trình, mạng, người dùng) và `Cgroups` (giới hạn tài nguyên phần cứng). Mỗi Container chỉ đóng gói duy nhất mã nguồn ứng dụng, các thư viện phụ thuộc (Dependencies) và các file cấu hình tối thiểu. Điều này giúp các Container khởi động chỉ trong vài giây, dung lượng siêu nhẹ (chỉ vài chục MB) và đạt hiệu suất xử lý gần như tương đương với việc chạy trực tiếp trên máy thật.

### 2. Từ điển các Keyword cốt lõi trong tệp cấu hình `docker-compose.yml`
File `docker-compose.yml` định dạng cú pháp YAML đóng vai trò cấu trúc và quản lý một tập hợp nhiều Container (Multi-container Application). Dưới đây là danh sách tường minh các từ khóa, ý nghĩa kỹ thuật sâu và ví dụ thực tế:

#### A. Các Từ khóa khai báo cấp cao nhất (Top-Level Elements)
* **`version:`** Định nghĩa phiên bản định dạng đặc tả của file Docker Compose (Ví dụ: `'3.8'`). Việc khai báo này giúp Docker Engine áp dụng đúng các tập lệnh và tính năng tương thích.
* **`services:`** Khối lệnh bắt đầu để định nghĩa cấu hình chi tiết cho từng container thành phần (như cơ sở dữ liệu, backend, frontend) sẽ chạy trong hệ thống.
* **`networks:`** Khai báo các mạng ảo nội bộ cô lập. Tất cả container tham gia chung vào một mạng có thể tự do kết nối và giao tiếp dữ liệu nội bộ với nhau một cách an toàn thông qua "Tên Dịch Vụ" (Service Name) thay vì địa chỉ IP động.
* **`volumes:`** Khai báo không gian lưu trữ dữ liệu bền vững (Persistent Data) độc lập với vòng đời của Container. Dữ liệu ghi vào Volume sẽ được lưu lại trực tiếp trên ổ cứng máy thật, không bị mất đi ngay cả khi container bị xóa hoàn toàn.

#### B. Các Từ khóa khai báo chi tiết bên trong từng Service (Service-Level Properties)

| Từ khóa | Ý nghĩa kỹ thuật chi tiết | Ví dụ minh họa thực tế |
| :--- | :--- | :--- |
| **`image`** | Chỉ định mẫu thiết kế (Docker Image) có sẵn trên Docker Hub hoặc local làm nền tảng để tạo lập Container. | `image: mariadb:10.6` |
| **`build`** | Chỉ định đường dẫn tới một thư mục chứa `Dockerfile`. Docker Compose sẽ tự động biên dịch (build) mã nguồn cục bộ thành Image riêng thay vì tải từ Internet. | `build: ./api` |
| **`container_name`** | Đặt tên định danh cố định và tường minh cho container, giúp người dùng dễ dàng theo dõi log, bật/tắt chính xác container đó. | `container_name: monitor_nodered` |
| **`ports`** | Ánh xạ cấu hình cổng theo cú pháp `CỔNG_MÁY_THẬT:CỔNG_CONTAINER`. Giúp mở đường cho các yêu cầu từ mạng ngoài truy cập trực tiếp vào dịch vụ bên trong container. | `ports: - "1880:1880"` |
| **`environment`** | Khai báo các biến môi trường cấu hình hệ thống, cho phép truyền dữ liệu bảo mật (như mật khẩu root, tên database, tài khoản truy cập) vào container. | `environment: - MYSQL_ROOT_PASSWORD=root_password` |
| **`volumes`** | Liên kết (mount) một ổ đĩa đã khai báo ở top-level hoặc một đường dẫn thư mục máy thật vào một thư mục bên trong container để lưu hoặc đồng bộ dữ liệu. | `volumes: - mariadb_data:/var/lib/mysql` |
| **`networks`** | Chỉ định container cụ thể tham gia vào một hoặc nhiều mạng nội bộ để cấu hình phân quyền giao tiếp chéo. | `networks: - monitor_net` |
| **`depends_on`** | Thiết lập sự phụ thuộc và thứ tự khởi động. Ví dụ: Backend API hay Node-RED phải chờ cho Database MariaDB/InfluxDB khởi động thành công trước thì mới được phép chạy. | `depends_on: - mariadb` |
| **`restart`** | Định cấu hình hành vi tự động khởi chạy lại của container nếu ứng dụng bên trong bị lỗi đột ngột (crash) hoặc khi hệ thống máy chủ vật lý bị khởi động lại. | `restart: always` |

### 3. Ưu điểm vượt trội khi triển khai ứng dụng bằng Docker
* **Giải quyết triệt để bài toán đồng bộ môi trường:** Loại bỏ hoàn toàn câu nói kinh điển của lập trình viên: *"Code chạy tốt trên laptop cá nhân của em nhưng mang lên server chấm điểm của thầy thì bị lỗi"*. Mọi thứ từ hệ điều hành nền, phiên bản python, thư viện bổ trợ đều được đóng gói tĩnh 100% vào Image.
* **Tối ưu hóa hiệu năng và chi phí phần cứng:** Không giống như máy ảo VM, các container không chiếm trước dung lượng RAM cố định. Hệ thống có thể chạy đồng thời 5, 6 dịch vụ khác nhau trên một máy tính xách tay cấu hình văn phòng một cách mượt mà.
* **Kiến trúc microservices độc lập và an toàn:** Sự cố phát sinh từ một dịch vụ (Ví dụ: Node-RED bị nghẽn mạng hoặc lỗi logic) sẽ bị cô lập hoàn toàn bên trong container đó, không thể tác động hay gây lỗi dây chuyền làm sập các dịch vụ kế cận như MariaDB hay Nginx.
* **Khả năng nhân bản và mở rộng thần tốc:** Đơn giản hóa việc chuyển đổi hạ tầng. Chỉ cần một câu lệnh duy nhất, toàn bộ hệ thống microservices phức tạp có thể được tái thiết lập hoàn chỉnh trên một máy chủ mới trong vòng chưa đầy một phút.

### 4. Quy trình triển khai Ứng dụng sang Máy chủ thực tế Offline (Không có Internet)
Khi ứng dụng đã qua các bước kiểm thử (test) đạt trạng thái ổn định trên laptop cá nhân, quy trình 4 bước chuẩn hóa để đóng gói và di chuyển hệ thống sang một máy chủ thật đặt ở phòng thí nghiệm, trung tâm dữ liệu hoặc môi trường nhà máy hoàn toàn không kết nối Internet được thực hiện như sau:

* **Bước 1 (Thực hiện trên máy laptop có Internet):** Tiến hành đóng gói toàn bộ các Docker Image của hệ thống đang chạy thành một file lưu trữ nén duy nhất có định dạng đuôi `.tar` thông qua lệnh:
  ```bash
  docker save -o deployment_offline_package.tar mariadb:10.6 influxdb:2.7 nodered/node-red:latest nginx:alpine monitor_flask_api:latest

 * **Bước 2 (Di chuyển vật lý dữ liệu):** Sao chép file nén deployment_offline_package.tar, file cấu hình tổng thể docker-compose.yml, thư mục giao diện tĩnh ./frontend và thư mục mã nguồn ./api vào một thiết bị lưu trữ ngoại vi (USB hoặc ổ cứng di động).

 * **Bước 3 (Nạp tài nguyên trên máy chủ Offline):** Kết nối thiết bị lưu trữ ngoại vi vào máy chủ thật. Mở Terminal tại thư mục nhận file trên máy chủ và thực hiện lệnh nạp (load) các image từ file nén vào Docker Engine cục bộ của máy chủ:

 ```bash
docker load -i deployment_offline_package.tar

 ```
 * **Bước 4 (Khởi chạy ứng dụng diện rộng):** Di chuyển trực tiếp  vào vị trí đặt file docker-compose.yml trên máy chủ thật và chạy lệnh kích hoạt hệ thống:

 ``` Bash
docker-compose up -d
 ```
## CHƯƠNG II: KIẾN TRÚC MÃ NGUỒN CẤU THÀNH HỆ THỐNG GIÁM SÁT REALTIME
## 1. Sơ đồ luồng dữ liệu kiến trúc hệ thống (Dataflow Architecture)
[Nguồn dữ liệu động (Gold/Weather)]   
           │ (HTTP Get / 5s)  
           ▼  
     [ NODE-RED ] ───(Nếu dữ liệu bất thường)───► [ Telegram Bot API ] ──► [ Nhóm 3 Thành Viên ]  
           │  
           ├───► [ MariaDB (Lưu giá trị tức thời) ] ◄─── [ Flask API ] ◄─── [ JS Fetch API / 2s ] ──┐  
           │                                                                                        ▼  
           └───► [ Grafana Dashboard qua thẻ IFRAME ]  

<img width="1921" height="1080" alt="giavang" src="https://github.com/user-attachments/assets/ba2191ef-7e23-4c92-9aa4-fe40682ba7f2" />

## 2. Tệp điều phối trung tâm: docker-compose.yml
YAMLversion: '3.8'

services:
  # 1. Cơ sở dữ liệu quan hệ lưu dữ liệu tức thời phục vụ Frontend API
 ``` Bash
  mariadb:
    image: mariadb:10.6
    container_name: monitor_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: monitor_db
      MYSQL_USER: monitor_user
      MYSQL_PASSWORD: user_password
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - monitor_net
```
  # 2. Cơ sở dữ liệu chuỗi thời gian (Time-Series) chuyên dụng để lưu trữ lịch sử
 ``` Bash
 influxdb:
    image: influxdb:2.7
    container_name: monitor_influxdb
    restart: always
    ports:
      - "8086:8086"
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=admin_password123
      - DOCKER_INFLUXDB_INIT_ORG=my_org
      - DOCKER_INFLUXDB_INIT_BUCKET=monitor_bucket
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitor_net
 ```
  # 3. Nền tảng điều phối dữ liệu tích hợp logic cảnh báo tự động
 ``` Bash
  nodered:
    image: nodered/node-red:latest
    container_name: monitor_nodered
    restart: always
    ports:
      - "1880:1880"
    volumes:
      - nodered_data:/data
    depends_on:
      - mariadb
      - influxdb
    networks:
      - monitor_net
 ``` 
  # 4. Backend Flask API làm cầu nối trung gian truy vấn dữ liệu từ MariaDB
 ``` Bash
  flask_api:
    build: ./api
    container_name: monitor_flask_api
    restart: always
    ports:
      - "5000:5000"
    depends_on:
      - mariadb
    networks:
      - monitor_net
 ```
  # 5. Webserver Nginx chịu trách nhiệm phân phối ứng dụng Frontend tĩnh
 ``` bash
  nginx:
    image: nginx:alpine
    container_name: monitor_nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
    depends_on:
      - flask_api
    networks:
      - monitor_net

volumes:
  mariadb_data:
  influxdb_data:
  nodered_data:

networks:
  monitor_net:
    driver: bridge
 ```
# Màn hình Terminal hiển thị tiến trình kéo dữ liệu image và khởi chạy thành công 5 cụm dịch vụ.

Lệnh chạy: docker-compose up -d

<img width="1467" height="701" alt="terminal" src="https://github.com/user-attachments/assets/10f78b7b-f934-4818-a002-02281322bfd4" />

# Danh sách bảng tiến trình thể hiện rõ tên dịch vụ, ID container và các cổng ánh xạ tương ứng ra bên ngoài máy host ở trạng thái "Up".

Lệnh thực hiện: docker ps -a

<img width="1474" height="702" alt="uP" src="https://github.com/user-attachments/assets/26a4e5d4-bfe4-4c9e-8ce7-ab500b8bc838" />


## 3. Thành phần Mã nguồn Backend Flask API (Thư mục ./api)
# A. File api/Dockerfile
 ``` Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
 ```
# B. File api/requirements.txtPlaintextflask
 ```
flask-cors
mysql-connector-python
 ```
# C. File api/app.pyPythonfrom flask import Flask, jsonify
 ```
from flask_cors import CORS
import mysql.connector
from datetime import datetime

app = Flask(__name__)
CORS(app) # Cho phép gọi API chéo từ cổng 80 của Nginx sang cổng 5000 của Flask

db_config = {
    'host': 'monitor_mariadb', # Kết nối nội bộ an toàn bằng tên service trong mạng Docker
    'user': 'monitor_user',
    'password': 'user_password',
    'database': 'monitor_db'
}

@app.route('/api/realtime', methods=['GET'])
def get_realtime_data():
    try:
        conn = mysql.connector.connect(**db_config)
        cursor = conn.cursor(dictionary=True)
        
        # Câu lệnh truy vấn lấy ra một bản ghi có dữ liệu mới nhất trong bảng
        cursor.execute("SELECT value, timestamp FROM gold_price ORDER BY id DESC LIMIT 1")
        result = cursor.fetchone()
        
        cursor.close()
        conn.close()
        
        if result:
            # Chuyển đổi định dạng datetime sang chuỗi để tránh lỗi JSON serialize
            if isinstance(result['timestamp'], datetime):
                result['timestamp'] = result['timestamp'].strftime('%Y-%m-%d %H:%M:%S')
            return jsonify(result), 200
        return jsonify({"value": 0, "timestamp": "Chưa có dữ liệu"}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
 ```


## 4. Thành phần Giao diện Ứng dụng Frontend (Thư mục ./frontend)File frontend/index.htmlHTML<!DOCTYPE html>
 ```
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Hệ Thống Giám Sát Dữ Liệu Realtime</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f5f6fa; margin: 0; padding: 25px; text-align: center; }
        .wrapper { max-width: 1200px; margin: 0 auto; }
        .card-realtime { background: #ffffff; padding: 30px; border-radius: 14px; box-shadow: 0 4px 20px rgba(0,0,0,0.06); display: inline-block; min-width: 320px; margin-bottom: 30px; }
        .value-display { font-size: 46px; color: #e74c3c; font-weight: bold; margin: 15px 0; }
        .time-display { color: #7f8c8d; font-size: 14px; font-style: italic; }
        .chart-section { background: #ffffff; padding: 15px; border-radius: 14px; box-shadow: 0 4px 20px rgba(0,0,0,0.06); }
        iframe { width: 100%; height: 520px; border: none; border-radius: 8px; }
    </style>
</head>
<body>
    <div class="wrapper">
        <h2>HỆ THỐNG GIÁM SÁT REALTIME & TRỰC QUAN HÓA LỊCH SỬ BIẾN ĐỘNG</h2>
        
        <div class="card-realtime">
            <h3>Giá Trị Tức Thời (MariaDB)</h3>
            <div id="data-value" class="value-display">0.0</div>
            <div class="time-display">Thời gian nhận: <span id="data-time">--:--:--</span></div>
        </div>

        <div class="chart-section">
            <h3>Biểu Đồ Xu Hướng Lịch Sử (InfluxDB via Grafana)</h3>
            <iframe src="http://localhost:3000/d-solo/monitor_panel_id/realtime-monitor?orgId=1&refresh=5s&panelId=1"></iframe>
        </div>
    </div>

    <script>
        function fetchRealtimeData() {
            // Thực hiện gọi API bất đồng bộ đến Flask API ở cổng 5000
            fetch('http://localhost:5000/api/realtime')
                .then(response => response.json())
                .then(data => {
                    if(data.value !== undefined) {
                        document.getElementById('data-value').innerText = data.value + " USD";
                        document.getElementById('data-time').innerText = data.timestamp;
                    }
                })
                .catch(err => console.error("Lỗi kết nối tới hệ thống API:", err));
        }
        // Tự động kích hoạt lấy số liệu mới liên tục sau chu kỳ thời gian ngắn (2 giây)
        setInterval(fetchRealtimeData, 2000);
        fetchRealtimeData();
    </script>
</body>
</html>
 ```

<img width="1921" height="1080" alt="realtime" src="https://github.com/user-attachments/assets/96a076a4-2e03-4f24-8eb4-784ae9ce5fd4" />

Vì dữ liệu giá vàng chưa vượt ngưỡng thông báo về telegram nên em thực hiện cách test nhỏ 

<img width="1921" height="1080" alt="giavang" src="https://github.com/user-attachments/assets/beb4e0ea-a214-4436-b923-e4f0a5dea9c6" />

<img width="1921" height="1080" alt="canhbao" src="https://github.com/user-attachments/assets/34efecf9-2b57-4e8c-8025-0e58c66885ce" />

## CHƯƠNG III: QUY TRÌNH ĐÓNG GÓI VÀ KHÔI PHỤC HỆ THỐNG TRONG MÔI TRƯỜNG OFFLINE
Để đáp ứng yêu cầu kiểm thử tính cơ động của ứng dụng khi triển khai trên hệ thống máy chủ vật lý bị cô lập hoàn toàn, không có kết nối Internet, kịch bản đóng gói và khôi phục được triển khai và minh chứng qua các bước:

# 1. Trích xuất toàn bộ Container hệ thống thành file nén vật lý .tar

Mô tả chi tiết: Thực hiện gom cụm tất cả các Docker Image của 5 dịch vụ thành phần trong dự án và nén chúng lại thành một tệp lưu trữ vật lý duy nhất để có thể di chuyển qua USB sang môi trường ngoại mạng Offline.

Lệnh thực hiện:

 ``` Bash
  docker save -o backup_monitor_system.tar mariadb:10.11 influxdb:2.7 nodered/node-red:latest nginx:alpine monitor_flask_api:latest
 ```
<img width="1903" height="1012" alt="image" src="https://github.com/user-attachments/assets/c124d93d-0932-4370-b13b-301cb5177303" />


# 2. Gỡ bỏ hoàn toàn mọi Container và Image cục bộ để giả lập máy chủ trống
Mô tả chi tiết: Sử dụng lệnh dừng toàn bộ hệ thống dịch vụ, sau đó tiến hành gỡ bỏ (xóa sạch) tất cả các container cũng như image đã lưu trong bộ nhớ máy host để đưa máy tính quay về trạng thái hoàn toàn trống rỗng (giả lập một máy chủ vật lý mới chưa từng được cài đặt phần mềm).

Lệnh thực hiện:

 ```Bash
  docker-compose down
  docker rmi $(docker images -q) --force
 ```

<img width="1470" height="702" alt="2 1" src="https://github.com/user-attachments/assets/3d270380-2601-452c-b492-12513e3f181a" />

<img width="1475" height="705" alt="check" src="https://github.com/user-attachments/assets/46370f29-5c69-4d86-a9d3-eda35fe7a93b" />

# 3. Nạp dữ liệu khôi phục hệ thống từ file nén vật lý không cần Internet

Mô tả chi tiết: Sử dụng lệnh docker load trích xuất ngược lại tệp tin .tar vào Docker Engine máy host (mô phỏng quá trình cắm USB cài đặt cho máy chủ Offline). Sau khi nạp thành công toàn bộ image, thực hiện lệnh Compose khởi chạy lại toàn hệ thống để chứng minh ứng dụng phục hồi trạng thái hoạt động bình thường, ổn định.

Lệnh thực hiện:

 ``` Bash
  docker load -i backup_monitor_system.tar
  docker-compose up -d
 ```
<img width="1022" height="362" alt="image" src="https://github.com/user-attachments/assets/ccc9b1c6-979e-4fd1-8b68-e90e5c6ae8b2" />

