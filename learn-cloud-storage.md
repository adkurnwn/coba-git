### Cloud Storage
Pada Cloud Storage, terdapat storage class yaitu pembagian jenis object storage berdasarkan frekuensi aksesnya:

**A. Standard / Hot Storage**
jenis storage class default dimana data yang tersimpan akan diakses secara sering dan cepat, seperti penyimpanan untuk website yang aktif, cdn, dsb. biaya storage class ini paling mahal dibanding storage class lain. contoh layanan yang menggunakan jenis storage class ini adalah `Amazon S3 Standard (S3 Standard)`.

**B. Infrequent Access (IA)**
Jenis storage class untuk data yang lebih jarang diakses, namun tetap membutuhkan kecepatan akses yang cepat saat diperlukan. digunakan untuk data jangka panjang, backup, atau file disaster recovery. Biaya penyimpanannya lebih murah dari Standard, namun ada biaya tambahan untuk setiap kali data diakses. Contoh layanan yang menggunakan jenis storage class ini adalah `Amazon S3 Standard-Infrequent Access (S3 Standard-IA)`.

**C. Archive Storage**
Jenis storage class berbiaya sangat rendah yang dirancang untuk pengarsipan data jangka panjang (bertahun-tahun) yang sangat jarang diakses. Proses pengambilan (retrieval) datanya tidak instan dan bisa memakan waktu beberapa menit hingga beberapa jam. Ideal untuk arsip data legal, medis, atau data mentah proyek yang sudah selesai. Biaya penyimpanannya sangat murah, namun biaya pengambilannya lebih mahal. Contoh layanan yang menggunakan jenis storage class ini adalah `Amazon S3 Glacier Flexible Retrieval`.

**D. Deep Archive Storage**
Jenis storage class dengan biaya paling murah yang ditujukan untuk data yang hampir tidak pernah diakses sama sekali. Ini adalah solusi untuk menyimpan data dalam jumlah masif selama bertahun-tahun untuk keperluan regulasi atau preservasi data. Waktu pengambilannya adalah yang paling lama, bisa mencapai 12 jam atau lebih. Contoh layanan yang menggunakan jenis storage class ini adalah `Amazon S3 Glacier Deep Archive`.
![enter image description here](https://pbs.twimg.com/media/GFwTqxrakAAjWiw.jpg:large)
#### 1. Object Storage
Object storage menyimpan data di dalamnya sebagai object. Object storage memiliki konsep metadata dan dilengkapi dengan pengidentifikasi unik yang bisa digunakan untuk mengakses dan mencari setiap unit data. Setiap unit data disimpan di dalam satu lokasi sehingga bisa dicari dan diakses dengan mudah. object storage mampu menyimpan data yang sangat besar. contoh layanan dari cloud provider: `AWS S3, Google Cloud Storage, Azure Blob Storage`.

#### 2. Block Storage
Block storage memecah data menjadi bagian-bagian terpisah yang berukuran sama, yang disebut blok. Setiap blok diberi alamat unik tetapi tidak memiliki metadata tambahan seperti pada object storage. Sistem operasi mengelola blok-blok ini dan menggabungkannya kembali menjadi file utuh saat data tersebut diminta oleh pengguna. Karena sifatnya yang cepat dan efisien, block storage ideal untuk aplikasi yang membutuhkan performa tinggi dan akses data latensi rendah, seperti database, mesin virtual (VM), dan sistem transaksional.contoh layanan dari cloud provider: `AWS EBS (Elastic Block Store), Google Persistent Disk `.

#### 3. File Storage
File storage, atau penyimpanan berbasis file, adalah metode yang paling umum dan mudah dikenali. Data disimpan dalam bentuk file, dan file-file tersebut diatur dalam folder dan subfolder dengan struktur hierarki. Untuk mengakses sebuah file, sistem harus menavigasi melalui path atau jalur dari direktori utama hingga ke lokasi file tersebut. Metode ini banyak digunakan pada hard drive komputer pribadi dan perangkat Network Attached Storage (NAS) untuk berbagi dokumen dengan mudah. bagus digunakan untuk menyimpan data/file berukuran kecil. contoh layanan dari cloud: `AWS EFS (Elastic File System)`.

