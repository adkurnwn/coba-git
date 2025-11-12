#### 1. Run PostgreSQL Container
```sh
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD='RootPassw0rd!' \
  -e POSTGRES_DB='appdb' \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15
```
Pada perintah tersebut, dijalankan sebuah container docker dengan nama `postgres` dengan menggunakan image `postgres:15`, yang berjalan pada port `5432`, menggunakan volume `pgdata`, serta mengatur password untuk user `postgres` default dan membuat database `appdb`.

#### 2. Create User untuk Aplikasi
```sh
# buat app_admin
docker exec -it postgres psql -U postgres -d appdb -c "CREATE ROLE app_admin WITH LOGIN PASSWORD 'AdminPassw0rd!' CREATEDB CREATEROLE;"

# buat app_rw: hanya login
docker exec -it postgres psql -U postgres -d appdb -c "CREATE ROLE app_rw WITH LOGIN PASSWORD 'AppRwPassw0rd!';"

# buat app_ro: read-only
docker exec -it postgres psql -U postgres -d appdb -c "CREATE ROLE app_ro WITH LOGIN PASSWORD 'AppRoPassw0rd!';"
```
Pada perintah tersebut, dibuat tiga user baru pada database `appdb`, yaitu `app_admin` dengan privilege untuk membuat database dan role, `app_rw` untuk akses read-write, serta `app_ro` untuk akses read-only.

#### 3. Berikan Privilege ke User
```sh
docker exec -it postgres psql -U postgres -d appdb -c "GRANT CONNECT ON DATABASE appdb TO app_admin, app_rw, app_ro;"
docker exec -it postgres psql -U postgres -d appdb -c "GRANT USAGE ON SCHEMA public TO app_admin, app_rw, app_ro;"
```
Perintah pertama memberikan izin CONNECT agar ketiga user dapat terhubung ke database appdb. Perintah kedua memberikan izin USAGE pada schema public yang merupakan schema default di Postgre. Dengan demikian, app_admin memiliki akses penuh untuk mengelola database, app_rw memiliki akses untuk melakukan read dan write, sedangkan app_ro hanya dapat melakukan write saja.