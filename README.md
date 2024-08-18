Giới thiệu về Hadoop
user
hadoop là gì ?
 
Hadoop là một nền tảng mã nguồn mở được phát triển bởi Apache dùng để xử lý và lưu trữ một lượng dữ liệu lớn theo mô hình phân tán. Nó có khả năng mở rộng, xử lý dữ liệu theo lô, và thường được sử dụng trong các môi trường Big Data.

### Các thành phần chính của Hadoop:

1. **Hadoop Distributed File System (HDFS)**: Hệ thống tệp phân tán của Hadoop, giúp lưu trữ dữ liệu lớn trên nhiều máy tính khác nhau.

2. **MapReduce**: Mô hình lập trình dùng để xử lý và phân tích dữ liệu lớn, chia quá trình xử lý thành hai giai đoạn chính: "Map" và "Reduce".

3. **YARN (Yet Another Resource Negotiator)**: Quản lý và phân phối tài nguyên trên hệ thống Hadoop.

4. **Hadoop Common**: Gồm các thư viện và tiện ích hỗ trợ các mô-đun khác của Hadoop.

### Ứng dụng của Hadoop
Hadoop thường được sử dụng trong các hệ thống xử lý dữ liệu lớn như phân tích dữ liệu, khai phá dữ liệu, và các ứng dụng liên quan đến dữ liệu phi cấu trúc, như mạng xã hội, log file, hay dữ liệu từ các thiết bị IoT.

Vì tính linh hoạt và khả năng mở rộng cao, Hadoop được sử dụng phổ biến trong nhiều tổ chức và doanh nghiệp xử lý Big Data.
user
Viết file docker compose xây dựng Hadoop kết nối với HUE
 
Để xây dựng một cụm Hadoop và kết nối với HUE sử dụng Docker Compose, bạn có thể sử dụng tệp cấu hình `docker-compose.yml` dưới đây. Cấu hình này sẽ bao gồm:

1. **Hadoop (HDFS, YARN)**: Xử lý và lưu trữ dữ liệu lớn.
2. **Hue**: Giao diện web giúp tương tác và quản lý Hadoop dễ dàng hơn.

Dưới đây là một mẫu tệp `docker-compose.yml`:

```yaml
version: '3.8'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    hostname: namenode
    ports:
      - "9870:9870" # HDFS Web UI
      - "9000:9000"
    volumes:
      - namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=hadoop-cluster
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
      - CORE_CONF_hadoop_http_staticuser_user=root
      - HDFS_CONF_dfs_namenode_name_dir=/hadoop/dfs/name
      - HDFS_CONF_dfs_replication=1
      - HDFS_CONF_dfs_permissions_enabled=false
    networks:
      - hadoop

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    hostname: datanode
    ports:
      - "9864:9864" # DataNode Web UI
    volumes:
      - datanode:/hadoop/dfs/data
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
      - CORE_CONF_hadoop_http_staticuser_user=root
      - HDFS_CONF_dfs_datanode_data_dir=/hadoop/dfs/data
      - HDFS_CONF_dfs_replication=1
      - HDFS_CONF_dfs_permissions_enabled=false
    networks:
      - hadoop
    depends_on:
      - namenode

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    hostname: resourcemanager
    ports:
      - "8088:8088" # YARN Web UI
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
      - YARN_CONF_yarn_resourcemanager_hostname=resourcemanager
      - YARN_CONF_yarn_scheduler_minimum_allocation_mb=128
      - YARN_CONF_yarn_scheduler_maximum_allocation_mb=8192
      - YARN_CONF_yarn_scheduler_minimum_allocation_vcores=1
      - YARN_CONF_yarn_scheduler_maximum_allocation_vcores=4
    networks:
      - hadoop
    depends_on:
      - namenode

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    hostname: nodemanager
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    networks:
      - hadoop
    depends_on:
      - resourcemanager

  historyserver:
    image: bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8
    container_name: historyserver
    hostname: historyserver
    ports:
      - "8188:8188" # MapReduce JobHistory Web UI
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    networks:
      - hadoop
    depends_on:
      - namenode
      - resourcemanager

  hue:
    image: gethue/hue:latest
    container_name: hue
    hostname: hue
    ports:
      - "8888:8888" # Hue Web UI
    environment:
      - HUE_CONF_hue_safetybelt=false
      - HUE_CONF_hadoop_hdfs_url=hdfs://namenode:9000
      - HUE_CONF_hadoop_yarn_url=http://resourcemanager:8088
    networks:
      - hadoop
    depends_on:
      - namenode
      - resourcemanager

volumes:
  namenode:
  datanode:

networks:
  hadoop:
    driver: bridge
```

