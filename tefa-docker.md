## Menggunakan Docker Compose
### A. Dockerfile tefa-frontoffice-service
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

#npm start
CMD ["npm", "start"]
```
pada dockerfile tersebut, digunakan base image node 20-alpine. lalu pada container dibuat dan digunakan folder /app untuk menyimpan dan menjalankan aplikasi. Package.json dicopy terlebih dahulu sebelum instalasi dependencies untuk memanfaatkan cache Docker layer, sehingga jika tidak ada perubahan pada dependencies, tidak perlu mengunduh ulang saat build. Setelah itu, seluruh kode aplikasi dicopy ke dalam container. Port 3000 diexpose untuk dapat diakses dari luar container. Terakhir, aplikasi dijalankan dengan perintah "npm start" saat container berjalan (dengan node.js).
### B. Dockerfile tefa-kitchen-service
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

#npm start
CMD ["npm", "start"]
```
dockerfile tersebut pada dasarnya sama dengan dockerfile dari `tefa-frontoffice-service`. hanya saja tidak dilakukan expose pada port tertentu (sesuai dengan env).
### C. Dockerfile tefa-waiter-service
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

#npm start
CMD ["npm", "start"]
```
dockerfile tersebut pada dasarnya sama dengan dockerfile dari `tefa-frontoffice-service`. hanya saja tidak dilakukan expose pada port tertentu (sesuai dengan env).

### D. Docker compose
```sh
version: '3.8'

services:
  #buat service rabbitmq sesuai dengan req di md dan env
  rabbitmq:
    image: rabbitmq:3.13-management
    ports:
      - "5672:5672" #diakses protocol amqp services lain
      - "15672:15672" #manage ui
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - tefa_network
    healthcheck:
      test: ["CMD", "rabbitmqctl", "status"]
      interval: 30s
      timeout: 10s
      retries: 5

  tefa-frontoffice-service:
    build:
      context: ./tefa-frontoffice-service
      args:
        - TAG=dev #tag dev di image
    ports:
      - "3000:3000"
    environment:
      - AMQP_URL=amqp://guest:guest@rabbitmq:5672 #host rabbitmq (karena di network yang sama)
      - PORT=3000
    depends_on:
      rabbitmq:
        condition: service_healthy
    networks:
      - tefa_network

  tefa-kitchen-service:
    build:
      context: ./tefa-kitchen-service
      args:
        - TAG=dev
    environment:
      - AMQP_URL=amqp://guest:guest@rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy
    networks:
      - tefa_network

  tefa-waiter-service:
    build:
      context: ./tefa-waiter-service
      args:
        - TAG=dev
    environment:
      - AMQP_URL=amqp://guest:guest@rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy
    networks:
      - tefa_network
  
networks:
  tefa_network:
    driver: bridge

volumes:
  rabbitmq_data:
