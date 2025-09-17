### Dockerfile Vue JS


#### 1. Singlestage port 3000
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install
RUN npm install -g serve

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["serve", "-s", "dist", "-l", "3000"]
```
pada dockerfile tersebut, digunakan node 20-alpine untuk membuat image Docker. Image alpine dipilih karena ukurannya yang jauh lebih kecil dibandingkan image Node.js reguler.
lalu dilakukan instalasi dependensi berdasarkan package*.json yang dicopy terlebih dahulu ke dalam container untuk memanfaatkan layer caching Docker, sehingga jika tidak ada perubahan pada file package.json, tahap instalasi dapat menggunakan cache.
lalu dilakukan build Vue.js dengan perintah `npm run build` yang akan menghasilkan file statis di folder dist yang siap untuk dideploy.
aplikasi dijalankan menggunakan package serve yang di-install secara global untuk menampilkan konten statis dari folder dist pada port 3000 yang telah diexpose.

#### 2. Singlestage port 80
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 80

CMD ["node", "server.js"]
```
pada dockerfile tersebut, hampir sama dengan dockerfile sebelumnya, hanya saja aplikasi dijalankan menggunakan server.js sebagai entry point, bukan menggunakan package serve. Selain itu, port yang diexpose adalah port 80 yang merupakan port standar untuk HTTP. File `server.js` berisi kode untuk menjalankan server yang akan menyajikan hasil build dari Vue.js.

#### 3. Multistage port 80
```sh
# Stage 1: Build Vue.js
FROM node:20-alpine as build-stage

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the code
COPY . .

# Build
RUN npm run build

# Stage 2: Serve the application with Nginx
FROM nginx:stable-alpine as production-stage

# Copy the built app from the build stage
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]

```

Dockerfile dengan multistage tersebut di-push ke docker hub, dengan image name [`adkurnwn/vue-test`](https://hub.docker.com/repository/docker/adkurnwn/vue-test/general). image yang dibuild dapat dilakukan docker run.

Contoh hasil dari singlestage port 3000:
![vue js singlestage port 3000](https://i.imgur.com/xNUtTEc_d.webp?maxwidth=1520&fidelity=grand)

hasil dari siglestage port 80 (Forwarding port 8089):
![enter image description here](https://i.imgur.com/Ra8QR73_d.webp?maxwidth=1520&fidelity=grand)