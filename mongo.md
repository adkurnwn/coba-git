#### 1. Run MongoDB Container
```sh
docker run -d \
  --name mongo \
  -p 27017:27017 \
  -v mongo_data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=RootPassw0rd! \
  mongo:latest
```
Pada perintah tersebut, dijalankan sebuah container docker dengan nama `mongo` dengan menggunakan image `mongo:latest`, yang berjalan pada port `27017`, dan menggunakan volume `mongo_data`, serta mengatur username dan password root untuk MongoDB.

#### 2. Membuat User untuk Aplikasi
A. Dengan menggunakan `mongosh`
Setelah container berjalan, buat user baru dengan hak akses `readWrite` pada database `appdb` menggunakan perintah berikut:
```sh
docker exec -it mongo mongosh -u root -p 'RootPassw0rd!' --authenticationDatabase admin --eval '
db.getSiblingDB("admin").createUser({
  user: "appuser",
  pwd: "AppPassw0rd!",
  roles: [{ role: "readWrite", db: "appdb" }]
});
'
```
Pada perintah tersebut, user `appuser` dengan password `AppPassw0rd!` dibuat dengan privilege `readWrite` pada database `appdb`.

B. Dengan entrypoint script

#### 3. Koneksi ke MongoDB
Untuk terkoneksi ke mongodb, dapat digunakan mongourl berupa:
```mongodb://admin:AdminPassw0rd!@localhost:27017/admin``` untuk user admin/root.

serta:
```mongodb://appuser:AppPassw0rd!@localhost:27017/appdb?authSource=admin``` untuk user aplikasi `appuser` pada database `appdb`.