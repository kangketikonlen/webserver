# 🚀 Panduan Standar Setup Docker + Web Server (Ubuntu & Proxmox LXC)

Panduan ini mencakup:

* Install Docker
* Menjalankan Docker tanpa `sudo`

---

## 1. Update sistem

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 2. Install package yang dibutuhkan

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

---

## 3. Tambahkan GPG key resmi Docker

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

## 4. Tambahkan repository Docker

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 5. Install Docker Engine

```bash
sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 6. Verifikasi instalasi

```bash
sudo docker run hello-world
```

Jika berhasil → Docker sudah terinstall dengan benar.

---

# 🔐 Menjalankan Docker Tanpa `sudo`

Secara default, Docker butuh akses root. Untuk menjalankan tanpa `sudo`:

---

## 7. Tambahkan user ke group `docker`

```bash
sudo usermod -aG docker $USER
```

---

## 8. Terapkan perubahan group

### Opsi A (direkomendasikan)

```bash
logout
# lalu login kembali
```

### Opsi B (cepat)

```bash
newgrp docker
```

---

## 9. Test tanpa sudo

```bash
docker run hello-world
```

Jika berjalan → berhasil.

---

# ⚠️ Catatan Penting

* Group `docker` memiliki akses setara root
* Hati-hati jika digunakan di server multi-user

---

# 🧪 Cek tambahan

```bash
docker version
docker info
```

---

# 🧠 Troubleshooting

| Masalah                | Penyebab          | Solusi                        |
| ---------------------- | ----------------- | ----------------------------- |
| permission denied      | group belum aktif | logout/login ulang            |
| docker tidak ditemukan | instalasi gagal   | ulangi install                |
| daemon tidak jalan     | service mati      | `sudo systemctl start docker` |
| masalah socket         | permission salah  | cek `/var/run/docker.sock`    |

---

# 🧩 Tambahan (Khusus Proxmox LXC)

Jika kamu menjalankan Docker di dalam LXC container Proxmox, perlu konfigurasi tambahan:

---

## 10. Edit konfigurasi container

Masuk ke host Proxmox:

```bash
nano /etc/pve/lxc/{vmid}.conf
```

Tambahkan baris berikut:

```bash
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
```

> Pastikan ditambahkan di bagian paling bawah file.

---

## 11. Reboot container

```bash
pct reboot {vmid}
```

---

## 12. Masuk ke container

```bash
pct enter {vmid}
```

---

## 13. Setup project Docker

```bash
cd ~
mkdir docker
cd docker

git clone https://github.com/kangketikonlen/webserver.git
cd webserver
```

Copy file environment:

```bash
cp .env.example .env
```

### Contoh isi `.env`

```env
# App Version
MARIADB_VERSION=11.2
REDIS_VERSION=7.4.0
OPENLITESPEED_VERSION=1.8.2-lsphp82
PHPMYADMIN_VERSION=latest

# Port Config
DATABASE_PORT=3306
WEBSERVER_PORT=80
WEBSERVER_SSL_PORT=443
WEBADMIN_PORT=7080
DBMS_PORT=8080

# Mariadb environment
MYSQL_ROOT_PASSWORD=
MYSQL_USER=sysadmin
MYSQL_PASSWORD=
MYSQL_PORT_FORWARD=3306

# Openlitespeed environment
DOMAIN=localhost

# General environment
TimeZone=Asia/Jakarta
```

> Isi password sebelum production

---

## 14. Jalankan docker compose

```bash
docker compose up -d
```

---

## ⚠️ Catatan

* LXC default tidak mendukung Docker tanpa konfigurasi ini
* Alternatif lebih aman: gunakan VM (KVM) daripada LXC

---

# 🌐 Setup Web Folder & LiteSpeed

## 15. Membuat folder website (Contoh Standar)

Masuk ke direktori sites:

```bash
cd webserver/sites/
```

### Contoh

Misal domain: `example.com`

Maka struktur folder:

```bash
mkdir example
cd example
mkdir html
```

Struktur akhir:

```bash
webserver/
└── sites/
    └── example/
        └── html/
```

Copy isi website ke folder `html`:

```bash
cp -r /path/ke/project/* html/
```

---

## 16. Konfigurasi LiteSpeed (Contoh Standar)

Masuk ke panel LiteSpeed:

* Menu **VHost Templates**
* Pilih **docker template**
* Klik **Add on Member Virtual Host Table**

### Contoh input

| Field       | Value       |
| ----------- | ----------- |
| VHost Name  | example     |
| Domain Name | example.com |

Klik **Save**

---

## 📌 Standarisasi Naming

Gunakan pola berikut agar konsisten:

| Item       | Format                            |
| ---------- | --------------------------------- |
| Folder     | nama domain tanpa www (`example`) |
| VHost Name | sama dengan folder (`example`)    |
| Domain     | domain asli (`example.com`)       |

---

## ⚠️ Catatan

* Nama folder dan vhost harus identik (case-sensitive)
* Hindari spasi atau karakter aneh
* Pastikan domain sudah mengarah ke server
