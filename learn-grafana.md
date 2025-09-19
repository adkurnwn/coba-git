### A. Instalasi Grafana
menginstall grafana pada vm ubuntu server sesuai dengan dokumentasi dari grafana [https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/).

```sh
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
# Updates the list of available packages
sudo apt-get update
# Installs the latest OSS release:
sudo apt-get install grafana
```
### B. Instalasi Prometheus
1. download binary prometheus
```sh
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.53.0.linux-amd64.tar.gz
cd prometheus-2.53.0.linux-amd64
```
2. jadikan sebagai service (agar dapat berjalan secara otomatis dengan systemctl)
konfigurasi di `/etc/systemd/system/prometheus.service`:
```sh
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/home/<user>/prometheus-2.53.0.linux-amd64/prometheus \
  --config.file=/home/<user>/prometheus-2.53.0.linux-amd64/prometheus.yml \
  --storage.tsdb.path=/home/<user>/prometheus-2.53.0.linux-amd64/data
Restart=always

[Install]
WantedBy=multi-user.target
```
lalu enable dan jalankan prometheus:
```sh
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
### C. Instalasi Node Exporter 
```sh
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
```
jalankan node exporter:
```sh
./node_exporter
```
konfigurasi prometheus (`/etc/prometheus/prometheus.yml`) dengan menambahkan blok:
```sh
- job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```
### D. Tampilkan metrics cpu/ram/disk pada grafana
1. Tambahkan data sources dari prometheus
![enter image description here](https://i.imgur.com/nUAsRet_d.webp?maxwidth=1520&fidelity=grand)
2. Tambahkan dashboard node exporter
menambahkan dashboard node exporter dengan cara melakukan import dengan kode `1860` untuk node exporter full, lalu memilih data source prometheus.
3. Hasil dashboard
![enter image description here](https://i.imgur.com/PUIWbnU_d.webp?maxwidth=1520&fidelity=grand)
### E. Instalasi nginx-prometheus-exporter
1. download binary
```sh
wget https://github.com/nginx/nginx-prometheus-exporter/releases/download/v1.5.0/nginx-prometheus-exporter_1.5.0_linux_amd64.tar.gz
```
2. jalankan node exporter
```sh
./nginx-prometheus-exporter
```
### F. Setup endpoint metrics nginx
membuat config nginx baru pada `/etc/nginx/conf.d/status.conf`, menggunakan module bawaan dari nginx yaitu stub_status:
```sh
server {
    listen 8080;
    server_name localhost;

    location /stub_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```
lalu merestart nginx dengan `sudo systemctl reload nginx`.
pada configurasi prometheus (`/etc/prometheus/prometheus.yml`) ditambahkan blok konfigurasi:
```sh
  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']
```
dengan kedua konfigurasi tersebut, prometheus dapat mengakses data metrik nginx melalui nginx-prometheus-exporter yang mengambil data dari endpoint stub_status di port 8080 dan menyediakannya dalam format yang bisa di-scrape pada port 9113. dari data yang sudah di-scrape tersebut, dapat digunakan sebagai source untuk grafana.
### G. Tampilkan nginx pada grafana
1. menambahkan dashboard baru dengan mengimpor dashboard dengan kode `12708` yang merupakan kode untuk dashboard `NGINX exporter`. kemudian memilih prometeus sebaga source.

2. menguji dengan curl request:
```sh
ade@ubuntuservervm:~$ while true; do   curl -s -o /dev/null -w "%{http_code}\n" http://192.168.110.135/; done
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
200
...
```
3. hasil monitoring:
![enter image description here](https://i.imgur.com/rN7OAxi_d.webp?maxwidth=1520&fidelity=grand)