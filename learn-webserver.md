### Webserver
Webserver adalah software (atau kadang juga hardware) yang berfungsi menerima request dari client, lalu memberikan response berupa data yang dapat berupa API, webpage, ataupun response lainnya.
![webserver architecture](https://i.imgur.com/ry9jLBg_d.webp?maxwidth=1520&fidelity=grand)

### Jenis Webserver
##### 1. Apache
Pada awalnya Apache didesain guna mendukung penuh sistem operasi UNIX. Selain cukup mudah dalam implementasinya, Apache juga memiliki beberapa program pendukung sehingga memberinkan layanan yang lengkap, seperti PHP, SSI dan kontrol akses. 
##### 2. Nginx
Nginx dikenal mampu melayani segala macam permintaan, seperti request pada dengan tingkat kepadatan lalu lintas atau traffic yang sangat padat. Nginx memang lebih unggul dari segi kualitas, kecepatan, dan dalam hal performanya.
Nginx memiliki banyak kelebihan dalam hal fitur, di antaranya URL rewriting, virtual host, file serving, reverse proxying, access control, dan masih banyak lagi.
##### 3. IIS
Web server IIS (Internet Information Services) adalah web server yang bekerja pada jenis protokol seperti DNS, TCP/IP, atau beragam software lain.
##### 4. Lighttpd

### Apa saja yang biasa digunakan di nginx?
##### 1. Static file server
Nginx digunakan untuk melayani file statis seperti gambar, CSS, JavaScript, atau file HTML langsung ke pengguna.
contoh implementasi:
```sh
server {
    listen 80;
    server_name adekurniawan.me;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
dari configurasi nginx tersebut, static file seperti html, css, dan js diakses dari folder `var/www/html`.

###### 2. Reverse proxy
Sebagai reverse proxy, Nginx menerima permintaan dari klien dan meneruskannya ke satu atau beberapa server backend. Fitur ini membantu menyembunyikan detail infrastruktur backend dan meningkatkan keamanan.
![reverse proxy](https://cf-assets.www.cloudflare.com/slt3lc6tev37/3msJRtqxDysQslvrKvEf8x/f7f54c9a2cad3e4586f58e8e0e305389/reverse_proxy_flow.png)
contoh implementasi:
```sh
server {
    listen 80;
    server_name api.adekurniawan.me;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
dari konfigurasi tersebut, semua request ke `api.adekurniawan.me` akan diteruskan ke port `3000` (node js).
##### 3. Load balancer
Nginx dapat membagi lalu lintas request masuk ke beberapa server backend untuk mendistribusikan beban secara merata atau sesuai algoritma yang digunakan.
![enter image description here](https://herza.id/wp-content/uploads/2022/12/load-balancing.jpg)

contoh implementasi:
terdapat 3 buah container vuejs yang dijalankan pada image yang sama dan masing-masing menggunakan port 3001, 3002, dan 3003.
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                  NAMES
8481a07842aa   vue-test:serverjs   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3001->80/tcp   vueserverjs1
cb61f0c8ec64   vue-test:serverjs   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3002->80/tcp   vueserverjs2
e99846973efb   vue-test:serverjs   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3003->80/tcp   vueserverjs3
```
`vueserver1`
![enter image description here](https://i.imgur.com/iRqAr8b_d.webp?maxwidth=1520&fidelity=grand)
`vueserver2`
![enter image description here](https://i.imgur.com/QfBwoy4_d.webp?maxwidth=1520&fidelity=grand)
`vueserver3`
![enter image description here](https://i.imgur.com/edO2o1e_d.webp?maxwidth=1520&fidelity=grand)

dari 3 container tersebut, masing-masing dianggap sebagai entitas yang berbeda namun identik. dengan menggunakan loadbalancer dengan algoritma round-robin berikut, request akan dibagi secara bergantian.
```sh
upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
file konfigurasi tersebut disimpan pada `/etc/nginx/conf.d/loadbalancer.conf`. dan dilakukan restart pada service nginx. hasilnya alamat server dengan port 80 (default http) akan menampilkan 3 container sebelumnya secara bergantian berdasarkan request.
![enter image description here](https://i.imgur.com/mRhBUzk_d.webp?maxwidth=1520&fidelity=grand)
##### 4. SSL/TLS termination
Nginx dapat menangani enkripsi dan dekripsi koneksi SSL/TLS di sisi depan, sehingga server backend hanya menerima lalu lintas yang sudah didekripsi.

##### 5. Caching server
Nginx dapat menyimpan salinan respons dari server backend dan melayani permintaan berikutnya langsung dari cache. Dengan caching, waktu respons menjadi lebih cepat dan beban pada server backend berkurang.

##### 6. Rate limiting & security
Nginx menyediakan fitur pembatasan jumlah permintaan (rate limiting) untuk mencegah penyalahgunaan seperti serangan DDoS.
misal penerapan ratelimiting dengan nginx:
pada file configurasi loadbalancer sebelumnya (`/etc/nginx/conf.d/loadbalancer.conf`). ditambahkan konfigurasi untuk rate limiting dengan membatasi request sebesar 5 request per detik:
```sh
#rate limit zone 5req/s
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;

upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;

    location / {
        #apply limit
        limit_req zone=mylimit burst=2 nodelay;

        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    #error response for rate limit
    error_page 503 @limit;
    location @limit {
        return 429 "Too Many Requests\n";
    }
}
```
sehingga ketika melebihi batas tersebut, client tidak akan dapat menerima response dari request yang dibuat.
```sh
ade@Ultron:/mnt/c/Users/adeku$ for i in {1..5}; do curl -I http://localhost; done
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 17 Sep 2025 03:16:55 GMT
Content-Type: text/html
Connection: keep-alive

HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 17 Sep 2025 03:16:55 GMT
Content-Type: text/html
Connection: keep-alive

HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 17 Sep 2025 03:16:55 GMT
Content-Type: text/html
Connection: keep-alive

HTTP/1.1 503 Service Temporarily Unavailable
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 17 Sep 2025 03:16:55 GMT
Content-Type: application/octet-stream
Content-Length: 18
Connection: keep-alive

HTTP/1.1 503 Service Temporarily Unavailable
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 17 Sep 2025 03:16:55 GMT
Content-Type: application/octet-stream
Content-Length: 18
Connection: keep-alive
```
##### 7. Streaming server
Nginx dapat menangani protokol streaming (misal HLS atau RTMP).
