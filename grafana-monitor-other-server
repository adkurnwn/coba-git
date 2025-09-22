### Setup pada server nginx
pada server nginx, dilakukan stub untuk menghasilkan log yang akan dikirimkan ke prometheus.

```sh
.....

    location /stub_status {
        stub_status;
        allow 127.0.0.1;      # allow localhost
        deny all;             # deny everyone else
    }
.....
```
log dari nginx dapat dikirimkan dan dibaca oleh prometheus dengan menggunakan `nginx-prometheus-exporter`. sehingga perlu dilakukan instalasi exporter tersebut dengan cara mengunduh binarynya:
```sh
wget https://github.com/nginx/nginx-prometheus-exporter/releases/download/v1.5.0/nginx-prometheus-exporter_1.5.0_freebsd_amd64.tar.gz
tar -xvzf nginx-prometheus-exporter_1.5.0_freebsd_amd64.tar.gz
```
lalu dijalankan, sehingga logs akan terekspose pada port 9113 yang akan dibaca dan diporses oleh prometheus.
### 1. Monitoring dengan Grafana+Prometheus Docker compose
#### A. Konfigurasi
pada configurasi docker compose berikut, dibuat service berupa grafana dan prometheus. grafana dibuat dengan mengekspose port 4000 dan menggunakan volume grafana-storage. sedangkan prometheus dibuat dengan volume prometheus-storage. kedua services tersebut dibuat dalam network yang sama yaitu  `monitoring`. hal ini dilakukan agar endpoint dari prometheus tidak dapat diakses keluar (hanya dapat diakses oleh network tersebut pada port 9090). prometheus juga dibuat dengan configurasi  `prometheus.yml`.

**`docker-compose.yml`**:
```sh
version: '3.8'

networks:
  monitoring:
    driver: bridge

services:
  grafana:
    image: grafana/grafana:12.2.0-17567790421
    container_name: grafana
    restart: unless-stopped
    ports:
      - '4000:3000'
    networks:
      - monitoring
    volumes:
      - grafana-storage:/var/lib/grafana

  prometheus:
    image: prom/prometheus:v3.6.0-rc.1
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-storage:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    expose:
      - 9090
    networks:
      - monitoring

volumes:
  grafana-storage: {}
  prometheus-storage: {}
```
**`prometheus.yml`**:
```sh
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  #scrape nginx loadbalancer server sebelah
  - job_name: 'nginx'
    static_configs:
      - targets: ['192.168.110.137:9113']
```
pada konfigurasi prometheus tersebut, dilakukan scraping pada endpoint server lain yang terdapat nginx pada `192.168.110.137:9113`.

lalu dijalankan dengan `docker compose up -d` agar kontainer prometheus dan grafana berjalan. lalu interface grafana dibuka pada alamat server pada port `4000`.
![enter image description here](https://i.imgur.com/d2xy2PW_d.webp?maxwidth=1520&fidelity=grand)

#### B. Connecting
Grafana akan berjalan pada port `40004` dan Prometheus ditambahkan sebagai data source pada grafana, lalu digunakan `http://prometheus:9090` sebagai server URL prometheus karena terdapat pada **network yang sama** (docker).
![enter image description here](https://i.imgur.com/4jijp2s_d.webp?maxwidth=760&fidelity=grand)
lalu dibuat dashboard dengan mengimport dashboard `12708` dan menggunakan prometheus sebagai data source. dashboard akan menampilkan monitoring dari nginx.
![enter image description here](https://i.imgur.com/DdLUmEU_d.webp?maxwidth=1520&fidelity=grand)
### 2. Monitoring dengan Native Grafana+Prometheus
#### A. Instalasi dan konfigurasi
**Grafana:**
```sh
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
# Updates the list of available packages
sudo apt-get update
# Installs the latest OSS release:
sudo apt-get install grafana
```
lalu jalankan dan buat grafana menjadi daemon: `sudo systemctl start grafana-server` dan `sudo systemctl enable grafana-server`

**Prometheus:**
```sh
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.darwin-amd64.tar.gz
tar -xvzf prometheus-3.5.0.darwin-amd64.tar.gz
```
lalu jadikan daemon dan jalankan dengan konfigurasi:
```sh
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus

[Install]
WantedBy=default.target
```
#### B. Connecting
Grafana akan berjalan pada port `3000` dan Prometheus ditambahkan sebagai data source pada grafana, lalu digunakan `http://localhost:9090` sebagai server URL prometheus.
lalu dibuat dashboard dengan mengimport dashboard `12708` dan menggunakan prometheus sebagai data source. dashboard akan menampilkan monitoring dari nginx.
![enter image description here](https://i.imgur.com/wLtxhb5_d.webp?maxwidth=1520&fidelity=grand)