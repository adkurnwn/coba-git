### Struktur Dasar Permission
Setiap file dan folder memiliki 3 jenis permission dan 3 jenis user.
##### 1. Jenis Permission
- read (`r`) => bisa membaca isi file, atau melihat daftar isi folder.
- write (`w`) => bisa mengubah isi file, atau menambah/menghapus file dalam folder.
- execute (`x`) => bisa mengeksekusi file (jika program/script), atau masuk ke folder (`cd`).
##### 2. Jenis User
- user owner (`u`) => pemilik file.
- user group (`g`) => grup yang terkait dengan file.
- others user (`o`) => semua pengguna lain di sistem.

misalnya pada folder home, dilakukan perintah `ls -l`:

```sh
┌──(kali㉿kali)-[~]
└─$ ls -l
total 36
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Desktop
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Documents
drwxr-xr-x 2 kali kali 4096 Nov  8  2024 Downloads
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Music
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Pictures
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Public
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Templates
drwxrw-r-- 2 kali kali 4096 Sep  9 19:42 testing
-rwxrw-r-- 1 kali kali    0 Sep  9 19:39 testing.sh
drwxr-xr-x 2 kali kali 4096 Oct 25  2024 Videos
```
pada item pertama (`testing.sh`), permissionnya adalah `-rwxrw-r--`. pada struktur tersebut terdapat 4 bagian utama (10 karakter):
- karakter pertama (`-`) => merupakan jenis, dapat berupa symbolic (`l`) atau folder (`d`) atau file (`-`).
- karakter ke 2, 3, 4 (`rwx`) => merupakan permission untuk user owner, `rwx` berarti user owner dapat melakukan write, read, dan execute.
- karakter ke 5, 6, 7 (`rw-`) => merupakan permission untuk user pada group yang sama dengan owner, `rw-` berarti user tersebut dapat melakukan write dan read saja.
- karakter ke 8, 9, 10 (`r--`) => merupakan permission untuk user others, `r--` berarti user lainnya hanya dapat melakukan read saja.

kode permission tersebut juga dapat berupa angka/oktal 3 digit, dimana masing masing digit merupakan hasil penjumlahan dari permission yang aktif. dengan:
`r`= 4
`w`= 2
`x`= 1
berarti jika permission adalah `-rwxrw-r--`, maka bentuk oktalnya adalah:
- digit pertama: `rwx` = 4+2+1 = 7
- digit kedua: `rw-` = 4+2+0 = 6
- digit ketiga: `r--` = 4+0+0 = 4
sehingga permissionnya adalah `764`. kode tersebut dapat diperoleh dengan perintah `stat -c "%a %n"`:
```sh
┌──(kali㉿kali)-[~]
└─$ stat -c "%a %n" testing.sh
764 testing.sh
```
cara untuk mengetahui nilai oktal tersebut dapat mengguankan calculator berikut: __[chmod-calculator.com](https://chmod-calculator.com/)__
![chmod-calculator.com](https://i.imgur.com/Z0Mscle_d.webp?maxwidth=760&fidelity=grand)
### Manipulasi Permission File
Memanipulasi permission untuk file dan folder dapat menggunakan perintah `chmod` (change mode). contoh penggunaannya adalah:
```sh
┌──(kali㉿kali)-[~]
└─$ chmod 777 testing.sh
```
perintah `chmod` diikuti dengan angka oktal permission yang akan diterapkan, lalu diikuti dengan nama file/folder yang akan diterapkan permission barunya. perintah tersebut digunakan oleh user owner. selain user owner, `superuser` dapat memanipulasi file/folder tersebut.
### Kepemilikan File
Setiap file/folder memiliki user owner dan group. dapat dilihat dengan perintah `ls -l nama-file-atau-folder`. misalnya:
```sh
┌──(kali㉿kali)-[~]
└─$ ls -l testing.sh
-rwxrwxrwx 1 kali kali 0 Sep  9 19:39 testing.sh
```
pada file testing.sh tersebut, user ownernya adalah `kali` dan groupnya adalah `kali`.
user yang dapat mengubah ownership file/folder adalh superuser, sehingga untuk mengubah kepemilikan file/folder, dapat menggunakan perintah `sudo chown user:group nama-file-atau-folder`.