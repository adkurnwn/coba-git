### Apa itu monitoring?
Monitoring adalah proses pengawasan dan pelacakan kinerja, kesehatan, dan status sistem, aplikasi, atau infrastruktur TI secara real-time. Tujuannya adalah untuk memastikan bahwa semua komponen berjalan dengan baik, mendeteksi masalah atau anomali sejak dini, dan memberikan wawasan yang dapat digunakan untuk meningkatkan kinerja sistem.

### Mengapa harus monitoring?
Monitoring penting dilakukan karena:
1. **Early Detection**: Memungkinkan deteksi dini terhadap masalah atau kegagalan sistem sebelum berdampak besar.
2. **Performance Optimization**: Membantu dalam mengidentifikasi bottleneck dan area yang memerlukan peningkatan kinerja.
3. **Resource Management**: Memastikan penggunaan sumber daya yang efisien dan menghindari pemborosan.
4. **Analysis & Reporting**: menghasilkan insight dari logs, metrics, dan reports untuk pengambilan keputusan.
5. **SLA (Service Level Agreement) Compliance**: memastikan kualitas layanan sesuai standar uptime, response time, dan performance.

### Tools untuk monitoring
tools yang biasa digunakan untuk monitoring antara lain:
1. **Prometheus**: Sistem monitoring dan alerting open-source yang dirancang untuk merekam metrics dalam format time-series.
2. **Grafana**: Platform open-source untuk visualisasi data dan monitoring yang sering digunakan bersama Prometheus.
3. **ELK Stack**: Kumpulan alat (Elasticsearch, Logstash, Kibana) yang digunakan untuk pengumpulan, pencarian, dan analisis log data.
### Diagram alur kerja monitoring
![enter image description here](https://i.imgur.com/TFZL06i_d.webp?maxwidth=1520&fidelity=grand)
Pada diagram tersebut, setiap server (A, B, C) menjalankan Monitoring Agent yang mengumpulkan data dari System server tersebut (misalnya resource seperti CPU, memory, disk) dan Services (aplikasi atau proses yang berjalan). Data dari masing-masing agent kemudian dikirim ke Monitoring System + timeseries Database (misalnya prometheus) sebagai pusat penyimpanan dan pengolahan. Hasil pengolahan tersebut ditampilkan melalui GUI Monitoring (dashboard/visualisasi, misalnya grafana) agar mudah dianalisis, serta terhubung dengan sistem Alerting yang akan memberikan notifikasi jika ditemukan masalah atau anomali pada server atau service yang dipantau.