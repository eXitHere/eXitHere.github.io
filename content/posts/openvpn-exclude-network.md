+++
title = 'ตั้งค่าให้สามารถเชื่อมต่อ Local Network ได้ ตอนที่เชื่อม openvpn'
date = 2024-06-06T00:00:00+07:00
+++

แก้ไขไฟล์ openvpn config หากมีการ import เข้าไปใน openvpn gui แล้วสามารถค้นหาไฟล์การตั้งค่าได้ที่

### สำหรับ macos

```sh
/Users/<your_username_here>/Library/Application\ Support/OpenVPN\ Connect/profiles
```

### เพิ่ม config

```sh
# Exclude my home network routes
route 192.168.1.0 255.255.255.0 net_gateway
route 192.168.2.0 255.255.255.0 net_gateway
```

### ทดสอบ

ทำการเชื่อมต่อ openvpn ใหม่ จะพบว่าสามารถเชื่อมต่อ local network ได้แล้ว หลักการง่าย ๆ ก็คือการทำให้ request ที่ไปยัง destination ดังกล่าวไม่ถูกส่งเข้าไปยัง openvpn นั้นเอง
