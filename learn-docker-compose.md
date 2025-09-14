### Docker compose wordpress, mysql, dan phpmyadmin
file `docker-compose.yml` berikut berisi konfigurasi untuk menjalankan WordPress, MySQL, dan phpMyAdmin secara bersamaan menggunakan Docker Compose. wordpress menggunakan volume `wordpress_data`, mysql menggunakan volume `db_data`. ketiga container terdapat pada network yang sama, yaitu `wordpress_compose_network`.
```sh
version: '3.8'

services:
  #WordPress
  wordpress:
    image: wordpress:latest
    ports:
      - "8081:80"
    volumes:
      - wordpress_data:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db
    restart: always
    networks:
      - wordpress_compose_network

  #MySQL
  db:
    image: mysql:8.0
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_password
    restart: always
    networks:
      - wordpress_compose_network

  #phpMyAdmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8082:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: root_password
    depends_on:
      - db
    restart: always
    networks:
      - wordpress_compose_network

networks:
  wordpress_compose_network:
    driver: bridge

volumes:
  wordpress_data:
  db_data:
```

code tersebut disimpan dengan nama docker-compose.yml, dan dijalankan dengan perintah
```sh
docker compose up
```
atau `docker compose up -d` agar contianer dapat berjalan di background (detached mode) sehingga terminal tetap bisa digunakan untuk perintah lain.

```sh
ade@Ultron:/mnt/c/Users/adeku$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS         PORTS                  NAMES
011462e01fdf   phpmyadmin/phpmyadmin   "/docker-entrypoint.…"   17 minutes ago   Up 3 seconds   0.0.0.0:8082->80/tcp   docker-compose-wp-phpmyadmin-1
81a91a469ad2   wordpress:latest        "docker-entrypoint.s…"   17 minutes ago   Up 3 seconds   0.0.0.0:8081->80/tcp   docker-compose-wp-wordpress-1
f5e4e4b5ea50   mysql:8.0               "docker-entrypoint.s…"   17 minutes ago   Up 3 seconds   3306/tcp, 33060/tcp    docker-compose-wp-db-1
```
note: karena file docker-compose terdapat pada folder bernama `docker-compose-wp`, maka network yang dibuat akan memiliki format nama awal sesuai dengan nama folder, yang berupa `docker-compose-wp_wordpress_compose_network`.

wordpress dengan docker compose di port 8081:
![wordpress dengan docker compose di port 8081](https://i.imgur.com/gZE6Wjf_d.webp?maxwidth=1520&fidelity=grand)

phpmyadmin pada port 8082:
![phpmyadmin pada port 8082](https://i.imgur.com/D38aGqW_d.webp?maxwidth=1520&fidelity=grand)