### Hướng dẫn triển khai:

1. Tạo tệp `docker-compose.yml` với nội dung như trên.
2. Chạy lệnh sau để khởi động các container:

   ```bash
   docker-compose up -d
   ```

3. Bạn có thể truy cập các dịch vụ qua các địa chỉ sau:
   - **Hue UI**: `http://localhost:8888`
   - **HDFS UI**: `http://localhost:9870`
   - **YARN UI**: `http://localhost:8088`

Hue cho phép bạn quản lý file HDFS, chạy các câu lệnh Hive, Pig, hoặc tương tác với YARN dễ dàng thông qua giao diện web.
user
user login HUE là gì ?
 
Trong môi trường mặc định khi cài đặt Hue, thông tin đăng nhập ban đầu thường là tài khoản **`hue`** với mật khẩu **`hue`** hoặc đôi khi không có mật khẩu (để trống). Tuy nhiên, điều này có thể thay đổi tùy thuộc vào cấu hình và cách triển khai cụ thể.

Nếu bạn chưa thiết lập người dùng cụ thể và không có tài khoản đăng nhập, bạn có thể tạo một tài khoản quản trị bằng cách chạy lệnh sau trong container Hue:

```bash
docker exec -it hue bash
```

Sau khi vào bên trong container:

```bash
hue createsuperuser
```

Lệnh này sẽ yêu cầu bạn tạo tên người dùng, địa chỉ email, và mật khẩu cho tài khoản quản trị viên. Sau đó, bạn có thể sử dụng tài khoản này để đăng nhập vào Hue.

Nếu bạn sử dụng cấu hình mặc định và gặp khó khăn khi đăng nhập, hãy kiểm tra lại tệp cấu hình hoặc tài liệu chính thức của Hue.
user
bỏ data vào hadoop như thế nào ?
 
Để đưa dữ liệu vào Hadoop (HDFS), bạn có thể thực hiện các bước sau bằng cách sử dụng lệnh `hdfs dfs` trong terminal. Dưới đây là các bước cơ bản:

### 1. **Xác minh HDFS đang chạy**

Trước khi tải dữ liệu lên HDFS, hãy đảm bảo rằng HDFS đã sẵn sàng và đang chạy. Bạn có thể kiểm tra trạng thái qua giao diện web tại `http://localhost:9870` (nếu dùng Docker) hoặc sử dụng lệnh:

```bash
hdfs dfsadmin -report
```

### 2. **Tạo thư mục trong HDFS (nếu cần)**

Bạn có thể tạo thư mục mới trong HDFS để tổ chức dữ liệu:

```bash
hdfs dfs -mkdir /user/tendangnhap/mydata
```

Ví dụ trên tạo thư mục `/user/tendangnhap/mydata` trong HDFS.

### 3. **Tải dữ liệu từ máy cục bộ lên HDFS**

Sử dụng lệnh sau để đưa một file từ máy cục bộ lên HDFS:

```bash
hdfs dfs -put /path/to/local/file.txt /user/tendangnhap/mydata/
```

Ví dụ:

```bash
hdfs dfs -put /home/user/data/sample.txt /user/tendangnhap/mydata/
```

### 4. **Xem dữ liệu trong HDFS**

Để xem danh sách file và thư mục trong HDFS, dùng lệnh:

```bash
hdfs dfs -ls /user/tendangnhap/mydata/
```

### 5. **Kiểm tra nội dung file trong HDFS**

Bạn có thể kiểm tra nội dung của file đã tải lên:

```bash
hdfs dfs -cat /user/tendangnhap/mydata/sample.txt
```

### 6. **Tích hợp với Hue**

Nếu bạn sử dụng Hue, bạn có thể truy cập HDFS qua giao diện web của Hue. Chọn mục **File Browser** và điều hướng đến thư mục mà bạn đã tải dữ liệu lên. Tại đây, bạn có thể quản lý file và thư mục một cách trực quan.

