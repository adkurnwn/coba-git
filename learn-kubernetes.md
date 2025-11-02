### Dasar Kubernetes

#### Pengertian Kubernetes
Kubernetes merupakan platform open-source yang digunakan untuk melakukan manajemen workloads aplikasi yang dikontainerisasi, serta menyediakan konfigurasi dan otomatisasi secara deklaratif. Kubernetes berada di dalam ekosistem yang besar dan berkembang cepat. Service, support, dan perkakas Kubernetes tersedia secara meluas.

Singkatnya, Kubernetes adalah platform open-source yang membantu menjalankan dan mengelola arsitektur microservice dengan cara mengatur deployment, scaling, dan komunikasi antar layanan secara otomatis.
#### Komponen Utama Kubernetes
1. Pods
Pods adalah unit terkecil di Kubernetes yang dapat dideploy dan dikelola. Setiap Pod dapat berisi satu atau lebih kontainer yang berbagi sumber daya jaringan dan penyimpanan.
2. Services
Services adalah abstraksi jaringan yang mendefinisikan cara mengakses Pod. Services dapat mengatur komunikasi antar Pod dan menyediakan load balancing.
3. Deployments
Deployments berfungsi sebagai blueprint untuk menjalankan dan mengelola Pods.
Dengan Deployment, kita bisa menentukan berapa banyak Pod yang dijalankan, versi aplikasi yang digunakan, serta mengatur update atau rollback secara otomatis tanpa menghentikan layanan.
4. Ingress
Ingress adalah bagian yang mengatur akses eksternal ke layanan di dalam cluster, biasanya melalui HTTP atau HTTPS. Ingress dapat menyediakan load balancing, terminasi SSL, dan routing berbasis nama host atau jalur.
5. Secrets
Secrets adalah objek yang digunakan untuk menyimpan informasi sensitif, seperti password, token, atau kunci SSH. Dengan menggunakan Secrets, informasi sensitif dapat disimpan dengan aman dan diakses oleh Pod.

#### Cara Kerja
Pertama, Deployment berfungsi sebagai blueprint yang menentukan bagaimana aplikasi dijalankan di dalam cluster Kubernetes. Melalui Deployment, dapat didefinisikan berapa banyak Pod yang harus dijalankan, versi image container yang digunakan, serta strategi pembaruan aplikasi seperti rolling update atau rollback.

Selanjutnya, Pod merupakan unit terkecil dalam Kubernetes yang menjadi wadah bagi satu atau beberapa container aplikasi. Setiap Pod memiliki alamat IP internal sendiri di dalam cluster, namun alamat ini bersifat sementara karena dapat berubah jika Pod dihentikan dan dibuat ulang. Oleh karena itu, Kubernetes menggunakan komponen lain bernama Service.

Service berfungsi sebagai penghubung antara Pods dan sebagai titik akses tetap bagi aplikasi. Service menyediakan alamat IP virtual yang tetap dan berperan sebagai load balancer internal untuk mendistribusikan lalu lintas ke beberapa Pod di belakangnya. Dengan demikian, meskipun Pod mengalami perubahan, aplikasi tetap dapat diakses secara konsisten melalui Service.

Setelah Service dikonfigurasi, langkah selanjutnya agar aplikasi dapat diakses oleh aaaa adalah dengan menggunakan Ingress. Ingress berfungsi sebagai pintu gerbang utama yang mengatur lalu lintas network dari luar menuju ke Service di dalam cluster Kubernetes. Melalui Ingress, dapat ditentukan aturan akses berbasis domain atau path tertentu, seperti mengarahkan permintaan ke `example.com/abc` menuju Service backend dan `example.com/web` menuju Service frontend.


#### Perintah Dasar Kubernetes
A. `kubectl get`
Perintah ini digunakan untuk mengambil informasi tentang sumber daya di dalam cluster Kubernetes, seperti Pods, Services, Deployments, dll.
1. `kubectl get deployments -n nama-namespace` - Menampilkan daftar Deployment di namespace tertentu.
2. `kubectl get pods -n nama-namespace` - Menampilkan daftar Pod di namespace tertentu.
3. `kubectl get services -n nama-namespace` - Menampilkan daftar Service di namespace tertentu.
4. `kubectl get ingress -n nama-namespace` - Menampilkan daftar Ingress di namespace tertentu.
5. `kubectl get deployments/nama-deployment -n nama-namespace -o yaml` - Menampilkan detail Deployment dalam format YAML.

