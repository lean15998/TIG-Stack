# 1. Tổng quan

- Telegraf – Influxdb – Grafana (TIG)  là một stack dùng để giám sát hệ thống, công cụ rất phổ biến.

- Telegraf và Influxdb đều là sản phẩm mã nguồn mở của InfluxData. Mặc dù InfluxData cung cấp một stack hoàn chỉnh để giám sát với Chronograf để visualize và Kapacitor để cảnh báo (TICK stack) nhưng chúng ta có thể sử dụng Grafana để thay thế cho cả Chronograf lẫn Kapacitor.

- Telegraf là một agent để thu thập và báo cáo các số liệu và dữ liệu. Nó có thể tích hợp để thu thập nhiều loại nguồn dữ liệu khác nhau của các số liệu, events, và logs trực tiếp từ các container và hệ thống mà nó chạy trên đó.

- InfluxDB là một Time Series Database (là database được tối ưu hóa để xử lý dữ liệu chuỗi thời gian, các dãy số được lập chỉ mục theo thời gian). Nó cung cấp một ngôn ngữ giống SQL để tương tác với dữ liệu.

- Grafana là một công cụ mã nguồn mở cho phép chúng ta truy vấn, hiển thị, cảnh báo các số liệu thu thập được. Nó có các tính năng nổi bật như hiển thị số liệu theo nhiều dạng biểu đồ khác nhau; cảnh báo qua nhiều kênh như email, telegram, slack, …; Hỗ trợ nhiều loại database như InfluxDB, ElasticSearch, Graphite, …; Cung cấp nhiều plugin, nhiều dashboard template.

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









