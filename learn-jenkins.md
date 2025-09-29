### Konsep Dasar Jenkins
jenkins adalah sebuah tools open source yang digunakan untuk otomatisasi proses development software terutama dalam hal ci/cd. jenkins dapat terintegrasi dengan berbagai tools lain seperti git, docker, kubernetes, dan lain-lain. 
pada jenkins, proses otomatisasi dilakukan melalui pipeline yang terdiri dari serangkaian tahapan (stages) yang dijalankan secara berurutan.
### Diagram Alur CI/CD dengan Jenkins
![enter image description here](https://i.imgur.com/RxKB6ZS_d.webp?maxwidth=1520&fidelity=grand)
### Instalasi Jenkins di Native
#### 1. instalasi jdk
```sh
sudo apt update
sudo apt install fontconfig openjdk-21-jre
```
#### 2. instalasi jenkins (LTS)
```sh
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```

setelah jenkins terinstal, jenkins secara otomatis terkonfigurasi sebagai service systemd dan berjalan di port 8080.
#### 3. akses jenkins
webui jenkins dapat diakses melalui `http://<ip-address>:8080`. ketika pertama kali mengakses jenkins, jenkins akan meminta untuk memasukkan password awal yang dapat ditemukan di `/var/lib/jenkins/secrets/initialAdminPassword`. ditampilkan dengan perintah:
```sh
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Instalasi Jenkins di Docker
instalasi jenkins di docker memerlukan container docker dind (docker in docker) agar jenkins dapat menjalankan perintah docker di dalamnya. dind adalah sebuah image docker yang menjalankan docker daemon di dalam container, jadi misalnya container jenkins melakukan build, sebenarnya daemon docker yang berjalan di dalam container dind yang melakukan build.
dengan cara:
```sh
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```
jika akan menjalankan container hasil build jenkins di docker, maka:

perlu build image docker jenkins sendiri yang sudah include docker cli dan docker daemon agar jenkins dapat menjalankan perintah docker. misalnya dengan membuat Dockerfile seperti berikut:
```Dockerfile
FROM jenkins/jenkins:2.516.3-jdk21
USER root
RUN apt-get update && apt-get install -y lsb-release ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
    https://download.docker.com/linux/debian $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" \
    | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && apt-get install -y docker-ce-cli && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow json-path-api"
```
lalu build image tersebut dan jalankan container dengan image hasil build tadi:
```sh
docker build -t myjenkins-blueocean:2.516.3-1 .
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.516.3-1
```
atau bisa menggunakan image jenkins biasa yang di link ke docker socket host (belum dicoba, lebih baik untuk ci/cd karena lebih sederhana dan performa lebih tinggi).

### Mencoba Jenkins
#### 1. menghubungkan jenkins dengan github

#### 2. membuat pipeline sederhana

#### 3. melakukan cicd sederhana dengan webhook