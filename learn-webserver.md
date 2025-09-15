### Webserver
Webserver adalah software (atau kadang juga hardware) yang berfungsi menerima request dari client, lalu memberikan response berupa data yang dapat berupa API, webpage, ataupun response lainnya.
![webserver architecture](https://i.imgur.com/OikWNST_d.webp?maxwidth=1520&fidelity=grand)

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
Nginx dapat membagi lalu lintas masuk ke beberapa server backend untuk mendistribusikan beban secara merata.
contoh implementasi:
```sh
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 80;
    server_name app.contoh.com;

    location / {
        proxy_pass http://backend;
    }
}
```
dari konfigurasi tersebut, request dibagi rata ke port 3000, 3001, dan 3002.
##### 4. SSL/TLS termination
Nginx dapat menangani enkripsi dan dekripsi koneksi SSL/TLS di sisi depan, sehingga server backend hanya menerima lalu lintas yang sudah didekripsi.

##### 5. Caching server
Nginx dapat menyimpan salinan respons dari server backend dan melayani permintaan berikutnya langsung dari cache. Dengan caching, waktu respons menjadi lebih cepat dan beban pada server backend berkurang.

##### 6. Rate limiting & security
Nginx menyediakan fitur pembatasan jumlah permintaan (rate limiting) untuk mencegah penyalahgunaan seperti serangan DDoS

##### 7. Streaming server
Nginx dapat menangani protokol streaming (misal HLS atau RTMP).