### Lưu ý
- Đảm bảo rằng các quyền truy cập và thư mục trên HDFS phù hợp với người dùng mà bạn đang làm việc.
- Nếu bạn đang sử dụng cụm Hadoop trên Docker, bạn cần vào container `namenode` hoặc bất kỳ container Hadoop nào để chạy các lệnh trên.

Ví dụ, chạy lệnh sau để vào container `namenode`:

```bash
docker exec -it namenode bash
```
user
tạo mount trong docker compose để upload dữ liệu lên mà ko cần phải docker exec -it namenode bash
 
Để tránh việc phải vào container bằng lệnh `docker exec -it namenode bash`, bạn có thể sử dụng tính năng mount volume trong Docker Compose. Bằng cách này, bạn có thể map một thư mục trên máy cục bộ vào một thư mục trong container. Dữ liệu mà bạn muốn upload lên Hadoop có thể được lưu trong thư mục trên máy cục bộ và tự động xuất hiện trong container.

Dưới đây là cách chỉnh sửa tệp `docker-compose.yml` để mount một thư mục từ máy cục bộ vào container Hadoop:

### Ví dụ `docker-compose.yml`:

```yaml
version: '3.8'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    hostname: namenode
    ports:
      - "9870:9870"
      - "9000:9000"
    volumes:
      - namenode:/hadoop/dfs/name
      - ./data:/input # Mount thư mục local ./data vào /input trong container
    environment:
      - CLUSTER_NAME=hadoop-cluster
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    networks:
      - hadoop

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    hostname: datanode
    ports:
      - "9864:9864"
    volumes:
      - datanode:/hadoop/dfs/data
    environment:
      - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
    networks:
      - hadoop
    depends_on:
      - namenode

  # Các dịch vụ khác (resourcemanager, nodemanager, hue, v.v.)
  # ...

volumes:
  namenode:
  datanode:

networks:
  hadoop:
    driver: bridge
```

### Giải thích:

- **`./data:/input`**: Đây là mount volume. Thư mục `./data` trên máy cục bộ của bạn sẽ được mount vào thư mục `/input` trong container `namenode`.
- Bạn chỉ cần copy hoặc di chuyển dữ liệu vào thư mục `./data` trên máy cục bộ. Dữ liệu này sẽ tự động có sẵn trong container tại đường dẫn `/input`.

### Cách upload dữ liệu từ thư mục đã mount lên HDFS:

Khi thư mục đã được mount, bạn có thể sử dụng lệnh `hdfs dfs` để upload dữ liệu từ thư mục `/input` vào HDFS mà không cần vào container:

```bash
docker-compose exec namenode hdfs dfs -put /input/filename.txt /user/tendangnhap/mydata/
```

Ví dụ:

```bash
docker-compose exec namenode hdfs dfs -put /input/sample.txt /user/tendangnhap/mydata/
```

### Lợi ích:

- Bạn không cần phải truy cập vào container mỗi khi muốn thêm dữ liệu.
- Dữ liệu trong thư mục `./data` sẽ luôn được đồng bộ với container.

Với cách này, việc upload dữ liệu lên Hadoop sẽ trở nên dễ dàng và tiện lợi hơn rất nhiều.
user
nếu muốn đẩy dữ liệu của mysql vào thì làm như thế nào ?
 
Để đẩy dữ liệu từ MySQL vào Hadoop (HDFS), bạn có thể sử dụng công cụ **Apache Sqoop**, một công cụ phổ biến để nhập và xuất dữ liệu giữa các hệ thống cơ sở dữ liệu quan hệ (RDBMS) như MySQL và HDFS.

Dưới đây là các bước cụ thể:

### 1. **Cài đặt Apache Sqoop**
Nếu bạn chưa có Sqoop, bạn cần cài đặt nó trên hệ thống của mình hoặc tích hợp vào cụm Hadoop hiện tại.

### 2. **Chuẩn bị MySQL và kết nối JDBC**
Bạn cần một file JDBC driver cho MySQL. Tải về file `.jar` và đặt vào thư mục của Sqoop, chẳng hạn:

```bash
cp mysql-connector-java-8.0.32.jar /path/to/sqoop/lib/
```

### 3. **Chạy lệnh Sqoop để nhập dữ liệu vào HDFS**
Sqoop có thể import toàn bộ bảng hoặc chỉ một phần dữ liệu. Dưới đây là một ví dụ về cách import toàn bộ bảng từ MySQL vào HDFS:

