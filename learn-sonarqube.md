### A. Konsep Dasar SonarQube
SonarQube adalah platform open-source yang digunakan untuk melakukan analisis code guna kerentanan dalam source code. Dengan integrasi SonarQube ke dalam pipeline CI/CD (salah satunya GitHub Actions), tim dapat memastikan kualitas code tetap terjaga sepanjang siklus pengembangan sebuah aplikasi. SonarQube mendukung berbagai bahasa pemrograman dan menyediakan dashboard yang mudah dipahami untuk memantau metrik kualitas code.

### B. Cara Kerja SonarQube
Pada SonarQube, terbagi menjadi 3 komponen berupa:
1. **SonarQube Server**: Tempat di mana analisis code dilakukan dan hasilnya disimpan. Server ini menyediakan dashboard untuk melihat hasil analisis.
2. **SonarQube Scanner**: Tools yang digunakan untuk melakukan analisis code. Scanner ini dapat dijalankan di berbagai lingkungan, termasuk dalam pipeline CI/CD seperti GitHub Actions.
3. **Dashboard SonarQube**: Antarmuka web yang menampilkan hasil analisis code, termasuk metrik kualitas, kerentanan, dan rekomendasi perbaikan.

Jadi, alur kerja SonarQube pada pipeline CI/CD Github Actions bermula dari source code yang ada di repository, kemudian pada saat pipeline dijalankan, SonarQube Scanner akan melakukan analisis terhadap source code tersebut (analisis berjalan pada runner github actions). Hasil analisis akan dikirim ke SonarQube Server (yang pada penginstallanya memerlukan database terlebih dahulu), yang kemudian dapat diakses melalui Dashboard SonarQube untuk melihat metrik kualitas dan kerentanan dalam code.

Diagram kerja SonarQube dalam pipeline github actions:

### B. Menggunakan SonarQube dengan GitHub Actions Reusable Workflows
Prerequisite:
- Memiliki instance SonarQube yang berjalan (dalam percobaan ini, digunakan SonarQube Community Edition yang di-hosting sendiri dengan instalasi dengan docker).

Langkah-langkah percobaan:
1. **Membuat Reusable Workflow untuk SonarQube Analysis**
   Buat repository bernama `cicd-reusable` pada organization `solutionlabs-group`. Pada branch `dev/ade`, buat file workflow `.github/workflows/reusable.yml` dengan isi sebagai berikut:

   ```yaml
    name: Reusable

    on:
    workflow_call:
        secrets:
        SONAR_TOKEN:
            required: true
        SONAR_HOST_URL:
            required: true

    jobs:
    scan:
        name: Code Scan
        runs-on: solutionlabs

        steps:
        - name: Checkout repository
            uses: actions/checkout@v4

        - name: Set up Node.js
            uses: actions/setup-node@v4
            with:
            node-version: '22.x'

        - name: Install dependencies
            run: npm install
        
        #npm install terlebih dahulu agar sonarqube dapat melakukan scan pada dependencies

        - name: Run SonarQube Scan
            uses: sonarsource/sonarqube-scan-action@v3
            env:
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
   ```

2. **Membuat Global Analysis Token**
   - Masuk ke SonarQube dan buka halaman "Administration" > "Security" > "Tokens".
   - Buat token baru dengan nama `github-actions`.
   - Salin token tersebut, karena hanya akan ditampilkan sekali.
![enter image description here](https://i.imgur.com/ePqmS0D_d.webp?maxwidth=760&fidelity=grand)
3. **Mengonfigurasi Secrets di GitHub**
    Setelah dilakukan percobaan, Secret `SONAR_TOKEN` dan `SONAR_HOST_URL` tidak dapat dioper ke repository caller. Oleh karena itu, perlu ditambahkan Secret tersebut secara manual di repository yang akan menggunakan workflow ini, yaitu pada repository `magang-devops`.
![enter image description here](https://i.imgur.com/JSzqMRr_d.webp?maxwidth=760&fidelity=grand)
4. **Memanggil Reusable Workflow di Repository Caller**
   Pada repository bernama `magang-devops`. Pada branch `dev/ade`, digunakan file workflow `.github/workflows/deployment.yml` dengan isi sebagai berikut:
   ```yaml
    name: CI/CD Pipeline

    on:
    push:
        branches: [ dev/ade ] 

    jobs:
    call-reusable-scan:
        name: Call SonarQube Scan
        uses: solutionlabs-group/cicd-reusable/.github/workflows/reusable.yml@dev/ade
        secrets: inherit
        
    build:
        needs: call-reusable-scan
        name: Build with NPM
        runs-on: solutionlabs

        steps:
        - name: Checkout repository
            uses: actions/checkout@v4

        - name: Set up Node.js
            uses: actions/setup-node@v4
            with:
            node-version: '22.x'

        - name: Install dependencies
            run: npm install

        - name: Build application
            run: npm run build
   ```

5. **Membuat file properties untuk SonarQube**
    Di repository `magang-devops`, dibuat file `sonar-project.properties` di root directory dengan isi sebagai berikut (untuk project vuejs):
    
    ```
    sonar.projectKey=magang-devops
    sonar.projectName=Magang DevOps Project
    sonar.sources=.
    sonar.sourceEncoding=UTF-8
    sonar.javascript.file.suffixes=.js,.jsx,.mjs,.vue
    ```
    
    File ini berisi konfigurasi proyek untuk SonarQube, termasuk kunci proyek, URL host, token login, sumber kode yang akan dianalisis, dan jalur laporan cakupan.

5. **Hasil Codescan dengan SonarQube**
   Setelah workflow dijalankan, hasil analisis kode dapat dilihat di dashboard SonarQube. seperti gambar berikut:


![enter image description here](https://i.imgur.com/d0Yftnx_d.webp?maxwidth=1520&fidelity=grand)
### C. Kesimpulan dan Catatan
1. Codescanning dengan SonarQube dapat diintegrasikan ke dalam pipeline CI/CD menggunakan reusable workflows di GitHub Actions. Namun, secrets yang bersifat global tidak dapat dioper ke repository caller (dalam hal ini secrets `SONAR_TOKEN` dan `SONAR_HOST_URL`), sehingga perlu ditambahkan secara manual di masing-masing repository yang menggunakan reusable workflow tersebut (kurang tepat untuk dikatakan sebagai reusable).
2. Codescanning dengan SonarQube pada github actions perlu menggunakan runner yang memiliki docker engine.