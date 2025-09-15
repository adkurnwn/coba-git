#### Pengertian Continous Integration (CI) dan Continous Delivery/Development (CD).
Continuous integration (CI) adalah pengintegrasian kode-kode ke dalam repository kemudian menjalankan pengujian secara otomatis, cepat, dan sering.

Sementara continous delivery atau continuous deployment (CD) adalah praktik yang dilakukan setelah proses CI selesai dan seluruh kode berhasil terintegrasi, sehingga aplikasi bisa dibuild lalu dirilis secara otomatis.

#### Sejarah dan alasan CI/CD muncul
Continuous Integration/Continuous Delivery (CI/CD) muncul sebagai respon terhadap keterbatasan metode pengembangan tradisional seperti waterfall, yang membuat integrasi kode jarang dilakukan dan proses rilis sangat lama. Pada akhir 1990-an hingga awal 2000-an, praktik Agile dan Extreme Programming memperkenalkan ide integrasi kode yang lebih sering untuk menghindari “integration hell”, di mana konflik kode menumpuk dan sulit diselesaikan. Martin Fowler kemudian mempopulerkan istilah Continuous Integration, sementara kemunculan alat seperti Jenkins, Travis CI, GitLab CI/CD, dan GitHub Actions mempercepat adopsinya secara luas.

Alasan utama CI/CD lahir adalah untuk mempercepat feedback, meningkatkan kualitas kode, dan mengurangi risiko saat rilis. Dengan pipeline otomatis yang meliputi build, test, dan deployment, setiap perubahan kode bisa langsung diuji dan disiapkan untuk produksi. Continuous Delivery memastikan aplikasi selalu siap dirilis, sementara Continuous Deployment melangkah lebih jauh dengan otomatisasi penuh hingga ke production.

#### Perbedaan antara Delivery dan Development
Continuous Delivery memastikan aplikasi selalu siap dirilis dengan build dan testing otomatis, tetapi keputusan rilis ke production masih dilakukan manual oleh tim, sedangkan Continuous Deployment bersifat otomatisasi penuh, di mana setiap perubahan kode yang lolos pengujian langsung dirilis ke production tanpa intervensi manusia.

sehingga: delivery memberi kendali manual, sementara deployment menekankan kecepatan dan otomatisasi.
#### Hubungan CI/CD dan Devops
CI/CD dan DevOps memiliki hubungan yang sangat erat karena CI/CD adalah salah satu praktik utama dalam penerapan budaya DevOps. DevOps sendiri adalah pendekatan yang menggabungkan development (Dev) dan operations (Ops) untuk mempercepat siklus pengembangan, meningkatkan kolaborasi tim, serta menjaga kualitas dan stabilitas software. Di dalamnya, CI/CD berperan sebagai “mesin otomatisasi” yang menjembatani proses coding hingga deployment, mulai dari integrasi kode, pengujian, build, hingga rilis.