```bash
sqoop import \
  --connect jdbc:mysql://localhost:3306/tendatabase \
  --username tennguoidung \
  --password matkhaunguoidung \
  --table tentable \
  --target-dir /user/tendangnhap/hdfs_directory \
  --num-mappers 1
```

### Giải thích các tùy chọn:
- **`--connect`**: Địa chỉ kết nối JDBC đến MySQL.
- **`--username`**: Tên người dùng MySQL.
- **`--password`**: Mật khẩu của MySQL.
- **`--table`**: Tên bảng cần import.
- **`--target-dir`**: Thư mục trong HDFS nơi dữ liệu sẽ được lưu trữ.
- **`--num-mappers`**: Số mapper sử dụng. Thông thường để đơn giản bạn có thể dùng `1`, nhưng nếu dữ liệu lớn, bạn có thể tăng số lượng mapper để cải thiện hiệu suất.

### 4. **Kiểm tra dữ liệu trong HDFS**
Sau khi import, bạn có thể kiểm tra dữ liệu đã được đẩy vào HDFS bằng lệnh:

```bash
hdfs dfs -ls /user/tendangnhap/hdfs_directory
```

### 5. **Tích hợp với Docker Compose (nếu cần)**
Nếu bạn sử dụng Docker Compose, bạn có thể tích hợp Sqoop vào cụm Hadoop. Ví dụ, thêm một service `sqoop` trong tệp `docker-compose.yml`:

```yaml
sqoop:
  image: bde2020/sqoop:latest
  container_name: sqoop
  environment:
    - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
  volumes:
    - ./lib/mysql-connector-java-8.0.32.jar:/opt/sqoop/lib/mysql-connector-java-8.0.32.jar
  networks:
    - hadoop
  depends_on:
    - namenode
    - datanode
```

Bạn có thể chạy lệnh Sqoop trực tiếp từ container:

```bash
docker-compose exec sqoop sqoop import \
  --connect jdbc:mysql://mysql:3306/tendatabase \
  --username tennguoidung \
  --password matkhaunguoidung \
  --table tentable \
  --target-dir /user/tendangnhap/hdfs_directory \
  --num-mappers 1
```

### 6. **Kết hợp với Hue để truy vấn dữ liệu**
Sau khi dữ liệu được nhập vào HDFS, bạn có thể sử dụng Hue để truy vấn hoặc quản lý dữ liệu này dễ dàng thông qua giao diện web.

Với các bước trên, bạn có thể dễ dàng chuyển dữ liệu từ MySQL vào HDFS và tận dụng Hadoop để xử lý dữ liệu lớn.
user
còn SQL server thì phải làm thế nào
 
Để đẩy dữ liệu từ SQL Server vào Hadoop (HDFS), bạn có thể sử dụng Apache Sqoop tương tự như khi làm với MySQL. Tuy nhiên, khi làm việc với SQL Server, có một số điểm khác biệt nhỏ cần lưu ý, chủ yếu liên quan đến việc kết nối JDBC.

### Các bước thực hiện:

### 1. **Chuẩn bị SQL Server JDBC Driver**
Bạn cần tải về driver JDBC cho SQL Server (thường là `mssql-jdbc-8.4.1.jre8.jar`) và đặt nó vào thư mục của Sqoop:

```bash
cp mssql-jdbc-8.4.1.jre8.jar /path/to/sqoop/lib/
```

### 2. **Cấu hình và sử dụng Sqoop để import dữ liệu từ SQL Server**

Lệnh Sqoop để import dữ liệu từ SQL Server sẽ có dạng như sau:

```bash
sqoop import \
  --connect "jdbc:sqlserver://localhost:1433;database=tendatabase" \
  --username tennguoidung \
  --password matkhaunguoidung \
  --table tentable \
  --target-dir /user/tendangnhap/hdfs_directory \
  --num-mappers 1
```

### Giải thích các tùy chọn:
- **`--connect`**: Chuỗi kết nối JDBC cho SQL Server. Cấu trúc thường là:  
  `jdbc:sqlserver://[địa chỉ SQL Server];database=[tên cơ sở dữ liệu]`.
