# 1. Giới thiệu
- Grafana là một công cụ mã nguồn mở cho phép chúng ta truy vấn, hiển thị, cảnh báo các số liệu thu thập được. Nó có các tính năng nổi bật như hiển thị số liệu theo nhiều dạng biểu đồ khác nhau; cảnh báo qua nhiều kênh như email, telegram, slack, …; Hỗ trợ nhiều loại database như InfluxDB, ElasticSearch, Graphite, …; Cung cấp nhiều plugin, nhiều dashboard template.

# 2. Cài đặt và sử dụng

### Cài đặt
https://grafana.com/docs/grafana/latest/installation/debian/

### Các thao tác cơ bản

#### Đăng nhập
- Mở trình duyệt web và truy cập vào `http://localhost:3000/` và đăng nhập với tài khoản `admin` mật khẩu `admin`.







# Alert notification

### Trạng thái và tình trạng của alert rule

- Alerting rule state
<ul>
  <ul>
  <li> Normal : Không có chuỗi thời gian nào được trả về ở trạng thái Pending hoặc Firing sate.
  <li> Pending : Ít nhất một chuỗi thời gian được trả về là Đang xử lý.
  <li> Firing : Ít nhất một chuỗi thời gian được trả về là kích hoạt.
</ul>
  </ul>
    
    
- Alert state
<ul>
  <ul>
  <li> Normal: Điều kiện cho alert rule là false đối với mọi chuỗi thời gian trả về.
  <li> Alerting: Điều kiện của alert rule là true với ít nhất một chuỗi thời gian được trả về. Khoảng thời gian mà điều kiện phải true trước khi kích hoạt alert.
  <li> Pending: Điều kiện của alert rule là true với ít nhất một chuỗi thời gian được trả về. Khoảng thời gian mà điều kiện phải true trước khi kích hoạt alert.
    <li> NoData : Alert rule không trả về một chuỗi thời gian, tất cả các giá trị cho chuỗi thời gian là rỗng hoặc bằng 0.
    <li> Error : Lỗi khi cố gắng đánh giá alert rule.
</ul>
  </ul>


- Alerting rule health

<ul>
  <ul>
  <li> Ok : Không có lỗi khi đánh giá alert rule.
  <li> Error : Lỗi khi đánh giá alert rule.
  <li> NoData : Không tồn tại data trong ít nhất một chuỗi thời gian được trả về trong quá trình alert rule.
</ul>
  </ul>


### Tạo và quản lý alert rule Grafana

- Add Grafana managed rule





