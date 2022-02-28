# 1. Tổng quan

- Telegraf – Influxdb – Grafana (TIG)  là một stack dùng để giám sát hệ thống, công cụ rất phổ biến.

- Telegraf và Influxdb đều là sản phẩm mã nguồn mở của InfluxData. Mặc dù InfluxData cung cấp một stack hoàn chỉnh để giám sát với Chronograf để visualize và Kapacitor để cảnh báo (TICK stack) nhưng chúng ta có thể sử dụng Grafana để thay thế cho cả Chronograf lẫn Kapacitor.

- Telegraf là một agent để thu thập và báo cáo các số liệu và dữ liệu. Nó có thể tích hợp để thu thập nhiều loại nguồn dữ liệu khác nhau của các số liệu, events, và logs trực tiếp từ các container và hệ thống mà nó chạy trên đó.

- InfluxDB là một Time Series Database (là database được tối ưu hóa để xử lý dữ liệu chuỗi thời gian, các dãy số được lập chỉ mục theo thời gian). Nó cung cấp một ngôn ngữ giống SQL để tương tác với dữ liệu.

- Grafana là một công cụ mã nguồn mở cho phép chúng ta truy vấn, hiển thị, cảnh báo các số liệu thu thập được. Nó có các tính năng nổi bật như hiển thị số liệu theo nhiều dạng biểu đồ khác nhau; cảnh báo qua nhiều kênh như email, telegram, slack, …; Hỗ trợ nhiều loại database như InfluxDB, ElasticSearch, Graphite, …; Cung cấp nhiều plugin, nhiều dashboard template.

  <img src="https://github.com/lean15998/TIG-Stack/blob/main/image/99.png">
- Nói ngắn gọn, TIG stack hoạt động như sau:

<ul>
<ul>
<li> Telegraf là 1 Agent dùng để thu thập số liệu hệ thống cũng như dịch vụ trên hệ thống mà nó chạy.
<li> Các số liệu được thu thập và đưa vào InfluxDB.
<li> Từ InfluxDB, dữ liệu sẽ được đẩy đến Grafana và được hiển thị theo các dạng đồ thị trực quan.
</ul>
</ul>

# 2. Cài đặt

## Mô hình triển khai

                                   | eth0
                        +----------------------------+
                        |                            |                           
                        |10.0.0.51                   |10.0.0.52                  
            +-----------+-----------+    +-----------+-----------+   
            |         [quynv]       |    |         [agent]       |   
            |         InfluxDB      +----+        Telegraf       |
            |         Grafana       |    |                       |   
            |                       |    |                       |  
            |                       |    |                       |   
            |                       |    |                       | 
            +-----------+-----------+    +-----------------------+


### Cài đặt InfluxDB

- Add repo influxdata

```sh
root@quynv:~# wget -qO- https://repos.influxdata.com/influxdb.key | apt-key add -
OK
root@quynv:~# source /etc/lsb-release
root@quynv:~# echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | tee /etc/apt/sources.list.d/influxdb.list
deb https://repos.influxdata.com/ubuntu focal stable
root@quynv:~# apt update 
```

- Cài đặt influxdb

```sh
root@quynv:~# apt install influxdb -y
```

- Khởi động dịch vụ và cấu hình khởi động cùng hệ thống:


```sh
root@quynv:~# systemctl start influxdb
root@quynv:~# systemctl enable influxdb
```

- Mở port cho influxdb

```sh
root@quynv:~# ufw allow 8086
Rules updated
Rules updated (v6)
root@quynv:~# ufw allow 8088
Rules updated
Rules updated (v6)
```

- Cấu hình influxdb

```sh
root@quynv:~# vi /etc/influxdb/influxdb.conf

[http]
# Determines whether HTTP endpoint is enabled.
enabled = true

# Determines whether the Flux query endpoint is enabled.
flux-enabled = true

# The bind address used by the HTTP service.
bind-address = ":8086"

# Determines whether HTTP request logging is enabled.
log-enabled = true
```

- Tạo user chứng thực

```sh
root@quynv:~# influx
Connected to http://localhost:8086 version 1.8.10
InfluxDB shell version: 1.8.10
> create database telegraf
> create user telegraf with password 'lean15998' with all privileges
> exit
```

### Cài đặt telegaf agent


- Add repo influxdata

```sh
root@agent:~# wget -qO- https://repos.influxdata.com/influxdb.key | apt-key add -
OK
root@agent:~# source /etc/lsb-release
root@agent:~# echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | tee /etc/apt/sources.list.d/influxdb.list
deb https://repos.influxdata.com/ubuntu focal stable
root@agent:~# apt update 
```