- **`--username`**: Tên người dùng SQL Server.
- **`--password`**: Mật khẩu của SQL Server.
- **`--table`**: Tên bảng cần import.
- **`--target-dir`**: Thư mục trong HDFS nơi lưu trữ dữ liệu.
- **`--num-mappers`**: Số lượng mapper. Có thể bắt đầu với `1`, và tăng lên nếu cần cải thiện hiệu suất cho dữ liệu lớn.

### 3. **Kiểm tra dữ liệu trong HDFS**
Sau khi import, bạn có thể kiểm tra dữ liệu trong HDFS:

```bash
hdfs dfs -ls /user/tendangnhap/hdfs_directory
```

### 4. **Tích hợp với Docker Compose**

Nếu bạn sử dụng Docker Compose, bạn có thể thêm dịch vụ `sqoop` với SQL Server JDBC driver vào cụm Hadoop. Ví dụ:

```yaml
sqoop:
  image: bde2020/sqoop:latest
  container_name: sqoop
  environment:
    - CORE_CONF_fs_defaultFS=hdfs://namenode:9000
  volumes:
    - ./lib/mssql-jdbc-8.4.1.jre8.jar:/opt/sqoop/lib/mssql-jdbc-8.4.1.jre8.jar
  networks:
    - hadoop
  depends_on:
    - namenode
    - datanode
```

Sau đó, bạn có thể chạy lệnh Sqoop trực tiếp từ container:

```bash
docker-compose exec sqoop sqoop import \
  --connect "jdbc:sqlserver://sqlserver:1433;database=tendatabase" \
  --username tennguoidung \
  --password matkhaunguoidung \
  --table tentable \
  --target-dir /user/tendangnhap/hdfs_directory \
  --num-mappers 1
```

### 5. **Kết hợp với Hue**
Sau khi dữ liệu được nhập vào HDFS, bạn có thể sử dụng Hue để duyệt và truy vấn dữ liệu thông qua giao diện web.

### Lưu ý:
- Đảm bảo rằng cổng kết nối (`1433` mặc định) của SQL Server đã được mở và có thể truy cập từ hệ thống chạy Sqoop.
- Nếu bạn gặp lỗi liên quan đến kết nối JDBC, hãy kiểm tra lại chuỗi kết nối và các tham số liên quan như `instanceName`, `authentication`, v.v.

Với các bước này, bạn có thể dễ dàng import dữ liệu từ SQL Server vào HDFS và sử dụng Hadoop để xử lý dữ liệu lớn.
user
lấy dữ liệu từ các api thì làm cách nào ?
 
Để lấy dữ liệu từ các API và đưa vào HDFS, bạn có thể thực hiện qua các bước sau:

### 1. **Sử dụng Python hoặc một công cụ tương tự để gọi API**

Python là một lựa chọn phổ biến vì nó có các thư viện như `requests` để dễ dàng gọi API và xử lý dữ liệu.

### 2. **Viết mã Python để gọi API và lưu dữ liệu vào HDFS**

Bạn có thể sử dụng thư viện `hdfs` trong Python để tương tác với HDFS. Dưới đây là một ví dụ:

```python
import requests
from hdfs import InsecureClient

# Địa chỉ API
api_url = 'https://api.example.com/data'

# Gọi API và lấy dữ liệu
response = requests.get(api_url)
data = response.json()

# Kết nối đến HDFS
client = InsecureClient('http://namenode:9870', user='hadoop')

# Lưu dữ liệu vào HDFS
with client.write('/user/tendangnhap/api_data.json', encoding='utf-8') as writer:
    json.dump(data, writer)
```

### Giải thích mã:
- **`requests.get(api_url)`**: Gọi API và lấy dữ liệu.
- **`client = InsecureClient('http://namenode:9870', user='hadoop')`**: Kết nối đến HDFS.
- **`client.write('/user/tendangnhap/api_data.json', encoding='utf-8')`**: Ghi dữ liệu vào HDFS.

### 3. **Tích hợp với Docker Compose**

Nếu bạn chạy Hadoop qua Docker Compose, bạn có thể thêm một container Python để thực thi mã này. Ví dụ:

Trong `docker-compose.yml`:

```yaml
api-fetcher:
  image: python:3.9
  container_name: api-fetcher
  volumes:
    - ./scripts:/scripts  # Thư mục chứa script Python
  networks:
    - hadoop
  depends_on:
    - namenode
  command: python /scripts/fetch_api_data.py
```

