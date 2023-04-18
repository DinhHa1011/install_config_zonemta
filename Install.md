### Tạo ứng dụng ZoneMTA
```
$ git clone git://github.com/zone-eu/zone-mta-template.git
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
sudo apt-get purpe nodejs
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
```