```
pada file docker compose tersebut, terdapat 3 services yang sebelumnya dibuat dockerfilenya. ditambah dengan service rabbitmq dengan langsung menggunakan imagenya dari registry docker.io.

keypoints:
1. network
    semua service berada pada network yang sama agar dapat saling terubung tanpa membuat beberapa service terekpose ke luar. network yang digunakan adalah `tefa_network`.
2. healthcheck
    rabbitmq memiliki konfigurasi healthcheck yang memastikan service benar-benar siap sebelum service lain yang bergantung padanya dimulai. ini dapat mencegah error koneksi saat service lain mencoba terhubung ke rabbitmq yang belum sepenuhnya siap. pada service lain diterapkan `depends_on`.
3. hostname rabbitmq
    dalam konfigurasi environment service lainnya, rabbitmq digunakan sebagai hostname (AMQP_URL=amqp://guest:guest@rabbitmq:5672) karena docker compose dapat secara otomatis menetapkan DNS untuk service berdasarkan nama servicenya dalam network yang sama.

setelah itu dilakukan perintah `docker compose up` untuk menjalankan services tersebut pada docker.

### E. Hasil
container berhasil berjalan:
```sh
ade@Ultron:/mnt/d/tefa$ docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED             STATUS                       PORTS                                                                                                         NAMES
60b03976f06a   tefa-tefa-frontoffice-service   "docker-entrypoint.s…"   About an hour ago   Up About an hour             0.0.0.0:3000->3000/tcp                                                                                        tefa-tefa-frontoffice-service-1
4bb5146abb78   tefa-tefa-kitchen-service       "docker-entrypoint.s…"   About an hour ago   Up About an hour             3000/tcp                                                                                                      tefa-tefa-kitchen-service-1
cf1f371772aa   tefa-tefa-waiter-service        "docker-entrypoint.s…"   About an hour ago   Up About an hour             3000/tcp                                                                                                      tefa-tefa-waiter-service-1
c2945b3a0555   rabbitmq:3.13-management        "docker-entrypoint.s…"   About an hour ago   Up About an hour (healthy)   4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   tefa-rabbitmq-1
```

1. service tefa-frontoffice-service dapat berjalan dan memberikan response:
```sh
ade@Ultron:/mnt/d/tefa$ curl --location 'http://localhost:3000/order' --header 'Content-Type: application/json' --data '{
    "order": {
        "name": "Bambang",
        "food": "Mie Ayam",
        "beverage": "Jeruk Es",
        "table_number": "01"
    }
}'
{"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}ade@Ultron:/mnt/d/tefa$
```
2. service tefa-kitchen-service dapat berjalan:
```sh
ade@Ultron:/mnt/d/tefa$ docker compose logs -f tefa-kitchen-service
WARN[0000] /mnt/d/tefa/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
tefa-kitchen-service-1  | 
tefa-kitchen-service-1  | > shipping-service@1.0.0 start
tefa-kitchen-service-1  | > node index.js
tefa-kitchen-service-1  | 
tefa-kitchen-service-1  | Order received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
tefa-kitchen-service-1  | Will be cooked soon!
tefa-kitchen-service-1  | 
tefa-kitchen-service-1  | Order ready to be served, sending to waiter.
tefa-kitchen-service-1  | Order received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
tefa-kitchen-service-1  | Will be cooked soon!
tefa-kitchen-service-1  | 
tefa-kitchen-service-1  | Order ready to be served, sending to waiter.
```
3. service tefa-waiter-service dapat berjalan:
```sh
ade@Ultron:/mnt/d/tefa$ docker compose logs -f tefa-waiter-service
WARN[0000] /mnt/d/tefa/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
tefa-waiter-service-1  | 
tefa-waiter-service-1  | > waiter-service@1.0.0 start
tefa-waiter-service-1  | > node index.js
tefa-waiter-service-1  | 
tefa-waiter-service-1  | Food & Beverages received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
tefa-waiter-service-1  | Order Served!
tefa-waiter-service-1  | 
tefa-waiter-service-1  | Food & Beverages received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
tefa-waiter-service-1  | Order Served!
tefa-waiter-service-1  | 
tefa-waiter-service-1  | Food & Beverages received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
tefa-waiter-service-1  | Order Served!
```

## Menggunakan Docker Run
membuat network baru dengan command:
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker network create tefa_network
3085d6210a31ef656a1af6a25305532859af5a03a30ebb81afe08e174160019d
```
### melakukan build image pada semua services
1. tefa-frontoffice-service:
```sh
ade@Ultron:/mnt/d/tefa/tefa-frontoffice-service$ docker build -t tefa-frontoffice-service:dev .
[+] Building 15.6s (11/11) FINISHED                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                           0.1s
 => => transferring dockerfile: 203B                                                                           0.0s
 => [internal] load metadata for docker.io/library/node:20-alpine                                              3.3s
 => [auth] library/node:pull token for registry-1.docker.io                                                    0.0s
 => [internal] load .dockerignore                                                                              0.1s
 => => transferring context: 2B                                                                                0.0s
 => [1/5] FROM docker.io/library/node:20-alpine@sha256:eabac870db94f7342d6c33560d6613f188bbcf4bbe1f4eb47d5e2a  0.0s
 => [internal] load build context                                                                              7.0s
 => => transferring context: 4.22MB                                                                            6.9s
 => CACHED [2/5] WORKDIR /app                                                                                  0.0s
 => [3/5] COPY package*.json ./                                                                                0.2s
 => [4/5] RUN npm install                                                                                      3.6s
 => [5/5] COPY . .                                                                                             0.6s 
 => exporting to image                                                                                         0.5s 
 => => exporting layers                                                                                        0.4s 
 => => writing image sha256:d8f86750eb5eb7fdcdfd62fff49d0ac1e2056dd340ac00e9b81329b220e190b4                   0.0s 
 => => naming to docker.io/library/tefa-frontoffice-service:dev   
 ```
2. tefa-kitchen-service:
 ```sh
 ade@Ultron:/mnt/d/tefa/tefa-kitchen-service$ docker build -t tefa-kitchen-service:dev .
[+] Building 2.2s (10/10) FINISHED                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                           0.1s
 => => transferring dockerfile: 168B                                                                           0.0s
 => [internal] load metadata for docker.io/library/node:20-alpine                                              1.2s
 => [internal] load .dockerignore                                                                              0.1s
 => => transferring context: 2B                                                                                0.0s
 => [internal] load build context                                                                              0.3s
 => => transferring context: 149.31kB                                                                          0.2s
 => [1/5] FROM docker.io/library/node:20-alpine@sha256:eabac870db94f7342d6c33560d6613f188bbcf4bbe1f4eb47d5e2a  0.0s
 => CACHED [2/5] WORKDIR /app                                                                                  0.0s
 => CACHED [3/5] COPY package*.json ./                                                                         0.0s
 => CACHED [4/5] RUN npm install                                                                               0.0s
 => [5/5] COPY . .                                                                                             0.1s
 => exporting to image                                                                                         0.2s
 => => exporting layers                                                                                        0.1s
 => => writing image sha256:c49388bce4103cc0113659005f9262fac188a88a7cca9fa9669364debda3bb30                   0.0s
 => => naming to docker.io/library/tefa-kitchen-service:dev          
 ```
 3. tefa-waiter-service:
 ```sh
 ade@Ultron:/mnt/d/tefa/tefa-waiter-service$ docker build -t tefa-waiter-service:dev .
[+] Building 2.0s (10/10) FINISHED                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                           0.1s
 => => transferring dockerfile: 168B                                                                           0.0s
 => [internal] load metadata for docker.io/library/node:20-alpine                                              0.9s
 => [internal] load .dockerignore                                                                              0.1s
 => => transferring context: 2B                                                                                0.0s
 => [1/5] FROM docker.io/library/node:20-alpine@sha256:eabac870db94f7342d6c33560d6613f188bbcf4bbe1f4eb47d5e2a  0.0s
 => [internal] load build context                                                                              0.3s
 => => transferring context: 149.09kB                                                                          0.2s
 => CACHED [2/5] WORKDIR /app                                                                                  0.0s
 => CACHED [3/5] COPY package*.json ./                                                                         0.0s
 => CACHED [4/5] RUN npm install                                                                               0.0s
 => [5/5] COPY . .                                                                                             0.1s
 => exporting to image                                                                                         0.2s
 => => exporting layers                                                                                        0.1s
 => => writing image sha256:91e5277dd34b81c44ee4f4ba3c6a2b23365471e30aff9b0b9ea1a647f6b818ca                   0.0s
 => => naming to docker.io/library/tefa-waiter-service:dev    
 ```

 ```sh
 ade@Ultron:/mnt/c/Users/adeku$ docker images
REPOSITORY                    TAG               IMAGE ID       CREATED         SIZE
tefa-waiter-service           dev               91e5277dd34b   2 minutes ago   141MB
tefa-kitchen-service          dev               c49388bce410   3 minutes ago   141MB
tefa-frontoffice-service      dev               d8f86750eb5e   4 minutes ago   145MB
```

### melakukan docker run
1. rabbitmq
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker run -d \
    --name rabbitmq \
    --network tefa_network \
    -p 5672:5672 \
    -p 15672:15672 \
    -v rabbitmq-data:/var/lib/rabbitmq \
    rabbitmq:3.13-management
3824e5bb56114373e55ae20d0be63cef2e4a9fe303174cca767ed19d4e5fd1eb
```
2. frontoffice
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker run -d \
    --name tefa-frontoffice \
    --network tefa_network \
    -p 3000:3000 \
    -e AMQP_URL=amqp://guest:guest@rabbitmq:5672 \
    -e PORT=3000 \
    tefa-frontoffice-service:dev
db4bea7bd7f82acd82970455bef2464ee3e6053273e95fcbbab72f23c210fb54
```

3. kitchen
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker run -d \
    --name tefa-kitchen \
    --network tefa_network \
    -e AMQP_URL=amqp://guest:guest@rabbitmq:5672 \
    tefa-kitchen-service:dev
082200d2e8a6bb1606f8568c229f843021fe4c913bbee00833530080d94c3671
```

4. waiter
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker run -d \
    --name tefa-waiter \
    --network tefa_network \
    -e AMQP_URL=amqp://guest:guest@rabbitmq:5672 \
    tefa-waiter-service:dev
fdf34ef42d923b59a8a9737dcbb4d47eb19bc5010abd2203fb59eeaaa4fbbb1f
```

### hasil
container berjalan:
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED              STATUS              PORTS                                                                                                         NAMES
fdf34ef42d92   tefa-waiter-service:dev        "docker-entrypoint.s…"   About a minute ago   Up About a minute                                                                                                                 tefa-waiter
082200d2e8a6   tefa-kitchen-service:dev       "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes                                                                                                                      tefa-kitchen
db4bea7bd7f8   tefa-frontoffice-service:dev   "docker-entrypoint.s…"   3 minutes ago        Up 3 minutes        0.0.0.0:3000->3000/tcp                                                                                        tefa-frontoffice
3824e5bb5611   rabbitmq:3.13-management       "docker-entrypoint.s…"   5 minutes ago        Up 5 minutes        4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbitmq
```
request dan response curl:
```sh
ade@Ultron:/mnt/c/Users/adeku$ curl --location 'http://localhost:3000/order' \
--header 'Content-Type: application/json' \
--data '{
    "order": {
        "name": "Bambang",
        "food": "Mie Ayam",
        "beverage": "Jeruk Es",
        "table_number": "01"
    }
}'
{"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
```
log dari tefa-kitchen:
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker logs tefa-kitchen

> shipping-service@1.0.0 start
> node index.js

Order received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
Will be cooked soon!

Order ready to be served, sending to waiter.
```
log dari tefa-waiter:
```sh
ade@Ultron:/mnt/c/Users/adeku$ docker logs tefa-waiter

> waiter-service@1.0.0 start
> node index.js

Food & Beverages received: {"name":"Bambang","food":"Mie Ayam","beverage":"Jeruk Es","table_number":"01"}
Order Served!
```