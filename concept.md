### ZoneMTA (internal code name X-699)
- Chuyển tiếp SMTP ra modern (MTA/MSA) được xây dựng trên Node.js và MongoDB (queue storage)
- Nó giống như Postfix cho outbound nhưng có thể sử dụng nhiều địa chỉ local IP và có thể dễ dàng mở rộng bằng cách sử dụng plugins
- ZoneMTA cung cấp kiểm soát chi tiết đối với các định tuyết thông báo khác nhau
- Người gửi đáng tin cậy có thể định tuyến thông qua vùng gửi ảo tốc độ cao (kết nối song song hơn) sử dụng địa chỉ IP uy tín hơn. Nhiều người gửi đáng tin cậy có thể định tuyến thông qua vùng gửi ảo chậm hơn (ít kết nối hơn) hoặc thông qua địa chỉ IP với ít danh tiếng hơn. 
- Ngoài ra server còn có nhiều tính năng phổ biến hơn phần mềm thương mại, như viết lại message, khởi động IP hoặc HTTP API cho post message
- ZoneMTA có thể so sánh với Haraka nhưng không như Haraka, nó chỉ dành cho outbound. Cả hệ thống chạy trên Node.js và có một hệ thống plugin tích hợp mặc dù các thiết kế có phần khác nhau. Hệ thống plugin (và nhiều hơn nữa là tốt) cho ZoneMTA thì kế thừa từ dự án Nodemailer và do đó không có quan hệ trực tiếp với Haraka
- Ngoài ra còn có giao diện quản trị dựa trên web (cần được cài đặt riêng)
### Upgrade notes
- ZoneMTA version 1.1 sử dụng một ứng dụng khác cơ chế config 1.0
- Ngoài ra, không còn ứng dụng dòng lệnh nào nữa, bạn cần bao gồm nó như một module
### Requirements
- Node.js v8.0.0+ cho việc run app
- MongoDB cho lưu trữ message trong queue
- Redis cho locking và counter
### Quickstart
- Giả sử Node.js (v8.0.0+), MongoDB chạy trên localhost và git. Không được nghe gì trên port 2525 (SMTP), 12080 (HTTP API) và 12081 (kênh dữ liệu nội bộ). Tất cả các port này đều có thể config được.
#### Tạo ứng dụng Zone MTA
- Tìm nạp template app ZoneMTA 
```
$ git clone git://github.com/zone-eu/zone-mta-template.git
$ cd zone-mta-template
$ npm install eslint --save-dev
$ npm init
$ npm install --production
$ npm start
```
- Nếu mọi thứ thành công, bạn nên có một SMTP relay với không xác thực chạy trên localhost port 2525 (không chấp nhận connect remote).
- Tiếp đó bạn có thể cố gắng install và config một plugin bổ sung hoặc chỉnh sửa cấu hình mặc định trong config folder
- Bảng điều khiển admin web nên được cài đặt riêng, nó không phải một phần của cài đặt mặc định
### Birds-eye-view of the system
#### 1. Incoming message pipeline
- Message bị drop để gửi bằng SMTP hoặc HTTP API. 
- Message được xử lý dưới dạng luồng, vì vậy sẽ không thành vấn đề nếu message có kích thước rất lớn (chấp nhận nếu một message rất lớn thì gửi sử dụng JSON API). 
- Ứng dụng này cũng áp dụng cho tính hàm băm cơ chế DKIM - hàm băm được tính theo từng đoạn khi luồng message chạy qua (chữ ký thực tế được tạo ra từ cơ chế hàm băm khi gửi message đến đích). 
- Luồng incoming bắt đầu từ incoming connection và kết thúc trong MongoDB DridFS, vì vậy nếu có một error trong step nào giữa cả 2, error là report lại cho client và message bị từ chối
- Nếu dữ liệu khách quan được lưu trữ trong GridFS nó sẽ được thu gom rác sau một thời gian (tất cả thân message không tham chiếu đến delivery row sẽ được xóa tự động

![](https://i.imgur.com/30G3aJB.png)

#### 2. Outgoing message pipeline
- Gửi message tới điểm đến (this image này đã lỗi thời)

![](https://i.imgur.com/YLRwH1c.png)

### Features
- Giao diện web. Xem trạng thái queue và debug các message bị trì hoãn thông qua giao diện web dễ sử dụng (cần phải được cài đặt một cách riêng biệt)
- cross platform. Bạn có thể run ZoneMTA thậm chí trên Windows
- Nhanh. Gửi hàng triệu tin nhắn mỗi ngày
- Tổng hợp kết nối
- Gửi message với chi phí thấp
- Auto đăng kí DKIM
- Thêm id message và Date header nếu thiếu
- Hỗ trợ vùng gửi: gửi message khác nhau sử dụng địa chỉ IP khác nhau
- Hỗ trợ tích hợp cho message bị trì hoãn: Chỉ sử dụng một giá trị tiện ích trong Date header và message thì không gửi ra ngoài trước thời điểm đó
- Chỉ định doamin nhận cụ thể cho vùng gửi cụ thể
- Queue lưu trữ trên MongoDB
- Được xây dựng hỗ trợ IPv6
- Report cho Prometheus
- Sử dụng STARTTLS cho outgoing message theo mặc định, vì vậy không có hình ảnh broken padlock trong Gmail
- Xử lý bounce thông minh hơn
- Điều tiết trên mỗi kết nối vùng gửi
- Phát hiện thư rác bằng Rspamd
- HTTP API để gửi message
- Plugin tùy chỉnh
- Auto back-off nếu một địa chỉ IP trên blacklist
- Email Address Internationalization (EAI) và tiện ích SMTPUTF8. Gửi mail tới địa chỉ unicode như андрис@уайлддак.орг
- Gửi tới HTTP sử dụng POST thay vì SMTP
#### Configuration
- Cấu hình mặc định có thể tìm thấy từ https://github.com/zone-eu/zone-mta/blob/master/config/default.js 
- Bạn có thể ghi đè các tùy chọn trong cấu hình ứng dụng cụ thể của bạn nhưng bạn không cần chỉ định những giá trị mà bạn muốn giữ mặc định
### Fratures
#### 1. Hỗ trợ message lớn
- Tất cả dữ liệu được xử lý theo khối mà không cần đọc toàn bộ message vào memory, vì vậy nó không quan trọng nếu message là 1kB hoặc 1GB.
####  2. DKIM signing
- Hỗ trợ đăng kí DKIM được xây dựng trên ZoneMTA 
- Bạn có thể cung cấp khóa DKIM sử dụng để xây dựng trên DKIM plugin hoặc cách khác là tạo plugin của riêng bạn để xử lý key manaement
- ZoneMTA tính toán tất cả hash cần thiết và có thể ký message nếu một key hoặc nhiều key được cung cấp
#### 3. Sending Zone
