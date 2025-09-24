Pada activity ini, dilakukan setup ci/cd pada project dengan tech stack `laravel`, `vue.js`, dan `mysql`. Skenario ci/cd adalah dengan membuat dockerfile untuk app (berisi laravel dan vue yang akan dibuild), lalu terdapat beberapa konfigurasi tambahan seperti entrypoint dan nginx.

### 1. Dockerfile
`Dockerfile`:
Dockerfile berikut menggunakan base image php:8.2-fpm (sesuai dengan versi laravel), lalu dipasang package yang dibutuhkan oleh laravel+filament+vuejs. 
```sh
FROM php:8.2-fpm

WORKDIR /var/www

RUN apt-get update && apt-get install -y \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libzip-dev \
    zip \
    unzip \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libsodium-dev \
    libicu-dev \
    nodejs \
    npm \
    supervisor \
    netcat-traditional \
    default-mysql-client

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring pcntl bcmath gd zip exif opcache sodium intl

# Install Composer
COPY --from=composer:2.8.5 /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock /var/www/
RUN composer install --no-dev --no-scripts --no-autoloader

# Copy package.json for Node dependencies
COPY package.json package-lock.json* /var/www/

# Copy application code
COPY . /var/www

# Create production environment files
COPY .env.example /var/www/.env

# Complete composer installation and generate autoload
RUN composer dump-autoload --optimize


RUN npm install
RUN npm run build
RUN npm prune --production

#Laravel application key
RUN php artisan key:generate --force \
    && php artisan config:cache 

# Create a backup of public files for volume initialization
RUN cp -r /var/www/public /var/www/public_backup

# Set proper permissions
RUN chown -R www-data:www-data /var/www \
    && chmod -R 755 /var/www \
    && chmod -R 775 /var/www/storage \
    && chmod -R 775 /var/www/bootstrap/cache \
    && chmod -R 755 /var/www/public

# Create supervisor configuration for Laravel processes
RUN mkdir -p /var/log/supervisor
COPY docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create entrypoint script to handle volume initialization
COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Expose port 9000 and start php-fpm server
EXPOSE 9000
ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]
```