Dengan CI/CD, tim DevOps bisa memastikan bahwa setiap perubahan kode diuji secara konsisten, siap dirilis kapan saja, dan bahkan otomatis masuk ke production. Hal ini mendukung prinsip utama DevOps yaitu automation, collaboration, dan continuous improvement. Singkatnya, DevOps adalah budaya dan metodologi, sementara CI/CD adalah praktik teknis dan alat yang membuat tujuan DevOps bisa tercapai secara nyata.
#### Deployment Frontend pada github action
##### 1. Membuat repository github dengan frontend vuejs
Tahap pertama adalah menyiapkan repository GitHub yang akan menjadi sumber code. Repository ini berisi project Vue.js yang diinisialisasi menggunakan npm init vue@latest, kemudian dipasang dependensinya dengan npm install. Lalu repository akan dipush ke [github](https://github.com/adkurnwn/frontend-test). Dengan adanya repository ini, pipeline GitHub Actions dapat dijalankan secara otomatis setiap kali ada perubahan kode yang dipush ke branch master atau main (tergantung konfigurasi).
##### 2. Menambahkan code quality scan
Code quality scan yang ditambahkan adalah CodeQL dengan konfigurasi default. code quality scan dengan CodeQL di GitHub Actions berarti kita memanfaatkan built-in security & code analysis engine milik GitHub untuk mendeteksi potensi bug, kerentanan keamanan, serta pola kode yang berisiko di dalam repository. CodeQL bekerja dengan menganalisis source code menggunakan queries yang sudah disediakan GitHub secara default untuk bahasa populer, termasuk JavaScript yang digunakan oleh Vue.js. Integrasi ini biasanya dilakukan dengan menambahkan workflow bawaan “CodeQL Analysis” yang **secara otomatis membuat job tersendiri di GitHub Actions**. contoh jobs pada code scanning:
![enter image description here](https://i.imgur.com/cDzQGeR_d.webp?maxwidth=1520&fidelity=grand)
Ketika workflow berjalan, CodeQL akan melakukan checkout repository, meng-setup bahasa pemrograman yang sesuai, lalu menjalankan proses build agar dapat memindai seluruh dependency dan file yang terkait. Setelah itu, engine CodeQL menjalankan sekumpulan query analisis statis terhadap kode. Hasil pemindaian dikirim kembali ke tab Security → Code scanning alerts di repository GitHub, sehingga developer bisa langsung melihat peringatan atau rekomendasi perbaikan. Dengan cara ini, pipeline CI/CD tidak hanya memastikan aplikasi bisa dibangun dan dideploy, tetapi juga menjaga standar keamanan serta kualitas kode secara berkelanjutan tanpa perlu menambahkan tool eksternal lain. contoh hasil code scanning:
![enter image description here](https://i.imgur.com/PMnvFc7_d.webp?maxwidth=1520&fidelity=grand)
##### 3. Deployment menggunakan docker dan upload image ke docker.io dengan jobs yang berbeda.
Github action memerlukan file yml yang terletak pada folder `./github/workflows`. pada file `cicd.yml` berikut, terdapat 3 buah jobs yang akan dijalankan secara berurutan pada github actions. job pertama adalah untuk melakukan build pada project vuejs yang kemudian akan diambil `/dist` nya. kemudian job kedua digunakan untuk melakukan build image docker dan melakukan push ke image registry docker.io. lalu job ketiga bersifat opsional (pada yml berikut hanya melakukan perintah echo) dapat digunakan untuk melakukan deployment ke server dengan menggunakan ssh. workflow github actions berikut akan dijalankan apabila terdapat push atau pull request ke branch `main` atau `master`.
pada workflow berikut, terdapat bagian env yang mengambil secrets dari repository github, sehingga github actions dapat melakukan push image ke docker.io dengan username dan PAT.
```sh
name: Frontend Vue.js CI/CD Pipeline

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:
    # Manual trigger

# Environment variables available to all jobs and steps in this workflow
env:
  DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
  DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
  IMAGE_NAME: vuejs-frontend-app
  IMAGE_TAG: ${{ github.sha }}

jobs:
# Job 1: build and test the vuejs
  build-and-test:
    name: Build and Test
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build
      run: npm run build

    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: dist
        path: dist/
        retention-days: 1
# Job 2: docker build image & push ke docker.io
  docker-build-push:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: build-and-test
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: dist
        path: dist/

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ env.DOCKER_HUB_USERNAME }}
        password: ${{ env.DOCKER_HUB_ACCESS_TOKEN }}

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }},${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:latest
        cache-from: type=registry,ref=${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:buildcache
        cache-to: type=registry,ref=${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:buildcache,mode=max

# Job 3: deploy ke server
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: docker-build-push
    environment: production
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Deploy to Production
      run: |
        echo "Deploying ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }} to production environment"
        echo "Deploying ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }} to production environment"
        # opsional untuk deployment ke server, misal:
        # ssh user@server "docker pull ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }} && docker-compose up -d"
```