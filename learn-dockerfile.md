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

#### 2. Singlestage port 80
```sh
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install
RUN npm install -g http-server

COPY . .

RUN npm run build

EXPOSE 80

CMD ["http-server", "dist", "-p", "80"]
```

#### 3. Multistage port 80
```sh
# Stage 1: Build the Vue.js application
FROM node:20-alpine as build-stage

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the application
RUN npm run build

# Stage 2: Serve the application with Nginx
FROM nginx:stable-alpine as production-stage

# Copy the built app from the build stage
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Copy custom nginx config if needed
# COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]

```

Dockerfile dengan multistage tersebut di-push ke docker hub, dengan image name [`adkurnwn/vue-test`](https://hub.docker.com/repository/docker/adkurnwn/vue-test/general). image yang dibuild dapat dilakukan docker run.

Contoh hasil dari singlestage port 3000:
![vue js singlestage port 3000](https://i.imgur.com/xNUtTEc_d.webp?maxwidth=1520&fidelity=grand)