### 2. Konfigurasi Tambahan
**`entrypoint.sh`**:
script ini dijalankan setiap kali container start (tertulis pada docckerfile entrypoint). tujuannya adalah memastikan Laravel dapat berjalan dengan mengatur volume untuk folder `public` pada laravel dan memastikan fungsi laravel lain dapat berjalan pada container.
```sh
#!/bin/bash
set -e

#Initialize public files volume
if [ -d "/var/www/public" ]; then
    PUBLIC_VOLUME_FILES=$(find /var/www/public -mindepth 1 -maxdepth 1 2>/dev/null | wc -l)
    
    if [ "$PUBLIC_VOLUME_FILES" -eq 0 ]; then
        echo "Initializing public files volume..."
        
        # Copy built public files from backup to volume
        if [ -d "/var/www/public_backup" ]; then
            cp -r /var/www/public_backup/* /var/www/public/ 2>/dev/null || true
            echo "Public files copied from backup"
        fi
    fi
fi

#Initialize storage
mkdir -p /var/www/storage/app/public /var/www/storage/logs /var/www/storage/framework/cache /var/www/storage/framework/sessions /var/www/storage/framework/views

# Initialize storage symlink if it doesn't exist
if [ ! -L "/var/www/public/storage" ] && [ -d "/var/www/storage/app/public" ]; then
    echo "Creating storage symlink..."
    ln -sf /var/www/storage/app/public /var/www/public/storage
fi

#ownership for volumes
chown -R www-data:www-data /var/www/storage /var/www/public 2>/dev/null || true
chmod -R 775 /var/www/storage 2>/dev/null || true  
chmod -R 755 /var/www/public 2>/dev/null || true

#update or add environment variable in .env file
update_env_var() {
    local key="$1"
    local value="$2"
    local env_file="/var/www/.env"
    
    # Escape special characters in the value for sed
    local escaped_value=$(echo "$value" | sed 's/[[\.*^$()+?{|]/\\&/g')
    
    if grep -q "^${key}=" "$env_file"; then
        # Update existing variable - only update if value actually changed
        current_value=$(grep "^${key}=" "$env_file" | cut -d'=' -f2-)
        if [ "$current_value" != "$escaped_value" ]; then
            sed -i "s|^${key}=.*|${key}=${escaped_value}|" "$env_file"
            echo "  ✓ Updated ${key}"
            return 0
        fi
        return 1  # No change needed
    else
        # Add new variable
        echo "${key}=${value}" >> "$env_file"
        echo "  ✓ Added ${key}"
        return 0
    fi
}

ENV_VARS_TO_SYNC=(
    "APP_NAME"
    "APP_ENV" 
    "APP_DEBUG"
    "APP_URL"
    "DB_CONNECTION"
    "DB_HOST"
    "DB_PORT"
    "DB_DATABASE"
    "DB_USERNAME"
    "DB_PASSWORD"
    "CACHE_DRIVER"
    "SESSION_DRIVER"
    "QUEUE_CONNECTION"
    "LOG_CHANNEL"
    "LOG_LEVEL"
    "MAIL_MAILER"
    "MAIL_HOST"
    "MAIL_PORT"
)

CHANGES_MADE=0
for var in "${ENV_VARS_TO_SYNC[@]}"; do
    if [ ! -z "${!var}" ]; then
        if update_env_var "$var" "${!var}"; then
            CHANGES_MADE=$((CHANGES_MADE + 1))
        fi
    fi
done

if [ $CHANGES_MADE -eq 0 ]; then
    echo "  No environment variables needed updating"
else
    echo "  Synchronized $CHANGES_MADE environment variables"
fi

echo "Environment variables sync completed"

#show database configuration
echo "Verifying database configuration:"
echo "  DB_CONNECTION: $(grep "^DB_CONNECTION=" /var/www/.env | cut -d'=' -f2- || echo 'NOT SET')"
echo "  DB_HOST: $(grep "^DB_HOST=" /var/www/.env | cut -d'=' -f2- || echo 'NOT SET')"
echo "  DB_DATABASE: $(grep "^DB_DATABASE=" /var/www/.env | cut -d'=' -f2- || echo 'NOT SET')"
echo "Testing network connectivity to database host..."
DB_HOST_FROM_ENV=$(grep "^DB_HOST=" /var/www/.env | cut -d'=' -f2- || echo 'db')
DB_PORT_FROM_ENV=$(grep "^DB_PORT=" /var/www/.env | cut -d'=' -f2- || echo '3306')

# Wait for the database service to be available
RETRY_COUNT=0
MAX_RETRIES=30
echo "Waiting for MySQL service to start on $DB_HOST_FROM_ENV:$DB_PORT_FROM_ENV..."

while ! nc -z "$DB_HOST_FROM_ENV" "$DB_PORT_FROM_ENV" 2>/dev/null; do
    RETRY_COUNT=$((RETRY_COUNT + 1))
    if [ $RETRY_COUNT -ge $MAX_RETRIES ]; then
        echo "❌ Database service not reachable after $MAX_RETRIES attempts"
        echo "Checking if database container is running..."
        echo "Current network connectivity test failed for $DB_HOST_FROM_ENV:$DB_PORT_FROM_ENV"
        exit 1
    fi
    echo "Database service not ready, waiting 2 seconds... (attempt $RETRY_COUNT/$MAX_RETRIES)"
    sleep 2
done

#test Laravel database connection
RETRY_COUNT=0
MAX_RETRIES=15
until php artisan tinker --execute="
try {
    \$pdo = DB::connection()->getPdo();
    echo 'Database connected successfully';
} catch (Exception \$e) {
    echo 'Connection failed: ' . \$e->getMessage();
    exit(1);
}
" 2>/dev/null; do
    RETRY_COUNT=$((RETRY_COUNT + 1))
    if [ $RETRY_COUNT -ge $MAX_RETRIES ]; then
        echo "❌ Failed to connect to database after $MAX_RETRIES attempts"
        echo "Current DB configuration:"
        grep "^DB_" /var/www/.env || echo "No DB_ variables found in .env"
        echo ""
        echo "Testing direct MySQL connection..."
        mysql -h"$DB_HOST_FROM_ENV" -P"$DB_PORT_FROM_ENV" -u"$(grep "^DB_USERNAME=" /var/www/.env | cut -d'=' -f2-)" -p"$(grep "^DB_PASSWORD=" /var/www/.env | cut -d'=' -f2-)" -e "SELECT 1;" 2>&1 || echo "Direct MySQL connection also failed"
        exit 1
    fi
    echo "Laravel database connection not ready, waiting 3 seconds... (attempt $RETRY_COUNT/$MAX_RETRIES)"
    sleep 3
done

php artisan config:clear

echo "Container initialization complete"

#Execute the main command
exec "$@"
```