### 4. **Lên lịch tự động (Optional)**
Bạn có thể sử dụng cron hoặc một job scheduler như Airflow để tự động hóa quá trình lấy dữ liệu từ API và lưu vào HDFS theo định kỳ.

### 5. **Kiểm tra dữ liệu trong HDFS**
Sau khi chạy script, bạn có thể kiểm tra dữ liệu đã được lưu trong HDFS:

```bash
hdfs dfs -ls /user/tendangnhap/api_data.json
```

### Các bước chi tiết:
- **Bước 1**: Viết script Python để lấy dữ liệu từ API.
- **Bước 2**: Chạy script và lưu dữ liệu vào HDFS.
- **Bước 3**: Tự động hóa và tích hợp với hệ thống hiện có nếu cần.

Cách này giúp bạn linh hoạt lấy dữ liệu từ bất kỳ API nào và đưa vào hệ thống Hadoop để xử lý và phân tích.
user
viết bằng javascript được không ?
 
Có thể sử dụng JavaScript (Node.js) để lấy dữ liệu từ API và lưu vào HDFS. Bạn có thể sử dụng các thư viện như `axios` để gọi API và `webhdfs` để tương tác với HDFS.

### Dưới đây là cách làm:

1. **Cài đặt Node.js và các thư viện cần thiết**

Đầu tiên, cài đặt Node.js và các gói cần thiết:

```bash
npm init -y
npm install axios webhdfs
```

2. **Viết mã Node.js để lấy dữ liệu từ API và lưu vào HDFS**

Dưới đây là một ví dụ mã:

```javascript
const axios = require('axios');
const WebHDFS = require('webhdfs');

// Cấu hình HDFS client
const hdfs = WebHDFS.createClient({
  user: 'hadoop', // Tên người dùng HDFS
  host: 'namenode', // Địa chỉ Namenode
  port: 9870, // Cổng WebHDFS
  path: '/webhdfs/v1'
});

// URL của API
const apiUrl = 'https://api.example.com/data';

// Gọi API và lấy dữ liệu
axios.get(apiUrl)
  .then(response => {
    const data = JSON.stringify(response.data);

    // Đường dẫn lưu trữ dữ liệu trên HDFS
    const remoteFilePath = '/user/tendangnhap/api_data.json';

    // Lưu dữ liệu vào HDFS
    const writeStream = hdfs.createWriteStream(remoteFilePath);

    writeStream.on('error', (err) => {
      console.error('Error writing to HDFS:', err);
    });

    writeStream.on('finish', () => {
      console.log('Data successfully written to HDFS');
    });

    // Ghi dữ liệu vào HDFS
    writeStream.write(data);
    writeStream.end();
  })
  .catch(error => {
    console.error('Error fetching API data:', error);
  });
```

### Giải thích mã:
- **`axios.get(apiUrl)`**: Gọi API và lấy dữ liệu JSON.
- **`WebHDFS.createClient({...})`**: Tạo một client để kết nối với HDFS qua WebHDFS.
- **`hdfs.createWriteStream(remoteFilePath)`**: Tạo một luồng ghi (write stream) vào HDFS.
- **`writeStream.write(data)`**: Ghi dữ liệu vào HDFS.

3. **Chạy mã Node.js**

Bạn có thể chạy mã này bằng lệnh:

```bash
node fetch_api_data.js
```

4. **Tích hợp với Docker Compose**

Nếu bạn chạy Hadoop qua Docker Compose, bạn có thể thêm một service Node.js vào:

Trong `docker-compose.yml`:

```yaml
api-fetcher:
  image: node:18
  container_name: api-fetcher
  volumes:
    - ./scripts:/scripts  # Thư mục chứa script JavaScript
  networks:
    - hadoop
  depends_on:
    - namenode
  working_dir: /scripts
  command: node fetch_api_data.js
```

Khi bạn khởi động cụm Docker Compose, script sẽ tự động chạy và lưu dữ liệu vào HDFS.

### Lưu ý:
- Đảm bảo rằng cổng WebHDFS (`9870`) đang mở và có thể truy cập từ container Node.js.
- Nếu bạn cần chạy script định kỳ, bạn có thể sử dụng cron hoặc tích hợp với các công cụ lập lịch như Airflow.

Với cách này, bạn có thể dễ dàng lấy dữ liệu từ bất kỳ API nào và lưu trữ trực tiếp vào HDFS thông qua Node.js.