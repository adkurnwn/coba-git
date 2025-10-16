### 1. MinIO
#### A. Mencoba MinIO di Local dengan Docker dan nip.io
MinIO diinstall di local menggunakan Docker bersamaan dengan container nginx untuk reverse proxy dengan docker compose. Domain yang digunakan adalah nip.io sehingga tidak perlu setting DNS karena nip.io akan otomatis mengarahkan ke IP lokal. versi minio yang digunakan adalah versi terbaru (RELEASE.2025-09-07T16-13-09Z-cpuv1), sehingga tidak memiliki fitur bucket policy di ui seperti versi lama.

```yml
services:
  minio:
    image: minio/minio:latest
    container_name: minio
    command: server /data --console-address ":9001"
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    volumes:
      - ./minio-data:/data
    networks:
      - minio-net

  nginx-proxy:
    image: nginx:latest
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx:/etc/nginx:ro
    depends_on:
      - minio
    networks:
      - minio-net

networks:
  minio-net:
    driver: bridge
```
konfigurasi nginx untuk reverse proxy MinIO:

```conf
events {}

http {
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    server {
        listen 443 ssl;
        server_name minio-api.127.0.0.1.nip.io;

        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://minio:9000;
        }
    }

    server {
        listen 443 ssl;
        server_name minio-ui.127.0.0.1.nip.io;

        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://minio:9001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }

    server {
        listen 80;
        server_name minio-api.127.0.0.1.nip.io minio-ui.127.0.0.1.nip.io;
        return 301 https://$host$request_uri;
    }
}
```
dengan konfigurasi diatas, dilakukan `docker compose up -d`, sehingga MinIO dapat diakses melalui:
- API: https://minio-api.127.0.0.1.nip.io
- UI: https://minio-ui.127.0.0.1.nip.io

#### B. Mengakses MinIO dari Cyberduck
Cyberduck adalah aplikasi GUI untuk mengakses berbagai layanan penyimpanan cloud, termasuk MinIO. Untuk mengakses MinIO menggunakan Cyberduck, diperlukan profile S3 dengan decprecated path style (hal ini dikarenakan minio tidak dikonfigurasi dengan virtual host, sehingga untuk mengakses setiap bucket, perlu menggunakan path, bukan subdomain).
![enter image description here](https://i.imgur.com/t7TOPTM_d.webp?maxwidth=760&fidelity=grand)

![enter image description here](https://i.imgur.com/q1tT8bt_d.webp?maxwidth=1520&fidelity=grand)

#### C. Mengakses MinIO dari rclone
Rclone diinstall dengan menggunakan command:
```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```
Setelah itu, dilakukan konfigurasi remote dengan command `rclone config`, dengan memilih:
- n) New remote
- name> local
- Storage> 4) S3
- provider> 22) Minio
- env_auth> false
- access_key_id> minioadmin
- secret_access_key> minioadmin
- endpoint> minio-api.127.0.0.1.nip.io
- y) Yes this is OK

Setelah konfigurasi selesai, dapat dilakukan test dengan command `rclone ls local: --no-check-certificate`, yang akan menampilkan daftar object yang ada di MinIO.
```bash
ade@Ultron:~$ rclone ls local: --no-check-certificate
      783 name-bucket-123/truk1.class
      446 name-bucket-123/truk1.ctxt
      707 name-bucket-123/truk1.java
      936 name-bucket-123/truk2$Truk2.class
      783 name-bucket-123/truk2.class
      446 name-bucket-123/truk2.ctxt
      672 name-bucket-123/truk2.java
   495585 r3ew/CMS Demo/LOGO_KABUPATEN_KLATEN.png
   247497 r3ew/CMS Demo/hero2.jpg
   277599 r3ew/CMS Demo/hero3.jpeg
   255488 r3ew/CMS Demo/profil.png
   346529 r3ew/CMS Demo/struktur.jpg
   454184 r3ew/CMS Demo/unnamed (1).png
   481748 r3ew/CMS Demo/unnamed (2).png
   466300 r3ew/CMS Demo/unnamed (3).png
   963630 r3ew/CMS Demo/unnamed (4).png
   347149 r3ew/CMS Demo/unnamed.png
     1243 test-bucket/exported-data.csv
    49152 test-bucket/image (35).png
```

