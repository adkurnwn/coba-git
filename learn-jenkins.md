### A. Konsep Dasar Jenkins
jenkins adalah sebuah tools open source yang digunakan untuk otomatisasi proses development software terutama dalam hal ci/cd. jenkins dapat terintegrasi dengan berbagai tools lain seperti git, docker, kubernetes, dan lain-lain. 
pada jenkins, proses otomatisasi dilakukan melalui pipeline yang terdiri dari serangkaian tahapan (stages) yang dijalankan secara berurutan.
### B. Diagram Alur CI/CD dengan Jenkins
![enter image description here](https://i.imgur.com/RxKB6ZS_d.webp?maxwidth=1520&fidelity=grand)
### C. Instalasi Jenkins di Native
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

### D. Instalasi Jenkins di Docker
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

### E. Mencoba Jenkins
Dalam percobaan ini, jenkins sudah dikonfigurasi pada server homelab dengan domain `jenkins.adekurniawan.me`.
#### 1. menghubungkan jenkins dengan github
A. menggunakan github app
Pada github, buat github app di developer settings dengan permission repository read only dan webhook untuk event push. dan install github app tersebut. lalu pada jenkins, buat credentials pada manage jenkins:
1. pilih `Manage Jenkins` -> `Manage Credentials` -> `(global)` -> `Add Credentials`
2. pilih `Kind` `GitHub App` lalu masukkan `App ID`, `Private Key`, dan `Installation ID` dari github app yang sudah dibuat tadi.
private key perlu digenerate ulang dengan cara:
```sh
openssl pkcs8 -topk8 -inform PEM -outform PEM -in key.pem -out converted-github-app.pem -nocrypt
```
lalu masukkan isi `converted-github-app.pem` ke jenkins.
3. test credential dengan cara `Test Connection`, jika berhasil, akan muncul pesan seperti berikut:
![enter image description here](https://i.imgur.com/e8VjpPz_d.webp?maxwidth=760&fidelity=grand)
B. Organization Folder
setelah itu, pada jenkins dapat menambahkan credential github app dan menghubungkan jenkins dengan akun github org. setelah terhubung, jenkins dapat langsung menampilkan semua repository yang ada di akun github org dengan memilih item `Organizations Folder` pada saat membuat item baru di jenkins.
pada configuration organization folder, dapat memilih credential github app yang sudah dibuat tadi, lalu memilih organisasi github yang akan dihubungkan seperti berikut dengan memilih semua repository:
![enter image description here](https://i.imgur.com/kNiqtAI_d.webp?maxwidth=760&fidelity=grand)
setelah itu, jenkins akan menampilkan semua repository yang ada di organisasi github tersebut. dan menampilkan daftar semua repository seperti berikut (yang memiliki jenkinsfile):
![enter image description here](https://i.imgur.com/ahmGEel_d.webp?maxwidth=1520&fidelity=grand)
#### 2. membuat pipeline sederhana
pipeline pada jenkins dapat dibuat dengan menambahkan file `Jenkinsfile` pada repository yang akan di build. contoh `Jenkinsfile` sederhana yang digunakan pada repository project vuejs:
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "frontend-test"
        DOCKERHUB_REPO = "adkurnwn/frontend-test"
        BUILD_TAG      = "dev-actions-jenkins-${env.BUILD_NUMBER}"  // unique per Jenkins build
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev/actions', url: 'https://github.com/kurww/frontend-test.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:latest -t $DOCKER_IMAGE:$BUILD_TAG .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        docker tag $DOCKER_IMAGE:$BUILD_TAG $DOCKERHUB_REPO:$BUILD_TAG

                        docker push $DOCKERHUB_REPO:$BUILD_TAG
                    '''
                }
            }
        }
    }
}
```
pada pipeline di atas, terdapat 3 stage yaitu:
1. Checkout: mengambil source code dari repository github
2. Build Docker Image: build image docker dari source code yang sudah di checkout
3. Push to Docker Hub: push image docker yang sudah dibangun ke Docker Hub. untuk dapat push ke docker hub, perlu menambahkan credential docker hub pada jenkins dengan cara:
- pilih `Manage Jenkins` -> `Manage Credentials` -> `(global)` -> `Add Credentials`
- pilih `Username with password` lalu masukkan username dan PAT docker hub.

#### 3. melakukan cicd sederhana dengan webhook
webhook dapat digunakan untuk memicu build pada jenkins ketika ada perubahan pada repository github. webhook sudah otomatis dibuat ketika menggunakan github app pada langkah sebelumnya. untuk menguji webhook, dapat melakukan perubahan pada repository github yang sudah terhubung dengan jenkins, lalu melakukan push ke branch yang sudah ditentukan pada jenkinsfile (pada contoh di atas adalah `dev/actions`). jika berhasil, maka jenkins akan otomatis memulai build pipeline seperti berikut:
![enter image description here](https://i.imgur.com/7thgiWC_d.webp?maxwidth=1520&fidelity=grand)
recent delivery pada webhook di github dapat dilihat pada menu `Settings` -> `Webhooks` pada repository github:
![enter image description here](https://i.imgur.com/i7Qfzk5_d.webp?maxwidth=760&fidelity=grand)
pipeline juga dapat dilihat pada menu `Blue Ocean`, dimana blue ocean adalah plugin jenkins yang memberikan tampilan yang lebih baik untuk melihat pipeline. pada blue ocean, dapat melihat detail dari setiap stage pada pipeline seperti berikut:
![enter image description here](https://i.imgur.com/ZWnqIsK_d.webp?maxwidth=1520&fidelity=grand)
log console:
```console
Branch indexing
14:33:46 Connecting to https://api.github.com using 2012202/******
Obtained Jenkinsfile from 90349049d87cfd6cc1bad6018f981a9f463a4310
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins
 in /var/jenkins_home/workspace/kurww_frontend-test_dev_actions
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
The recommended git tool is: NONE
using credential jenkins-app
Cloning the remote Git repository
Cloning with configured refspecs honoured and without tags
Cloning repository https://github.com/kurww/frontend-test.git
 > git init /var/jenkins_home/workspace/kurww_frontend-test_dev_actions # timeout=10
Fetching upstream changes from https://github.com/kurww/frontend-test.git
 > git --version # timeout=10
 > git --version # 'git version 2.39.5'
using GIT_ASKPASS to set credentials 
 > git fetch --no-tags --force --progress -- https://github.com/kurww/frontend-test.git +refs/heads/dev/actions:refs/remotes/origin/dev/actions # timeout=10
 > git config remote.origin.url https://github.com/kurww/frontend-test.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/dev/actions:refs/remotes/origin/dev/actions # timeout=10
Avoid second fetch
Checking out Revision 90349049d87cfd6cc1bad6018f981a9f463a4310 (dev/actions)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 90349049d87cfd6cc1bad6018f981a9f463a4310 # timeout=10
Commit message: "Update LoadingSpinner.vue"
First time build. Skipping changelog.
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Checkout)
[Pipeline] git
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/kurww_frontend-test_dev_actions/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/kurww/frontend-test.git # timeout=10
Fetching upstream changes from https://github.com/kurww/frontend-test.git
 > git --version # timeout=10
 > git --version # 'git version 2.39.5'
 > git fetch --tags --force --progress -- https://github.com/kurww/frontend-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/dev/actions^{commit} # timeout=10
Checking out Revision 90349049d87cfd6cc1bad6018f981a9f463a4310 (refs/remotes/origin/dev/actions)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 90349049d87cfd6cc1bad6018f981a9f463a4310 # timeout=10
 > git branch -a -v --no-abbrev # timeout=10
 > git checkout -b dev/actions 90349049d87cfd6cc1bad6018f981a9f463a4310 # timeout=10
Commit message: "Update LoadingSpinner.vue"
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build Docker Image)
[Pipeline] sh
+ docker build -t frontend-test:latest -t frontend-test:dev-actions-jenkins-1 .
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile:
#1 transferring dockerfile: 497B done
#1 DONE 1.4s

#2 [auth] library/nginx:pull token for registry-1.docker.io
#2 DONE 0.0s

#3 [auth] library/node:pull token for registry-1.docker.io
#3 DONE 0.0s

#4 [internal] load metadata for docker.io/library/node:20-alpine
#4 ...

#5 [internal] load metadata for docker.io/library/nginx:stable-alpine
#5 DONE 2.7s

#4 [internal] load metadata for docker.io/library/node:20-alpine
#4 DONE 3.9s

#6 [internal] load .dockerignore
#6 transferring context:
#6 transferring context: 189B done
#6 DONE 1.2s

#7 [build-stage 1/6] FROM docker.io/library/node:20-alpine@sha256:eabac870db94f7342d6c33560d6613f188bbcf4bbe1f4eb47d5e2a08e1a37722
#7 DONE 0.0s

#8 [production-stage 1/2] FROM docker.io/library/nginx:stable-alpine@sha256:8f2bcf97c473dfe311e79a510ee540ee02e28ce1e6a64e1ef89bfad32574ef10
#8 DONE 0.0s

#9 [internal] load build context
#9 transferring context: 122.37kB 0.0s done
#9 DONE 1.3s

#10 [build-stage 5/6] COPY . .
#10 CACHED

#11 [build-stage 6/6] RUN npm run build
#11 CACHED

#12 [build-stage 2/6] WORKDIR /app
#12 CACHED

#13 [build-stage 3/6] COPY package*.json ./
#13 CACHED

#14 [build-stage 4/6] RUN npm ci
#14 CACHED

#15 [production-stage 2/2] COPY --from=build-stage /app/dist /usr/share/nginx/html
#15 CACHED

#16 exporting to image
#16 exporting layers done
#16 writing image sha256:8489fdd067bdf491235f8dee122354ad7e74d2e3220eaffcd05199cc85939aee 0.1s done
#16 naming to docker.io/library/frontend-test:latest
#16 naming to docker.io/library/frontend-test:latest 0.3s done
#16 naming to docker.io/library/frontend-test:dev-actions-jenkins-1
#16 naming to docker.io/library/frontend-test:dev-actions-jenkins-1 0.3s done
#16 DONE 1.3s

 [33m2 warnings found (use docker --debug to expand):
[0m - FromAsCasing: 'as' and 'FROM' keywords' casing do not match (line 2)
 - FromAsCasing: 'as' and 'FROM' keywords' casing do not match (line 13)
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Push to Docker Hub)
[Pipeline] withCredentials
Masking supported pattern matches of $DOCKER_USER or $DOCKER_PASS
[Pipeline] {
[Pipeline] sh
+ echo ****
+ docker login -u **** --password-stdin
Login Succeeded
+ docker tag frontend-test:dev-actions-jenkins-1 ****/frontend-test:dev-actions-jenkins-1
+ docker push ****/frontend-test:dev-actions-jenkins-1
The push refers to repository [docker.io/****/frontend-test]
c72911bf8b75: Preparing
3662d61b1197: Preparing
19c101c8a6e8: Preparing
d3c82e18bdb8: Preparing
72997926d63c: Preparing
c79f431f2893: Preparing
60e1771cb327: Preparing
f76dbf6a56fe: Preparing
7003d23cc217: Preparing
c79f431f2893: Waiting
60e1771cb327: Waiting
f76dbf6a56fe: Waiting
7003d23cc217: Waiting
72997926d63c: Layer already exists
19c101c8a6e8: Layer already exists
3662d61b1197: Layer already exists
d3c82e18bdb8: Layer already exists
c72911bf8b75: Layer already exists
c79f431f2893: Layer already exists
60e1771cb327: Layer already exists
f76dbf6a56fe: Layer already exists
7003d23cc217: Layer already exists
dev-actions-jenkins-1: digest: sha256:b0f601a23c6a3eab6287e37a146df20540b8630a71668e1aeb600a3695977332 size: 2198
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline

GitHub has been notified of this commit’s build result

Finished: SUCCESS
```