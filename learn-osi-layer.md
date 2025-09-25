### OSI Layers
Cara kerja OSI:
1. Informasi berawal dari layer Application. Informasi kemudian melewati layer presentation dan layer session. Pada tahap ini biasanya belum dilakukan transformasi data. Informasi yang melalui ketiga layer ini disebut PDU (Protocol Data Unit).
2. Setelah melewati layer session, informasi diteruskan ke layer transport. Pada layer ini, informasi dipecah menjadi beberapa segment dan ditambahkan header yang berisi informasi penting seperti nomor port asal dan tujuan. Informasi pada layer transport disebut segment.
3. Segmen dari layer transport kemudian diteruskan ke layer network. Pada layer ini, segmen dibungkus dalam packet dengan menambahkan header yang berisi informasi seperti alamat IP asal dan tujuan. Informasi pada layer network disebut packet.
4. Paket dari layer network kemudian diteruskan ke layer data link. Pada layer ini, paket dibungkus dalam frame dengan menambahkan header dan trailer yang berisi informasi seperti alamat MAC asal dan tujuan, serta informasi deteksi kesalahan. Informasi pada layer data link disebut frame.
5. Frame dari layer data link kemudian diteruskan ke layer physical. Pada layer ini, frame diubah menjadi sinyal fisik (seperti sinyal listrik, cahaya, atau gelombang radio) yang dapat ditransmisikan melalui media fisik seperti kabel atau udara. Informasi pada layer physical disebut bit.

![enter image description here](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*A5tXuXTWXdIrUChT.jpg)
#### Enkapsulasi Data dalam OSI
Proses enkapsulasi adalah mekanisme pembungkusan data saat melewati setiap layer OSI. Setiap layer menambahkan header (dan kadang trailer) yang berisi informasi kontrol yang diperlukan untuk komunikasi. Berikut adalah proses enkapsulasi data dalam OSI:
#### 1. Physical Layer
Physical layer adalah lapisan pertama dari model OSI yang bertanggung jawab untuk transmisi fisik data melalui media komunikasi. Lapisan ini mencakup aspek-aspek seperti kabel, konektor, sinyal listrik, dan perangkat keras lainnya yang digunakan untuk mengirimkan data.
#### 2. Data Link Layer
Data Link Layer adalah lapisan kedua dari model OSI yang bertanggung jawab untuk pengiriman data antar perangkat dalam jaringan lokal. Lapisan ini mengatur framing, pengalamatan fisik (MAC address), dan deteksi serta koreksi kesalahan pada tingkat data link.
#### 3. Network Layer
Network Layer adalah lapisan ketiga dari model OSI yang bertanggung jawab untuk pengalamatan dan pengiriman paket data antar jaringan. Lapisan ini mengatur routing, pengalamatan logis (IP address), dan pengendalian lalu lintas data.
#### 4. Transport Layer
Transport Layer adalah lapisan keempat dari model OSI yang bertanggung jawab untuk pengiriman data end-to-end antara aplikasi. Lapisan ini mengatur segmentasi, pengendalian aliran, dan pemulihan kesalahan.
#### 5. Session Layer
Session Layer adalah lapisan kelima dari model OSI yang bertanggung jawab untuk mengelola sesi komunikasi antara aplikasi. Lapisan ini mengatur pembentukan, pemeliharaan, dan penghentian sesi.

#### 6. Presentation Layer
Presentation Layer adalah lapisan keenam dari model OSI yang bertanggung jawab untuk format data dan representasi. Lapisan ini mengatur konversi data, enkripsi, dan kompresi agar dapat dipahami oleh aplikasi.

#### 7. Application Layer
Application Layer adalah lapisan ketujuh dari model OSI yang bertanggung jawab untuk antarmuka pengguna dan interaksi aplikasi. Lapisan ini menyediakan layanan jaringan langsung kepada pengguna dan aplikasi, serta mengatur protokol yang digunakan untuk komunikasi antar aplikasi.