#### D. Menggunakan MinIO versi lama dengan fitur-fitur lengkap di UI
Versi minio yang digunakan untuk percobaan diatas adalah versi terbaru, yang tidak memiliki fitur bucket policy di UI. Untuk menggunakan fitur-fitur lengkap di UI, dapat menggunakan versi lama, seperti pada versi RELEASE.2025-02-18T16-25-55Z. Dengan versi ini, dapat membuat bucket dengan berbagai opsi seperti public read, public read/write, dll.
![enter image description here](https://i.imgur.com/UtqUFi1_d.webp?maxwidth=1520&fidelity=grand)
### 2. Ceph
#### A. Mencoba Ceph di VM dengan Cephadm
Ceph diinstall di VM menggunakan Cephadm, yang merupakan tool resmi dari Ceph untuk mengelola cluster Ceph. VM yang digunakan adalah VM dengan OS Ubuntu 24.04, dengan 20GB + 10GB + 10GB + 10GB disk. Setelah VM siap, dilakukan instalasi Cephadm dengan command:
```bash
apt install -y cephadm
```
selanjutnya, dilakukan bootstrap cluster Ceph dengan command:
```bash
cephadm bootstrap --mon-ip 192.168.110.142
```
terminal akan menampilkan informasi akses dashboard Ceph, yang dapat diakses melalui browser. Setelah itu, dilakukan login ke dashboard Ceph dengan user `admin` dan password yang ditampilkan di terminal.

Dashboard ui ceph diakses melalui https://192.168.110.142:8443:
![enter image description here](https://i.imgur.com/TTAJTqk_d.webp?maxwidth=1520&fidelity=grand)

#### B. Menambahkan OSD di Ceph
Setelah cluster Ceph berhasil di bootstrap, langkah selanjutnya adalah menambahkan OSD (Object Storage Daemon) ke cluster Ceph. OSD adalah komponen yang bertanggung jawab untuk menyimpan data di cluster Ceph. Dalam percobaan ini, digunakan 3 disk tambahan masing-masing 10GB untuk OSD. Penambahan OSD dilakukan melalui dashboard Ceph dengan memilih menu "OSD" dan kemudian "Create OSD". Setelah itu, dipilih disk yang akan digunakan sebagai OSD dan dilakukan pembuatan OSD.

Selanjutnya, untuk dapat menggunakan fitur object storage di Ceph, perlu dibuat pool RADOS Gateway (RGW). Pool RGW dibuat dengan menggunakan command di dalam chephadm shell (masuk dengan command `sudo cephadm shell`):
```bash
ceph orch apply rgw *<name>* [--realm=*<realm-name>*] [--zone=*<zone-name>*] --placement="*<num-daemons>* [*<host1>* ...]"
```
sehingga terdapat pool seperti berikut:
![enter image description here](https://i.imgur.com/vPF0X2N_d.webp?maxwidth=1520&fidelity=grand).

pada fitur bucket di dashboard ui ceph, terdapat fitur untuk mengatur bucket policy, seperti public read, public read/write, dll.
![enter image description here](https://i.imgur.com/YHC9uOy_d.webp?maxwidth=760&fidelity=grand)
namun pada dashboard ui ceph tidak terdapat fitur untuk melihat object yang ada di dalam bucket, sehingga perlu menggunakan tool lain seperti rclone atau cyberduck.

mengakses ceph dari cyberduck:
![enter image description here](https://i.imgur.com/DgZKZ4f_d.webp?maxwidth=760&fidelity=grand)
![enter image description here](https://i.imgur.com/5vXZtp6_d.webp?maxwidth=1520&fidelity=grand)


#### C. Perbandingan Object Storage Ceph dan Minio
Jika dibandingkan antara Ceph dan MinIO, terdapat beberapa perbedaan yang mencolok, diantaranya:
- Ceph tidak hanya menyediakan object storage, tetapi juga menyediakan block storage dan file system.
- Ceph lebih kompleks dalam hal instalasi dan konfigurasi dibandingkan MinIO.
- Ceph memiliki fitur pola bucket yang lebih lengkap dibandingkan MinIO, dan dapat diatur melalui dashboard UI.
- Ceph mengharuskan replica, sehingga membutuhkan lebih banyak resource untuk menjalankan cluster Ceph.
- MinIO lebih ringan dan mudah digunakan dibandingkan Ceph.

### 3. Garage
#### A. Mencoba Garage pada Docker
Garage diinstall di local menggunakan Docker. Konfigurasi Garage disimpan dalam file `garage.toml`, dengan konfigurasi sebagai berikut:
```toml
metadata_dir = "/tmp/meta"
data_dir = "/tmp/data"
db_engine = "sqlite"

replication_factor = 1

rpc_bind_addr = "[::]:3901"
rpc_public_addr = "127.0.0.1:3901"
rpc_secret = "$(openssl rand -hex 32)"

[s3_api]
s3_region = "garage"
api_bind_addr = "[::]:3900"
root_domain = ".s3.garage.localhost"

[s3_web]
bind_addr = "[::]:3902"
root_domain = ".web.garage.localhost"
index = "index.html"

[k2v_api]
api_bind_addr = "[::]:3904"

[admin]
api_bind_addr = "[::]:3903"
admin_token = "$(openssl rand -base64 32)"
metrics_token = "$(openssl rand -base64 32)"
```
konfigurasi disimpan pada `~/garage/config/garage.toml`, dan storage disimpan pada `~/garage/storage/meta` dan `~/garage/storage/data`. Setelah itu, dilakukan running container garage dengan command:

```bash
docker run \
  -d \
  --name garaged \
  -p 3900:3900 -p 3901:3901 -p 3902:3902 -p 3903:3903 \
  -v ~/garage/config/garage.toml:/etc/garage.toml \
  -v ~/garage/storage/meta:/var/lib/garage/meta \
  -v ~/garage/storage/data:/var/lib/garage/data \
  dxflrs/garage:v2.1.0
```
garage storage tidak mempunyai fitur dashboard ui, sehingga untuk mengelolanya perlu menggunakan cli. dan untuk mengakses bucket, perlu menggunakan tool lain seperti rclone atau cyberduck.
#### B. Mengakses Garage dari Cyberduck
Diperlukan access key dan secret key untuk mengakses garage, yang dapat dihasilkan dengan command:
```bash
garage key create nextcloud-app-key
```
yang kemudian akan menampilkan access key dan secret key. Setelah itu, dilakukan allow access key ke bucket (sebelumnya sudah dibuat) dengan command:
```bash
garage bucket allow \
  --read \
  --write \
  --owner \
  nextcloud-bucket \
  --key nextcloud-app-key
```
![enter image description here](https://i.imgur.com/yEq4KM5_d.webp?maxwidth=760&fidelity=grand)
![enter image description here](https://i.imgur.com/dh3ZTZE_d.webp?maxwidth=1520&fidelity=grand)
#### C. Perbandingan Garage dengan MinIO
- Garage tidak memiliki fitur dashboard UI, sehingga pengelolaan dilakukan melalui CLI.