- Tạo ứng dụng ZoneMTA
```
$ git clone git://github.com/zone-eu/zone-mta-template.git
$ cd zone-mta-template
$ npm install eslint --save-dev
$ npm init
$ npm install --production
$ npm start
```
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
- Cài đặt mongodb
```
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
apt-key list (lệnh kiểm tra khóa)
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list (tạo tệp `sources.list.d` trong thư mục `mongodb-org-4.4.list`)
sudo apt update
sudo apt install mongodb-org
```
