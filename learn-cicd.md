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

##### 3. Deployment hasil build ke server
A. Setup nginx
install nginx pada server
```sh
sudo apt install nginx
```
tambahkan userbaru untuk digunakan oleh github actions
```sh
sudo adduser --disabled-password --gecos "" adedeploy
```
setup access ssh (dengan menggunakan localmachine)
```sh
ssh-keygen -t ed25519 -C "github-actions-adedeploy" -f ./adedeploy_github_key
```
setup access ssh di server dengan mengunggah authorized key.
siapkan folder untuk meletakkan hasil build dan akan digunakan oleh nginx melalui sites enabled.
```sh
sudo mkdir -p /home/adedeploy/frontend_app
sudo chown -R adedeploy:adedeploy /home/adedeploy/frontend_app
```
buat file configurasi baru untuk menambahkan avaliable sites pada `/etc/nginx/sites-available/frontend.conf` dan kemudian dilakukan link ke enabled sites
```sh
server {
    listen 80;
    server_name frontend.adekurniawan.me;

    root /home/adedeploy/frontend_app;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```
```sh
sudo ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/
```
setelah langkah tersebut, dilakukan pointing servername frontend.adekurniawan.me ke ip server.

B. File config github actions
setup secrets di repository github
![enter image description here](https://i.imgur.com/SemjR27_d.webp?maxwidth=1520&fidelity=grand)
`cicd.yml`
```sh
name: Frontend Vue.js Build and Deploy

on:
  push:
    branches: [ dev/actions ]
  workflow_dispatch:
    # Manual trigger

# Environment variables for deployment
env:
  DEV_DEPLOY_SERVER: ${{ secrets.DEV_DEPLOY_SERVER }}
  DEV_DEPLOY_USER: ${{ secrets.DEV_DEPLOY_USER }}
  DEV_DEPLOY_PATH: ${{ secrets.DEV_DEPLOY_PATH }}
  DEV_SSH_PRIVATE_KEY: ${{ secrets.DEV_SSH_PRIVATE_KEY }}
  DEV_DEPLOY_PORT: ${{ secrets.DEV_DEPLOY_PORT || '22' }}

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-22.04
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Node.js
      run: |
        echo "Versi nodejs:"
        node -v || echo "Node.js isnt installed"

        # Install NVM (buat milih versi nodejs)
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
        export NVM_DIR="$HOME/.nvm"
        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # Load NVM
        
        # Install and use Node.js 22 (sesuai versi vue sekarang)
        nvm install 22
        nvm use 22

        # cek versi lagi
        echo "Installed Node.js version:"
        node --version
        echo "Installed npm version:"
        npm --version
    
        # Set up npm cache manually
        npm config set cache ~/.npm-cache --global

    - name: Install dependencies
      run: npm ci

    - name: Build
      run: npm run build

    - name: Upload build artifacts
      run: |
        # buat archive folder dist
        echo "Creating artifact archive..."
        mkdir -p $GITHUB_WORKSPACE/artifacts
        tar -czf $GITHUB_WORKSPACE/artifacts/dist.tar.gz -C dist .
    
    - name: Set up SSH key
      run: |
        # Create SSH directory
        mkdir -p ~/.ssh
        # Save the private key from GitHub secrets
        echo "${{ env.DEV_SSH_PRIVATE_KEY }}" > ~/.ssh/deploy_key
        # Set proper permissions (SSH requires these permissions)
        chmod 600 ~/.ssh/deploy_key
        # Add server to known hosts to prevent first-time connection prompts
        ssh-keyscan -p ${{ env.DEV_DEPLOY_PORT }} -H ${{ env.DEV_DEPLOY_SERVER }} >> ~/.ssh/known_hosts
      
    # Deploy the built files to the server
    - name: Deploy to web server
      run: |
        echo "Starting deployment to ${{ env.DEV_DEPLOY_SERVER }}..."
        
        # Create the destination directory if it doesn't exist
        ssh -i ~/.ssh/deploy_key -p ${{ env.DEV_DEPLOY_PORT }} ${{ env.DEV_DEPLOY_USER }}@${{ env.DEV_DEPLOY_SERVER }} "mkdir -p ${{ env.DEV_DEPLOY_PATH }}"

        # Copy the existing tar file to the server
        scp -i ~/.ssh/deploy_key -P ${{ env.DEV_DEPLOY_PORT }} $GITHUB_WORKSPACE/artifacts/dist.tar.gz ${{ env.DEV_DEPLOY_USER }}@${{ env.DEV_DEPLOY_SERVER }}:${{ env.DEV_DEPLOY_PATH }}/dist.tar.gz
        
        # The rest stays the same
        ssh -i ~/.ssh/deploy_key -p ${{ env.DEV_DEPLOY_PORT }} ${{ env.DEV_DEPLOY_USER }}@${{ env.DEV_DEPLOY_SERVER }} "cd ${{ env.DEV_DEPLOY_PATH }} && rm -rf *.html *.js *.css assets && tar -xzf dist.tar.gz && rm dist.tar.gz"
        
        echo "Deployment completed successfully!"
```
C. hasil jobs pada github actions
![enter image description here](https://i.imgur.com/biKvB9s_d.webp?maxwidth=1520&fidelity=grand)
D. hasil project frontend yang berhasil dilakukan ci/cd (tanpa docker)
![enter image description here](https://i.imgur.com/oXlpmFU_d.webp?maxwidth=1520&fidelity=grand)

##### 4. Deployment menggunakan docker dan upload image ke docker.io dengan jobs yang berbeda.
1. setting reverse proxy nginx
membuat file konfigurasi pada `/etc/nginx/sites-available/docker-frontend`
```sh
server {
    listen 80;
    server_name docker-frontend.adekurniawan.me;

    location / {
        proxy_pass http://localhost:8085;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
jadikan konfigurasi menjadi enabled sites
```sh
sudo ln -s /etc/nginx/sites-available/docker-frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```
setelah langkah tersebut, dilakukan pointing servername docker-frontend.adekurniawan.me ke ip server.

2. file cicd-docker.yml
Github action memerlukan file yml yang terletak pada folder `./github/workflows`. pada file `cicd.yml` berikut, terdapat 3 buah jobs yang akan dijalankan secara berurutan pada github actions. job pertama adalah untuk melakukan build pada project vuejs yang kemudian akan diambil `/dist` nya. kemudian job kedua digunakan untuk melakukan build image docker dan melakukan push ke image registry docker.io. lalu job ketiga bersifat opsional (pada yml berikut hanya melakukan perintah echo) dapat digunakan untuk melakukan deployment ke server dengan menggunakan ssh. workflow github actions berikut akan dijalankan apabila terdapat push ke branch `dev/actions`.
pada workflow berikut, terdapat bagian env yang mengambil secrets dari repository github, sehingga github actions dapat melakukan push image ke docker.io dengan username dan PAT.
kemudian pada deploy ke server, github actions akan menggunakan ssh untuk melakukan pull image dari registry docker hub dan membuat container.
```sh
name: Frontend Vue.js CI/CD Pipeline Docker Image

on:
  push:
    branches: [ dev/actions ]
  workflow_dispatch:
    # Manual trigger

# Environment variables
env:
  DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
  DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
  IMAGE_NAME: frontend-test
  IMAGE_TAG: dev-actions-${{ github.sha }}
  DEV_DEPLOY_SERVER: ${{ secrets.DEV_DEPLOY_SERVER }}
  DEV_DEPLOY_USER: ${{ secrets.DEV_DEPLOY_USER }}
  DEV_DEPLOY_PATH: ${{ secrets.DEV_DEPLOY_PATH }}
  DEV_SSH_PRIVATE_KEY: ${{ secrets.DEV_SSH_PRIVATE_KEY }}
  DEV_DEPLOY_PORT: ${{ secrets.DEV_DEPLOY_PORT || '22' }}

jobs:
  build-and-dockerize:
    name: Build and Test
    runs-on: ubuntu-22.04
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Node.js
      run: |
        echo "Versi nodejs:"
        node -v || echo "Node.js isnt installed"

        # Install NVM (buat milih versi nodejs)
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
        export NVM_DIR="$HOME/.nvm"
        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # Load NVM
        
        # Install and use Node.js 22 (sesuai versi vue sekarang)
        nvm install 22
        nvm use 22

        # cek versi lagi
        echo "Installed Node.js version:"
        node --version
        echo "Installed npm version:"
        npm --version
    
        # Set up npm cache manually
        npm config set cache ~/.npm-cache --global

    - name: Install dependencies
      run: npm ci

    - name: Build
      run: npm run build

    - name: Login to Docker Hub
      run: |
        echo "${{ env.DOCKER_HUB_ACCESS_TOKEN }}" | docker login -u ${{ env.DOCKER_HUB_USERNAME }} --password-stdin
        echo "login berhasil: ${{ env.DOCKER_HUB_USERNAME }}"

    - name: Build Docker image
      run: |
        # build image (dengan format namauser/namarepo:branch-githubsha)
        docker build -t ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }} .

    - name: Push Docker image to Docker Hub
      run: |
        docker push ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}

  deploy:
    name: Deploy
    runs-on: ubuntu-22.04
    needs: build-and-dockerize
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up SSH
      run: |
        # Create SSH directory
        mkdir -p ~/.ssh
        # Save the private key from GitHub secrets
        echo "${{ env.DEV_SSH_PRIVATE_KEY }}" > ~/.ssh/deploy_key
        # Set proper permissions (SSH requires these permissions)
        chmod 600 ~/.ssh/deploy_key
        # Add server to known hosts to prevent first-time connection prompts
        ssh-keyscan -p ${{ env.DEV_DEPLOY_PORT }} -H ${{ env.DEV_DEPLOY_SERVER }} >> ~/.ssh/known_hosts
      
    - name: Deploy to Server
      run: |
        CONTAINER_NAME=frontend-test-container
        
        ssh -i ~/.ssh/deploy_key -p ${{ env.DEV_DEPLOY_PORT }} ${{ env.DEV_DEPLOY_USER }}@${{ env.DEV_DEPLOY_SERVER }} << EOF
          set -e
          
          # pull image
          echo "Pulling Docker image..."
          docker pull ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          
          # stop and remove any existing container
          echo "Stopping existing container if running..."
          docker stop $CONTAINER_NAME 2>/dev/null || true
          docker rm $CONTAINER_NAME 2>/dev/null || true
          
          # run container
          echo "Starting new container..."
          docker run -d \
            --name $CONTAINER_NAME \
            -p 8085:80 \
            --restart unless-stopped \
            ${{ env.DOCKER_HUB_USERNAME }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          
          # Verify deployment
          echo "Verifying deployment..."
          docker ps | grep $CONTAINER_NAME
          
          # Clean up old images to save space
          echo "Cleaning up old images..."
          docker system prune -af --volumes
        EOF

```

image berhasil di push ke [docker hub registry](https://hub.docker.com/repository/docker/adkurnwn/vuejs-frontend-app/general):
![docker hub](https://i.imgur.com/WdNn9N5_d.webp?maxwidth=1520&fidelity=grand)
container berhasil dibuat:
```sh
userade@devops:~$ docker ps
CONTAINER ID   IMAGE                                                                         COMMAND                  CREATED          STATUS          PORTS                                     NAMES
1f5aaba175e7   adkurnwn/frontend-test:dev-actions-eef76558c8557c8e04d41ce1a9db45835568caf1   "/docker-entrypoint.…"   40 minutes ago   Up 40 minutes   0.0.0.0:8085->80/tcp, [::]:8085->80/tcp   frontend-test-container

```
hasil deploy dapat dilihat pada docker-frontend.adekurniawan.me:
![enter image description here](https://i.imgur.com/7J0vgMH_d.webp?maxwidth=1520&fidelity=grand)