**`nginx.conf`**:
```sh
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 100M;

    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/xml
        image/svg+xml;

    include /etc/nginx/sites-available/*.conf;
}
```
### 3. Docker Compose
**`docker-compose.yml`**:
konfigurasi docker compose berikut akan menjalankan container dari image yang telah dibuild dan dipush ke docker.io. service lain yang dijalankan adalah mysql dan nginx webserver. pada beberapa digunakan environment yang berasal dari environment file workflow.
```sh
services:
  app:
    image: adkurnwn/campigo:${IMAGE_TAG}
    container_name: campigo-app
    restart: unless-stopped
    working_dir: /var/www
    environment:
      APP_NAME: Campigo
      APP_ENV: production
      APP_DEBUG: "false"
      APP_URL: http://localhost
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: ${DB_PORT}
      DB_DATABASE: ${DB_DATABASE}
      DB_USERNAME: ${DB_USERNAME}
      DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - app-storage:/var/www/storage
      - app-public:/var/www/public
    networks:
      - campigo
    depends_on:
      db:
        condition: service_healthy

  webserver:
    image: nginx:alpine
    container_name: campigo-webserver
    restart: unless-stopped
    ports:
      - "8085:80"
    volumes:
      - app-public:/var/www/public:ro
      - app-storage:/var/www/storage:ro
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./docker/nginx/sites/:/etc/nginx/sites-available/:ro
      - ./docker/nginx/ssl/:/etc/ssl/:ro
    depends_on:
      - app
    networks:
      - campigo

  db:
    image: mysql:8.0
    container_name: campigo-db
    restart: unless-stopped
    expose:
      - "3306"
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - campigo-db-data:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - campigo
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-proot_password"]
      timeout: 20s
      retries: 10
      interval: 10s
      start_period: 40s

volumes:
  campigo-db-data:
    driver: local
  app-storage:
    driver: local
  app-public:
    driver: local

networks:
  campigo:
    driver: bridge
```
### 4. Github Workflow
**`cicd.yml`**:
pada workflow berikut, terdapat 2 job yaitu job build dan job deploy. pada job build, dilakukan build image yang selanjutnya di push ke dockerhub. pada job deploy, dilakukan ssh ke server, lalu dilakukan pull image dari dockerhub dan melakukan docker compose untuk menjalankan services. workflow ini berjalan ketika dilakukan push ke branch `dev/actions`. pada yml berikut, banyak digunakan github actions marketplace.
```sh
name: Build and Deploy Campigo

on:
  push:
    branches: 
      - dev/actions
    workflow_dispatch:

env:
  DOCKER_IMAGE: adkurnwn/campigo
  IMAGE_TAG: dev-actions-${{ github.sha }}
  DOCKER_HUB_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
  DOCKER_HUB_ACCESS_TOKEN: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
  DEV_DEPLOY_SERVER: ${{ secrets.DEV_DEPLOY_SERVER }}
  DEV_DEPLOY_USER: ${{ secrets.DEV_DEPLOY_USER }}
  DEV_SSH_PRIVATE_KEY: ${{ secrets.DEV_SSH_PRIVATE_KEY }}
  DEV_DEPLOY_PORT: ${{ secrets.DEV_DEPLOY_PORT }}
  DEV_DEPLOY_PATH: ${{ secrets.DEV_DEPLOY_PATH }}
  
jobs:
  build:
    runs-on: ubuntu-24.04
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ env.DOCKER_HUB_USERNAME }}
          password: ${{ env.DOCKER_HUB_ACCESS_TOKEN }}
          
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_IMAGE }}
          tags: ${{ env.IMAGE_TAG }}
            
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          
  deploy:
    runs-on: ubuntu-24.04
    needs: build
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy to production server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ env.DEV_DEPLOY_SERVER }}
          username: ${{ env.DEV_DEPLOY_USER }}
          key: ${{ env.DEV_SSH_PRIVATE_KEY }}
          port: ${{ env.DEV_DEPLOY_PORT }}
          script: |
            # Set deployment variables
            export DEPLOY_PATH="${{ env.DEV_DEPLOY_PATH }}"
            export IMAGE_TAG="${{ env.IMAGE_TAG }}"
            
            # Create deployment directory if it doesn't exist
            mkdir -p "$DEPLOY_PATH"
            cd "$DEPLOY_PATH"
            
            # Download docker-compose.prod.yml if it doesn't exist
            if [ ! -f "docker-compose.prod.yml" ]; then
              echo "Downloading docker-compose.prod.yml..."
              curl -L -o docker-compose.prod.yml https://raw.githubusercontent.com/${{ github.repository }}/${{ github.ref_name }}/docker-compose.prod.yml
            fi
            
            # Download nginx configuration files
            echo "Downloading nginx configuration..."
            mkdir -p docker/nginx/sites docker/nginx/ssl
            curl -L -o docker/nginx/nginx.conf https://raw.githubusercontent.com/${{ github.repository }}/${{ github.ref_name }}/docker/nginx/nginx.conf || echo "nginx.conf not found, using default"
            curl -L -o docker/mysql/my.cnf https://raw.githubusercontent.com/${{ github.repository }}/${{ github.ref_name }}/docker/mysql/my.cnf || echo "my.cnf not found, using default"
            
            # Set IMAGE_TAG for docker-compose
            echo "IMAGE_TAG=$IMAGE_TAG" > .env
            
            # Pull latest images
            echo "Pulling latest Docker images..."
            docker compose -f docker-compose.prod.yml pull
            
            # Stop services gracefully
            echo "Stopping current services..."
            docker compose -f docker-compose.prod.yml down --timeout 30
            
            # Start services
            echo "Starting services..."
            docker compose -f docker-compose.prod.yml up -d
            
            # Wait for services to be healthy
            echo "Waiting for services to become healthy..."
            timeout 300 bash -c '
              while true; do
                if docker compose -f docker-compose.prod.yml ps | grep -q "Up.*healthy"; then
                  echo "Services are healthy"
                  break
                fi
                echo "Waiting for services to be healthy..."
                sleep 5
              done
            '
            
            # Verify deployment
            echo "Verifying deployment..."
            docker compose -f docker-compose.prod.yml ps
            
            # Clean up old images
            echo "Cleaning up old Docker images..."
            docker image prune -f

            echo "Deployment completed successfully!"
            
      - name: Deployment notification
        if: always()
        run: |
          if [ ${{ job.status }} == 'success' ]; then
            echo "Deployment successful!"
          else
            echo "Deployment failed!"
          fi
```
### 5. Repository Secrets
secrets pada repository diatur pada github. ini digunakan agar informasi rahasia yang akan digunakan pada workflow tidak terekspose.
![enter image description here](https://i.imgur.com/hMIxlLt_d.webp?maxwidth=760&fidelity=grand)
### 6. Build & Deploy
setelah dilakukan push ke branch `dev/ade`, github actions akan melakukan job build & deploy.
![enter image description here](https://i.imgur.com/xpIzRkj_d.webp?maxwidth=1520&fidelity=grand)
![enter image description here](https://i.imgur.com/KQOTCqk_d.webp?maxwidth=1520&fidelity=grand)
![enter image description here](https://i.imgur.com/PoySBip_d.webp?maxwidth=1520&fidelity=grand)

container berjalan pada server:
```sh
root@iZk1a1nf5wshjdz0pef7yrZ:~# docker ps
CONTAINER ID   IMAGE                                                                   COMMAND                  CREATED          STATUS                    PORTS                                     NAMES
5741a0a979c5   nginx:alpine                                                            "/docker-entrypoint.…"   40 minutes ago   Up 37 minutes             0.0.0.0:8085->80/tcp, [::]:8085->80/tcp   campigo-webserver
889cdee7897e   adkurnwn/campigo:dev-actions-eafb4b5d3feebf5cf35ec72f2831ade23d287761   "/entrypoint.sh php-…"   40 minutes ago   Up 39 minutes             9000/tcp                                  campigo-app
14b51d35a65a   mysql:8.0                                                               "docker-entrypoint.s…"   40 minutes ago   Up 40 minutes (healthy)   3306/tcp, 33060/tcp                       campigo-db
```