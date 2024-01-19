+++
title = 'Setup Uptime Kuma'
date = 2024-01-19T22:26:45+07:00
+++

## Introduction

โน๊ตนี้คือ วิธีการ setup uptime kuma แบบไม่ใช้ docker

-   สำหรับข้อมูลฉบับเต็มสามารถดูได้ที่ [louislam/uptime-kuma](https://github.com/louislam/uptime-kuma)

#### 1. Basic setup

```sh
sudo apt update && sudo apt upgrade -y
sudo timedatectl set-timezone Asia/Bangkok

sudo apt install curl git -y
```

#### 2. Install nodejs (v.20) and pm2

```sh
curl -sL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh

sudo bash /tmp/nodesource_setup.sh

sudo apt install nodejs

# Verify
node -v

# Install yarn
sudo npm install -g pm2

# Verify
pm2 --version
```

#### 3. Install Uptime Kuma

```sh
git clone https://github.com/louislam/uptime-kuma.git

cd uptime-kuma

npm run setup

npm install pm2 -g && pm2 install pm2-logrotate

pm2 start server/server.js --name uptime-kuma

pm2 save && pm2 startup
```

`http://ip:3001`

#### 4. Update Uptime Kuma

```sh
git fetch --all
git checkout 1.23.11 --force

npm install --production
npm run download-dist

pm2 restart uptime-kuma
```
