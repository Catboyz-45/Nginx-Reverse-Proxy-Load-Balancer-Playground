# 🔀 Nginx Reverse Proxy & Load Balancer Playground

โปรเจกต์สาธิตการทำ **Reverse Proxy** และ **Load Balancing** ด้วย **Nginx + Docker Compose** โดยใช้ Express API ตัวอย่างเป็น backend แล้วให้ Nginx กระจายคำขอ (request) ไปยัง container ของ API ที่รันซ้ำหลายชุด (scale) พร้อมกัน เพื่อดูพฤติกรรม load balancing แบบ round-robin ที่มาจาก DNS ภายในของ Docker Compose เอง

---

## 🎯 แนวคิดของโปรเจกต์

* มี service ชื่อ `api` (Express server ง่ายๆ ที่ตอบกลับ hostname ของ container ที่รับ request)
* มี service ชื่อ `nginx` ทำหน้าที่เป็น **reverse proxy** หน้าบ้าน รับ request แล้วส่งต่อไปที่ `api:3000`
* เมื่อ scale service `api` ให้มีหลาย container พร้อมกัน (เช่น 5 ตัว) Docker Compose จะสร้าง DNS ที่ resolve ชื่อ `api` ไปยังหลาย IP โดยอัตโนมัติ → Nginx proxy ไปที่ `api:3000` แต่ละครั้งจึงถูกส่งไปคนละ container สลับกัน ทำให้เกิดพฤติกรรม **load balancing** แบบ round-robin โดยไม่ต้องเขียน `upstream` block เอง

---

## 🛠️ เทคโนโลยีที่เลือกใช้ (Tech Stack)

* **[Docker](https://www.docker.com/) / [Docker Compose](https://docs.docker.com/compose/):** จัดการและรัน container หลายตัวพร้อมกัน รวมถึงฟีเจอร์ `--scale` สำหรับเพิ่ม/ลดจำนวน container ของ service
* **[Nginx](https://nginx.org/):** ทำหน้าที่ reverse proxy / load balancer หน้าสุด
* **[Node.js](https://nodejs.org/) + [Express](https://expressjs.com/):** เขียน API ตัวอย่างสำหรับทดสอบ (`app.js`) รันบน `node:lts-alpine` image

---

## 📁 โครงสร้างโปรเจกต์ (Project Structure)

```
docker-compose-nginx-reversproxy-loadbalancer/
├── app.js               # Express API ตัวอย่าง (ตอบกลับ hostname ของ container)
├── Dockerfile           # สร้าง image ของ service "api" จาก node:lts-alpine
├── docker-compose.yml   # นิยาม service "api" และ "nginx"
├── nginx.conf           # ตั้งค่า Nginx ให้ listen พอร์ต 3000 แล้ว proxy_pass ไปที่ api:3000
├── package.json         # ระบุ dependency หลัก (express)
└── readme.txt           # คำสั่งตัวอย่างสำหรับทดสอบ scale
```

> ⚠️ หมายเหตุ: ใน `docker-compose.yml` service `nginx` mount volume `./conf.d:/etc/nginx/conf.d` แต่ในรีโปนี้ยังไม่มีโฟลเดอร์ `conf.d/` จริง (มีแค่ไฟล์ `nginx.conf` อยู่ที่ root) หากจะรันให้ Nginx ใช้ config ที่ตั้งใจไว้จริง ต้องสร้างโฟลเดอร์ `conf.d/` แล้วย้าย `nginx.conf` เข้าไปไว้ข้างในก่อน ไม่งั้น Docker จะ mount โฟลเดอร์เปล่าทับ และ Nginx จะใช้ค่า default แทน

---

## 🚀 วิธีใช้งาน (Getting Started)

```bash
# build และรัน container ทั้งหมด (แบบ background)
docker-compose up -d

# ทดสอบ scale service "api" ให้รันพร้อมกัน 5 container
docker-compose up -d --scale api=5

# ลดจำนวนลงเหลือ 2 container
docker-compose up -d --scale api=2

# เข้าถึง API ผ่าน Nginx ที่ host port
curl http://localhost:3210
```

เรียกซ้ำหลายๆ ครั้งจะเห็นค่า `hostname` ใน response เปลี่ยนไปมาระหว่าง container ของ `api` ที่ scale ไว้ ซึ่งเป็นหลักฐานว่า Nginx กระจาย load ไปแต่ละ container จริง

---

## 👥 ผู้พัฒนา (Developer)

* **Nuttaphat** ([@Catboyz-45](https://github.com/Catboyz-45))
