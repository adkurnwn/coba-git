### Komponen Untuk Membuat Container (Docker)
##### 1. Docker Engine 
Docker engine merupakan aplikasi utama yang menjalankan docker, terdapat daemon dan CLI.
##### 2. Dockerfile 
Dockerfile merupakan file yang berisi instruksi untuk melakukan build image docker.
##### 3. Image 
Image merupakan `template` yang berisi os dan aplikasi. image docker dapat didaperoleh dari build menggunakan Dockerfile maupun dari pull image dari registry. Container docker dapat dibuat dan dijalankan melalui image docker ini.
##### 4. Registry 
Registry adalah tempat menyimpan image docker pada `cloud`, sehingga dapat dilakukan pull image darimanapun melalui internet.

### Docker basic commands
```sh
docker image ls
```
perintah docker image ls digunakan untuk melihat daftar image yang terdapat di local. hal yang sama dapat dilakukan denga perintah `docker images`.

```sh
docker image pull <nama-image:tag>
```
perintah docker image pull digunakan untuk mendownload image dari registry. image registry adalah tempat untuk menyimpan image docker di `cloud`. terdapat perintah shortcut berupa `docker pull`.

```sh
docker image build
```
perintah docker image build digunakan untuk melakukan build image dari `Dockerfile`. terdapat perintah shortcut berupa `docker build`. biasanya dieksekusi pada direktori repository yang akan dijadikan image dan memiliki Dockerfile dengan perintah `docker build -t nama:tag .`.

```sh
docker image push <nama-image:tag>
```
perintah docker push digunakan untuk mengunggah image local ke registry. terdapat perintah shortcut berupa `docker push`.

```sh
docker image rm <nama-image:tag>
```
perintah docker image rm digunakan untuk menghapus image. terdapat perintah shortcut berupa `docker rmi`.

```sh
docker image tag <SOURCE_IMAGE[:TAG]> <TARGET_IMAGE[:TAG]>
```
perintah docker image tag digunakan untuk memberikan tag pada image. terdapat perintah shortcut berupa `docker tag`.

```sh
docker container ls
```
perintah docker container ls digunakan untuk melihat daftar container yang sedang berjalan. hal yang sama dapat dilakukan dengan perintah `docker ps`. untuk melihat seluruh container, baik yang sedang running atau exited, dapat menambahkan argumen `-a`.

```sh
docker container run <image>
```
perintah docker container run digunakan untuk membuat dan menjalankan container baru berdasarkan sebuah image. terdapat perintah shortcut berupa `docker run`.

```sh
docker container stop <id/nama container>
```
perintah docker container stop digunakan untuk menghentikan container yang sedang berjalan. terdapat perintah shortcut berupa `docker stop`.

```sh
docker container start <id/nama container>
```
perintah docker container start digunakan untuk menjalankan kembali container yang sudah pernah dibuat namun sedang dalam kondisi berhenti. terdapat perintah shortcut berupa `docker start`.

```sh
docker container restart <id/nama container>
```
perintah docker container restart digunakan untuk memulai ulang container yang sedang berjalan. terdapat perintah shortcut berupa `docker restart`.

```sh
docker container rename <old_name> <new_name>
```
perintah docker image tag digunakan untuk mengubah nama container.

```sh
docker container rm <id/nama container>
```
perintah docker container rm digunakan untuk menghapus container yang sudah berhenti. terdapat perintah shortcut berupa `docker rm`.

```sh
docker container logs <id/nama container>
```
perintah docker container logs digunakan untuk menampilkan log output dari container tertentu. terdapat perintah shortcut berupa `docker logs`.

```sh
docker network ls
```
perintah docker network ls digunakan untuk melihat daftar network.

```sh
docker <image/container/volume/network> inspect <nama-object>
```
perintah docker inspect tersebut digunakan untuk meng-inspect object docker berupa image, container, volume, dan network. yang dikembalikan dari perintah tersebut adalah informasi keseluruhan object dalam json.

### Platform untuk menjalankan container (docker)
Docker umumnya dijalankan pada os linux (platform utama & native untuk container). Docker juga dapat dijalankan pada windows dengan menggunakan wsl2.

### Docker Standalone, Compose, & Orkestra
##### 1. Docker Standalone
Docker standalone adalah salah satu cara/konsep dalam menjalankan container docker dengan hanya 1 container saja (misal dengan perintah docker run). biasanya digunakan dalam proses development/testing. kelebihan docker standalone adalah mudah dipelajari dan langsung digunakan, namun akan tidak efisien jika harus mengelola banyak kontainer yang saling bergantung.

##### 2. Docker Compose
Docker Compose adalah cara lain dalam menjalankan container yang lebih advanced daripada docker standalone, dimana docker compose digunakan untuk mendefinisikan dan menjalankan aplikasi multi-container (dengan docker-compose.yml). dengan perintah `docker compose up`, semua container tersebut akan di-build dan dijalankan secara bersamaan. docker compose berguna untuk environment development lokal di mana aplikasi terdiri dari beberapa layanan, seperti frontend, backend, dan database.

##### 3. Docker Orkestrasi
Orkestrasi adalah cara untuk mengelola, menskalakan, dan mengotomatisasi deployment dan networking dari banyak kontainer di banyak mesin (cluster). Alat orkestrasi dapat berupa Kubernetes atau Docker Swarm, berfungsi untuk memastikan bahwa aplikasi selalu tersedia dan berjalan dengan optimal, dapat secara otomatis menyalakan ulang container yang gagal, menskalakan jumlah kontainer jika lalu lintas meningkat, dan mendistribusikan beban kerja secara merata. orkestrasi dapat diterapkan untuk lingkungan produksi yang membutuhkan availibility dan scalability.