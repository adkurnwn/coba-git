### 1. Problem (Possibly)
##### Error di Grafana
![enter image description here](https://i.imgur.com/qu2sDRE.png)
endpoint tersebu (`http://localhost:9090/api/v1/query_range`) adalah milik service prometheus yang merupakan data source dari monitoring grafana tersebut. connection refused karena mungkin pembatasan akses ke endpoint atau service tidak berjalan.

##### Checking service prometheus
setelah dilakukan pengecekan dengan systemctl, service daemon dari prometheus terhenti per tanggal 2025-08-26 08:33:51 UTC. jadi permasalahan terjadi karena prometheus berhenti.
```sh
ahmad@ytc-server:~$ sudo systemctl status prometheus
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Tue 2025-08-26 08:33:51 UTC; 4 weeks 0 days ago
       Docs: https://prometheus.io/docs/introduction/overview/
    Process: 432218 ExecStart=/usr/local/bin/prometheus --enable-feature=remote-write-receiver --config.file=/etc/prometheus>
   Main PID: 432218 (code=exited, status=2)

Warning: journal has been rotated since unit was started, output may be incomplete.

```

log prometheus yang lebih detail tidak dapat ditampilkan karena `journal has been rotated since unit was started, output may be incomplete.` (log sudah dihapus oleh sistem karena lewat masa retention).

##### Checking volume/storage usage
volume `/dev/vdb1` pada server yang dimount ke `mnt/storage` (digunakan untuk menyimpan metrics prometheus) penuh `100%`.
```sh
ahmad@ytc-server:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            435M     0  435M   0% /dev
tmpfs            97M  1.3M   96M   2% /run
/dev/vda1        24G   20G  2.7G  89% /
tmpfs           483M   16K  483M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/loop0       64M   64M     0 100% /snap/core20/2599
/dev/loop1       92M   92M     0 100% /snap/lxd/29619
/dev/loop3       64M   64M     0 100% /snap/core20/2582
/dev/loop2       92M   92M     0 100% /snap/lxd/32662
/dev/vdb1        98G   93G     0 100% /mnt/storage
/dev/loop6       50M   50M     0 100% /snap/snapd/24792
/dev/loop4       51M   51M     0 100% /snap/snapd/25202
tmpfs            97M     0   97M   0% /run/user/1003
```

Kesimpulan: kemungkinan terbesar grafana tidak menampilkan grafik dari metrics adalah karena penyimpanan untuk prometheus penuh lalu membuat service prometheus failed yang menyebabkan metrics dari endpoint prometheus (`localhost:9090`) tidak dapat diakses (tidak ada).

### 2. Fixing
perbaikan dapat dilakukan dengan cara expand volume/storage untuk prometheus, lalu rerun prometheus. selain expand, dapat juga mengosongkan saja storage tersebut, tetapi data metrics sebelumnya akan hilang (tidak benar karena data sebelumnya diperlukan).