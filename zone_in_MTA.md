## 1. Bounces
  - preferIPv6=false      <ưu tiên sử dụng IPv4 hơn IPv6>
  - ignoreIPv6=true       <không sử dụng IPv6 để liên lạc ngay cả khi có sẵn> => hữu ích trong trường hợp kết nối IPv6 không ổn định hoặc gây ra sự cố tương thích vì một số ứng dụng hoặc dịch vụ nhấr định
  - processes=1           <có bao nhiêu tiến trình worker>
  - connections=2         
  - pool="default"
  
## 2. Default
  - preferIPv6=false
  - ignoreIPv6=true
  - processes=1
  - connections=5
  - pool="default"

## 3. Route
  - preferIPv6=false
  - ignoreIPv6=true
  - processes=1
  - connections=5
  - pool = "default"
  - [routed.routingHeaders]
      "x-user-id" = "123"
  < xác định các giá trị tiêu đề cụ thể trong cấu hình bằng tùy chọn định tuyến Header>
