# Panduan Langkah demi Langkah Traefik 2.2 + Docker Pendahuluan

## Dalam panduan ini, kita akan menjalani langkah-langkah berikut:

1. Menyiapkan dan mengonfigurasi Traefik dalam wadah Docker
2. Menyiapkan Let's Encrypt untuk sertifikat HTTPS otomatis
3. Mendeploy layanan sederhana (Portainer) dan mengeksposnya ke internet

## Prasyarat

Untuk dapat mengikuti langkah-langkah ini, Anda memerlukan hal-hal berikut:

1. Docker
2. Docker Compose
3. Sebuah domain
4. Port 80 dan 443 diteruskan ke host Docker Anda

## Langkah 1 — Mengkonfigurasi dan Menjalankan Traefik

Traefik adalah project resmi Docker, dapat di lihat pada `hub.docker.io` untuk image dasar jadi kita hanya perlu konfigurasi sederhana.

Konfigurasi sederhana ini yaitu pengaturan kata sandi terenkripsi agar Anda dapat mengakses dasbor pemantauan.

Anda akan menggunakan utilitas htpasswd untuk membuat kata sandi terenkripsi ini. Pertama, pasang utilitas ini, yang disertakan dalam paket apache2-utils:

```
sudo apt-get install apache2-utils
```
Kemudian buat kata sandi dengan htpasswd. Gantikan `secure_password` dengan kata sandi yang ingin Anda gunakan untuk pengguna admin Traefik:

```
htpasswd -nb admin secure_password
```
Anda akan menggunakan hasil ini dalam file konfigurasi Traefik untuk mengatur Otentikasi Dasar HTTP untuk pemeriksaan kesehatan Traefik dan dasbor pemantauan. Salin seluruh baris keluaran ini agar Anda dapat menempelkannya nanti.

Untuk mengkonfigurasi server Traefik, Anda akan membutuhkan dua file konfigurasi baru yang disebut `traefik.toml` dan `traefik_dynamic.toml` menggunakan format TOML. TOML adalah bahasa konfigurasi yang mirip dengan file INI, tetapi terstandarisasi. File-file ini memungkinkan kita mengkonfigurasi server Traefik dan berbagai integrasi, atau penyedia layanan, yang ingin Anda gunakan. Dalam panduan ini, Anda akan menggunakan tiga penyedia yang tersedia dari Traefik: api, docker, dan acme. Yang terakhir dari ini, acme, mendukung sertifikat TLS menggunakan Let's Encrypt.

Untuk mendapatkan file konfigurasi anda dapat membuat baru atau mengunduh dari repositori ini.

## Langkah 2 – Menjalankan Kontainer Traefik

Pada langkah ini, Anda akan membuat jaringan Docker untuk proxy yang akan digunakan bersama kontainer. Selanjutnya, Anda akan mengakses dasbor Traefik. Jaringan Docker diperlukan agar Anda dapat menggunakannya dengan aplikasi yang dijalankan menggunakan Docker Compose.

Buat jaringan Docker baru dengan nama web:

```
docker network create web
```
Ketika kontainer Traefik mulai berjalan, Anda akan menambahkannya ke dalam jaringan ini. Kemudian Anda dapat menambahkan kontainer tambahan ke dalam jaringan ini nanti untuk di-proxy oleh Traefik.

Selanjutnya, buat berkas kosong yang akan menyimpan informasi Let’s Encrypt Anda. Anda akan membagikannya ke dalam kontainer agar Traefik dapat menggunakannya:

```
touch acme.json
```
Traefik hanya akan dapat menggunakan berkas ini jika pengguna root di dalam kontainer memiliki akses baca dan tulis yang unik ke berkas tersebut. Untuk melakukan ini, kunci izin pada berkas acme.json sehingga hanya pemilik berkas yang memiliki izin baca dan tulis.

```
chmod 600 acme.json
```
Setelah berkas tersebut dilewatkan ke Docker, pemiliknya akan secara otomatis berubah menjadi pengguna root di dalam kontainer.

Terakhir, buat kontainer Traefik dengan perintah ini:

```
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ./traefik.toml:/traefik.toml \
  -v ./traefik_dynamic.toml:/traefik_dynamic.toml \
  -v ./acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2
```

atau bisa dengan mengunakan `docker-compose` pada repositori ini:

```
docker-compose up -d
```