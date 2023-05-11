### Tạo ứng dụng ZoneMTA
```
$ git clone https://github.com/zone-eu/zone-mta-template.git
$ cd zone-mta-template
$ npm install eslint --save-dev
$ npm init
$ npm install --production
$ npm start
```
#### Nodejs
- Kiểm tra version của nodejs (version 10.19.0 không tương thích) 
- Xóa nodejs cũ
```
sudo apt-get remove nodejs
sudo apt-get purge nodejs
sudo apt-get autoremove
```
- Lệnh cài nodejs
```
sudo apt update
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
cat /etc/apt/sources.list.d/nodesource.list (tạo kho lưu trữ)
sudo apt -y install nodejs
```
- Kiểm tra version 
```
node -v
```
#### Mongodb
- Cài đặt mongodb
```
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
apt-key list (lệnh kiểm tra khóa)
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list (tạo tệp `sources.list.d` trong thư mục `mongodb-org-4.4.list`)
sudo apt update
sudo apt install mongodb-org
```
- Bắt đầu dịch vụ mongodb và kiểm tra cơ sở dữ liệu
```
sudo systemctl start mongod.service       # bắt đầu dịch vụ mongodb
sudo systemctl status mongod              # kiểm tra trạng thái dịch vụ
```
![](https://i.imgur.com/JDbZoor.png)
```
sudo systemctl enable mongod              
mongo --eval 'db.runCommand({ connectionStatus: 1 })'         # xác minh cơ sở dữ liệu đang hoạt động
```
![](https://i.imgur.com/0nVkvfn.png)

##### tạo use/password
###### tạo một user admin
mongo

```
use admin
db.createUser({
        user: "admin",
        pwd: "xTK9a37fqDVvDXY8sCTbwqQ8",
        roles: [{role: "userAdminAnyDatabase" , db: "admin"}]
});

use zone-mta
db.createUser({
        user: "zonemta",
        pwd: "emJQ5bQqAw9SrV5r9cKQzTjd",
        roles: [{role: "userAdmin" , db: "zone-mta"},
                { role: "readWrite", db: "test" }]
});

use zone-mta
db.updateUser("zonemta", {roles: [{ role : "userAdmin", db : "zone-mta" }, {role: "readWrite", db: "zone-mta"}]})
```
exit

mongo ip -u zonemta  -p emJQ5bQqAw9SrV5r9cKQzTjd --authenticationDatabase zone-mta
#### Redis
- install redis
```
sudo apt update
sudo apt install redis-server
```
- config redis
  - Để quản lý Redis như một dịch vụ, đặt supervised directive thành `systemd`
```
sudo vim /etc/redis/redis.conf
```
![](https://i.imgur.com/oNMc7UF.png)

``` 
sudo systemctl restart redis.service
```
- check redis service status
```
sudo systemctl status redis
```
![](https://i.imgur.com/KlOexTE.png)

```
sudo systemctl enable redis-server
```
#### ZoneMTA
```
npm install eslint --save-dev
npm init
npm install --production
npm start
npm run config
```
### Config and run as a service

Tạo file `/etc/systemctl/system/zonemta.service`

```
[Unit]
Description=Zone Mail Transport Agent
Conflicts=sendmail.service exim.service postfix.service
After=mongod.service redis.service

[Service]
Environment="NODE_ENV=production"
WorkingDirectory=/opt/zone-mta-template/
ExecStart=/usr/bin/node --max-old-space-size=2048 index.js --config="config/zonemta.toml"
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
Restart=always

[Install]
WantedBy=multi-user.target
```
- Start service

```
systemctl restart zonemta
```
- file `config/interfaces/feeder.toml`, interface và port listen cho smtp, tắt starttls, enable authentication

```
[feeder]
enabled=true
processes=10
maxSize=20971520
host="0.0.0.0"
port=25
authentication=true
maxRecipients=1000
starttls=false
secure=false
```
- Cấu hình authen cho mỗi lần relay mail đến, thêm user/password vào file `config/zonemta.toml`

```
name="BizflyCloudZone"
ident="zone-mta"

[api]
port=12080
host="0.0.0.0"
user="..."
pass="..."
```
- file `config/plugins/http-auth.toml`

```
["core/http-auth"]
enabled="receiver"
# only check authentication for interfaces with following names
interfaces=["feeder"]
```

- Test thử login http://ip:12080/test-auth, nhập user/pass xem kết quả

### config zone
- File `config/pools.toml`

```
[[default]]
address="0.0.0.0"
name="relay.bizflycloud.vn"

[[stg-vccorp]]
address="0.0.0.0"
name="stg.bizflycloud.vn"
```

- File `config/zones/stg-bizflycloud.toml`

```
[stg-bizflycloud]
preferIPv6=false
ignoreIPv6=true
processes=4
connections=10
pool="stg-vccorp"
senderDomains=["domain"]
```

## Config postfix send email test to zonemta
- Mặc định zonemta chỉ hỗ trợ basic auth dạng username:password (không hỗ trợ username@domain:password)
- Cấu hình postfix relay email sang zonemta (sasl auth)
### Config trên postfix server
- Cài đặt postfix

```
sudo apt-get update -y
sudo apt install mailutils -y
```
- Sửa file config `main.cf`

```
#transport_maps = hash:/etc/postfix/transport
sender_dependent_default_transport_maps =  hash:/etc/postfix/transport

smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = loopback-only
mydestination = $myhostname, localhost.$mydomain, $mydomain
```

- reload postfix config

```
postfix reload
```
- relay tất cả các mail có from là (email) sang zonemta server ip => sửa file `/etc/postfix/transport`

```
email smtp:ip:port
```
- apply config

```
postmap /etc/postfix/transport
```
- Sửa file `/etc/postfix/sasl_password` chứa ip:port, user:pass dùng để login vào zonemta

```
ip user:pass
```
- apply config 

```
postmap /etc/postfix/sasl_passwd
```
- Gửi thử một email (from trùng với tên email đã được cấu hình trong file transport thì mới relay được)

```
echo "This is the body of the email" | mail  -a "From: mail gửi"  -s "This is the subject line"  mail nhận
```
- Check log postfix trong /var/log/mail.log
- email đã được relay sang zonemta `relay=ip[ip]:port` có `status=sent` là thành công
- Check log zonemta `journalctl -fu zonemta`

## Sending Zone


