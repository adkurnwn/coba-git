### Wordpress standalone dengan native database server
Wordpress dijalankan secara standalone pada dengan container docker, sedangkan database yang digunakan adalah native dari host os (port 3306).

1. pull image wordpress
untuk mendapatkan image docker dari wordpress, dilakukan pull image dengan menggunakan perintah:
```sh
docker pull wordpress
```
2. menjalankan container
```sh
docker run -d \
    --name wp_standalone \
    -p 8083:80 \
    -v wp_standalone_data:/var/www/html \
    -e WORDPRESS_DB_HOST=192.168.1.13:3306 \
    -e WORDPRESS_DB_USER=wp_user_db \
    -e WORDPRESS_DB_PASSWORD=wp123 \
    -e WORDPRESS_DB_NAME=wordpress_docker \
    wordpress:latest
```
dengan perintah tersebut, container wordpress akan dibuat dengan nama `wp_standalone` dan berjalan pada port `8083` host. perintah tersebut juga membuat dan menggunakan volume `wp_standalone_data` sebagai volume dari container wordpress. serta menggunakan env berupa WORDPRESS_DB_HOST, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD.

note: host database merupakan alamat ip dari host os (windows), sehingga apabila menggunakan `localhost`, container akan mengira network yang dimaksud adalah localhost dari container itu sendiri.

3. membuka webpage wordpress
![wordpress di port 8083](https://i.imgur.com/6mJ0o3e_d.webp?maxwidth=1520&fidelity=grand)


### Wordpress standalone dengan native database container pada network yang sama
1. membuat network baru `wp_standalone_network`
```sh
docker network create wp_standalone_network
```
2. menjalankan container mysql
```sh
docker run -d \
    --name wp_mysql_container \
    --network wp_standalone_network \
    -v wp_mysql_data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=samepersonsameoldmistakes \
    -e MYSQL_DATABASE=wordpress \
    -e MYSQL_USER=wp_user_db_beda \
    -e MYSQL_PASSWORD=wp123 \
    mysql:8.0
```
3. mendapatkan ip address container mysql
dengan perintah `docker container inspect wp_mysql_container`, ip address dari container mysql dapat ditemukan dan selanjutnya dapat digunakan untuk menjalankan contianer wordpress pada network yang sama.
```sh
"Networks": {
                "wp_standalone_network": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "0e:6c:50:ab:43:e5",
                    "DriverOpts": null,
                    "GwPriority": 0,
                    "NetworkID": "b7ce66a9ea7aaeb0b18cd2ca296c94fb2e1ca8597712544797d968cb18d5a003",
                    "EndpointID": "d00a6995c2a216d5ffa76720852d7edfcfb2b45985c31e22df8f25facb59f090",
                    "Gateway": "172.19.0.1",
                    "IPAddress": "172.19.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": [
                        "wp_mysql_container",
                        "5cf55abcca2c"
                    ]
                }
```
ip addressnya adalah `172.19.0.2`.

4. menjalankan container wordpress baru
```sh
docker run -d \
    --name wp_standalone_db_also_docker \
    --network wp_standalone_network \
    -p 8085:80 \
    -v wp_standalone_with_mysqldocker_data:/var/www/html \
    -e WORDPRESS_DB_HOST=172.19.0.2:3306 \
    -e WORDPRESS_DB_USER=wp_user_db_beda \
    -e WORDPRESS_DB_PASSWORD=wp123 \
    -e WORDPRESS_DB_NAME=wordpress \
    wordpress:latest
```

![wordpress dengan docker standalone dan container mysql di port 8085](https://i.imgur.com/h0Ov5js_d.webp?maxwidth=1520&fidelity=grand)