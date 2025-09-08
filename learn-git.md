
# Learn using git

```sh
git clone git@github.com:adkurnwn/coba-git.git
```
git clone digunakan untuk mengambil code dari repository yang sudah dibuat

```sh
git checkout -b dev/ade
```
git checkout -b digunakan untuk membuat branch baru, dev/ade merupakan nama branchnya



```sh
git branch
```
git branch digunakan untuk menampilkan daftar branch. juga dapat digunakan untuk membuat branch baru dengan menambahkan argumen nama branch di belakangnya, misalnya `git branch dev/john`. selain itu, command ini dapat menghapus branch dengan `git branch -d nama-branch` atau mengganti nama branch dengan `git branch -m nama-baru`

```sh
git checkout main
```
git checkout main digunakan untuk berpindah ke branch `main`

```sh
git pull origin main
```
git pull origin main digunakan untuk mengambil update terbaru dari branch `main` pada remote origin, lalu langsung menggabungkannya ke branch `main` local

```sh
git pull
```
git pull digunakan untuk mengambil (fetch) pembaruan terbaru dari remote repository dan langsung menggabungkannya (merge) ke local branch yang sedang aktif.

```sh
git rm --cached
```
git rm --cached digunakan untuk berhenti melacak file tertentu di git, tetapi file tersebut tetap ada di direktori local. dapat dipakai untuk menghapus file yang seharusnya dimasukkan ke dalam `.gitignore`. penggunaannya adalah berupa `git rm --cached <namafile>`.

```sh
git status
```
git status digunakan untuk menampilkan status repository, termasuk file yang sudah diubah, file baru yang belum dilacak, serta file yang sudah berada di staging area dan siap di-commit.

```sh
git add .
```
git add . digunakan untuk menambahkan semua perubahan di direktori saat ini (termasuk subdir) ke staging area, termasuk file baru, file yang diubah, dan file yang dihapus.

```sh
git add -u
```
git add -u digunakan untuk menambahkan perubahan hanya pada file yang sudah dilacak git sebelumnya, misalnya file yang diupdate atau dihapus, tetapi tidak termasuk file baru.

```sh
git add --all
```
git add --all digunakan untuk menambahkan semua jenis perubahan pada repository ke staging area, baik file baru, file yang dimodifikasi, maupun file yang dihapus meskipun berada pada direktori root repository.

```sh
git commit -m -> sebutkan perintahnya
```
git commit -m "pesan" digunakan untuk menyimpan perubahan yang sudah di-stage ke dalam riwayat git dengan menambahkan pesan singkat sebagai keterangan commit. argumen `-m` artinya dalam commit terdapat message.

```sh
git push
```
git push digunakan untuk mengirim commit dari repository local ke remote repository. jika branch belum dikaitkan ke remote, digunakan `git push -u origin nama-branch` untuk sekaligus menetapkan branch tujuan.

```sh
git branch -r
```
git branch -r digunakan untuk menampilkan daftar branch yang ada di remote repository. misalnya origin/main, origin/dev/ade, origin/dev/johan

```sh
git fetch -p
```
git fetch -p digunakan untuk mengambil update terbaru dari remote tanpa langsung menggabungkannya ke branch lokal. argumen -p (prune) berfungsi untuk menghapus referensi branch remote yang sudah dihapus dari server (sehingga branch di lokal ikut terhapus jika branch pada remote sudah dihapus).

```sh
git pull --rebase
```
git pull --rebase digunakan untuk mengambil update dari remote dan menempatkan commit local di atas commit terbaru remote. cara ini membuat riwayat commit lebih rapi karena tidak menambah merge commit baru.