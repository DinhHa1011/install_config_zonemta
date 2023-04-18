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