contoh:
```bash
➜  adeku kubectl get deployments -n xxxxxx-dev
Warning: Use tokens from the TokenRequest abc or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
xxxxxx-exmp-abc-example              1/1     1            1           4d7h
xxxxxx-exmp-fe-xxxxxxxxx             1/1     1            1           4d10h
xxxxxx-example-abc-bbbbbbbbb         1/1     1            1           4d9h
xxxxxx-example-abc-exmp-xxxxxxxxx    1/1     1            1           4d10h
xxxxxx-example-abc-cccccccccccc      1/1     1            1           4d9h
xxxxxx-example-abc-aaaa-exampleabc   1/1     1            1           4d9h
➜  adeku kubectl get pods -n xxxxxx-dev
Warning: Use tokens from the TokenRequest abc or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                                      READY   STATUS    RESTARTS       AGE
xxxxxx-exmp-abc-example-59f7dcddbb-s8cf6              1/1     Running   0              2d10h
xxxxxx-exmp-fe-xxxxxxxxx-76b76b5cc5-mp44j             1/1     Running   0              2d4h
xxxxxx-example-abc-bbbbbbbbb-5ccff87bff-vw4tw         1/1     Running   1 (2d5h ago)   2d10h
xxxxxx-example-abc-exmp-xxxxxxxxx-6f68dc64c-88l7v     1/1     Running   0              3d9h
xxxxxx-example-abc-cccccccccccc-54b4cd5b5b-s6prv      1/1     Running   0              3d9h
xxxxxx-example-abc-aaaa-exampleabc-59bc485f95-9zvfx   1/1     Running   0              3d9h
➜  adeku kubectl get services -n xxxxxx-dev
Warning: Use tokens from the TokenRequest abc or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
xxxxxx-exmp-fe-xxxxxxxxx-service             ClusterIP   10.xxx.183.xxx   <none>        80/TCP              5d9h
xxxxxx-kong-service                          ClusterIP   10.xxx.183.xxx  <none>        8000/TCP,8001/TCP   5d9h
xxxxxx-example-abc-bbbbbbbbb-service         ClusterIP   10.xxx.183.xxx   <none>        3000/TCP            5d9h
xxxxxx-example-abc-exmp-xxxxxxxxx-service    ClusterIP   10.xxx.183.xxx    <none>        3000/TCP            5d9h
xxxxxx-example-abc-cccccccccccc-service      ClusterIP   10.xxx.183.xxx   <none>        3000/TCP            5d9h
xxxxxx-example-abc-aaaa-exampleabc-service   ClusterIP   10.xxx.183.xxx   <none>        3000/TCP            5d9h
➜  adeku kubectl get ingress -n xxxxxx-dev
Warning: Use tokens from the TokenRequest abc or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                   CLASS    HOSTS                        ADDRESS     PORTS     AGE
xxxxxx-exmp-ab-xxxxxxxxx-ingress   public   xxxxxx-abcd.exmp.id      127.0.0.1   80, 443   5d6h
xxxxxx-abcd-ingress                public   abc-dev-xxxxxx.exmp.id   127.0.0.1   80, 443   5d6h
```

B. `kubectl describe`
Perintah ini digunakan untuk menampilkan informasi detail tentang sumber daya tertentu di dalam cluster Kubernetes.
1. `kubectl describe deployment nama-deployment -n nama-namespace` - Menampilkan detail Deployment tertentu.
2. `kubectl describe pod nama-pod -n nama-namespace` - Menampilkan detail Pod tertentu.
3. `kubectl describe service nama-service -n nama-namespace` - Menampilkan detail Service tertentu.
4. `kubectl describe ingress nama-ingress -n nama-namespace` - Menampilkan detail Ingress tertentu.

C. `kubectl logs`
Perintah ini digunakan untuk mengambil log dari kontainer yang berjalan di dalam Pod.
1. `kubectl logs nama-pod -n nama-namespace` - Mengambil log dari Pod tertentu.
2. `kubectl logs deployment/nama-deployment -n nama-namespace` - Mengambil log dari semua Pod yang dikelola oleh Deployment tertentu.
3. `kubectl logs nama-pod -c nama-kontainer -n nama-namespace` - Mengambil log dari kontainer tertentu di dalam Pod.

D. `kubectl apply`
Perintah ini digunakan untuk membuat atau memperbarui resource di dalam cluster Kubernetes berdasarkan file konfigurasi YAML.
1. `kubectl apply -f nama-file.yaml -n nama-namespace` - Menerapkan konfigurasi dari file YAML ke namespace tertentu.
2. `kubectl apply -k nama-direktori/ -n nama-namespace` - Menerapkan konfigurasi dari direktori yang berisi file YAML (yang ada pada kustomization.yaml) ke namespace tertentu.

E. `kubectl config view`
Perintah ini digunakan untuk melihat konfigurasi kubectl saat ini.

F. `kubectl create`
Perintah ini digunakan untuk membuat resource baru di dalam cluster Kubernetes.
1. `kubectl create namespace nama-namespace` - Membuat namespace baru.
2. `kubectl create secret` - Membuat secret baru.
3. `kubectl create deployment nama-deployment` - Membuat deployment baru.

G. `kubectl delete`
Perintah ini digunakan untuk menghapus resource di dalam cluster Kubernetes.
1. `kubectl delete namespace nama-namespace` - Menghapus namespace tertentu.
2. `kubectl delete pod nama-pod -n nama-namespace` - Menghapus Pod tertentu.
3. `kubectl delete deployment nama-deployment -n nama-namespace` - Menghapus deployment tertentu.
4. `kubectl delete secret nama-secret -n nama-namespace` - Menghapus secret tertentu.