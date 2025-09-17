### Dockerfile tefa-frontoffice-service
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
### Dockerfile tefa-kitchen-service
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
### Dockerfile tefa-waiter-service
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

### Docker compose
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

### Hasil
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