- Cài đặt influxdb

```sh
root@agent:~# apt install influxdb-client -y
```

- Cài đặt telegraf

```sh
root@agent:~# wget https://dl.influxdata.com/telegraf/releases/telegraf_1.21.4-1_amd64.deb
root@agent:~# dpkg -i telegraf_1.21.4-1_amd64.deb
```
- Khởi động dịch vụ và cấu hình khởi động cùng hệ thống

```sh
root@agent:~# systemctl start telegraf
root@agent:~# systemctl enable telegraf
```


- Cấu hình telegraf

```sh
root@agent:~# vim /etc/telegraf/telegraf.conf

hostname = "agent"
[[outputs.influxdb]] 
urls = ["http://10.0.0.51:8086"]
database = "telegraf"
username = "telegraf"
password = "lean15998"

```
<ul>
  <ul>
<li> urls: ta chọn phương thức đẩy dữ liệu, có thể qua socket (chỉ dùng khi telegraf và influxdb trên cùng 1 host), qua giao thức upd, hoặc sử dụng http (khuyến khích dùng vì influxdb hỗ trợ tốt hơn với api http)
<li> database: Ta sẽ định nghĩa tên database dùng để lưu trữ dữ liệu trên influxdb, khi telegraf gửi dữ liệu tới nếu chưa có database nó sẽ tự tạo ra database với tên này.
<li> username và password: Nhập user và pass đã tạo ở influxdb để telegraf dùng nó và chứng thực.
  </ul>
  </ul>
  
- Kiểm tra kết nối đến db

```sh
root@agent:~# influx -host 10.0.0.51 -username 'telegraf' -password "lean15998"
Connected to http://10.0.0.51:8086 version 1.8.10
InfluxDB shell version: 1.6.4
> show databases
name: databases
name
----
_internal
telegraf
> show users
user     admin
----     -----
telegraf true
> exit
```
  
### Cài đặt Grafana

- Add repo và cài đặt grafana

```sh
root@quynv:~# wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
OK
root@quynv:~# add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
root@quynv:~# apt-get install -y grafana
```
  
- Khởi động dịch vụ và cấu hình khởi động cùng hệ thống  
  
```sh
root@quynv:~# systemctl start grafana-server
root@quynv:~# systemctl enable grafana-server
```
  
- Mở port cho grafana

```sh
root@quynv:~# ufw allow 3000
Rules updated
Rules updated (v6)
```

# Sử dụng dashboard grafana


### Truy cập vào dashboard `http://10.0.0.51:3000`

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/01.png">

### Thêm fluxdb làm nguồn dữ liệu cho grafana

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/02.png">

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/03.png">

- Nhập các thông tin cảu fluxdb bao gồm

<ul>
  <ul>
    <li> Tên InfluxDB
    <li> Đường dẫn truy cập InfluxDB(vì cài cùng trên 1 server với grafana nên là localhost)
    <li> Tài khoản xác thực. Nhập tài khoản admin của InfluxDB và mật khẩu
    <li> Database detail ta nhập user và password của telegraf
      </ul>
  </ul>

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/04.png">
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/05.png">
  
  
### Thêm dashboad giám sát (Chọn add new panel)

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/06.png">

- Thêm các thông số query dứ liệu từ influxdb và chọn dạng đồ thị
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/07.png">
- Kết quả
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/08.png">

### Import dashboard
- Import Dashboard số 1902
 <img src="https://github.com/lean15998/TIG-Stack/blob/main/image/97.PNG">
- Chọn thư mục và Nguồn dữ liêu
 <img src="https://github.com/lean15998/TIG-Stack/blob/main/image/96.PNG">
- Kết quả
 <img src="https://github.com/lean15998/TIG-Stack/blob/main/image/98.PNG">
### Alert notification

- Cấu hình smtp cho grafana

```sh
root@quynv:~# vim /etc/grafana/grafana.ini

[smtp]

enabled = true
host = smtp.gmail.com:587
user = quy15091998@gmail.com
password = **********
from_address = quy15091998@gmail.com
from_name = Grafana

root@quynv:~# systemctl restart grafana-server

```

- Thêm contact point

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/14.png">

- Thêm notification policy

<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/15.png">


- Tạo rule cảnh báo
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/09.png">
- Nhập các thông tin cảnh báo ram sử dụng vượt quá 125400 Mb
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/10.png">
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/11.png">
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/12.png">

- Kết quả
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/13.png">

- Lịch sử trạng thái
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/17.png">
- Kiểm tra mail cảnh báo
<img src="https://github.com/lean15998/TIG-Stack/blob/main/image/16.png">
