### 1. Reusable Workflows Github Action
#### A. Konsep dasar
Reusable workflows di GitHub Actions adalah fitur yang dapat membuat satu workflow yang bisa dipanggil oleh banyak repository lain. Dengan cara ini, tidak perlu menulis konfigurasi worflow CI/CD berulang kali di setiap repo. Workflow yang reusable biasanya didefinisikan dengan event `workflow_call`, kemudian bisa menerima parameter berupa inputs maupun secrets dari caller. Manfaat utama adalah konsistensi pipeline antar project, lebih mudah dalam pemeliharaan, dan hanya perlu mengupdate satu tempat jika ada perubahan.
#### B. Komponen Utama
1. **Workflow Caller**: Workflow yang memanggil reusable workflow. Biasanya didefinisikan di repository yang berbeda.
2. **Reusable Workflow**: Workflow yang dapat dipanggil oleh workflow caller. Didefinisikan dengan event `workflow_call`.
3. **Inputs**: Parameter yang dapat diteruskan dari caller ke reusable workflow.
4. **Secrets**: Informasi sensitif yang dapat diteruskan dari caller ke reusable workflow.

### 2. Contoh Penggunaan
#### A. Membuat Reusable Workflow
Repository `cicd-reusable` dibuat pada organization `solutionlabs-group`. Pada branch `dev/ade`, dibuat file workflow `.github/workflows/deployment-build.yml` dengan isi sebagai berikut:

```yaml
name: Build Frontend App

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version to use'
        required: false
        default: '16.x'
        type: string

jobs:
  build:
    name: Build with NPM
    runs-on: solutionlabs

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ inputs.node-version }}

      - name: Install dependencies
        run: npm install

      - name: Build application
        run: npm run build
```
pada workflow di atas, kita mendefinisikan reusable workflow yang bernama "Build Frontend App". workflow ini dapat dipanggil oleh workflow lain dan menerima input `node-version` untuk menentukan versi Node.js yang akan digunakan. workflow tersebut menggunakan runner `solutionlabs`.
Pada repository reusable ini, perlu dilakukan konfigurasi pada tab `Settings > Actions > General` untuk mengizinkan workflow dari repository lain memanggil reusable workflow ini. Pilih opsi `Accessible from repositories in the 'solutionlabs-group' organization`.
![enter image description here](https://i.imgur.com/lnPyRQk_d.webp?maxwidth=760&fidelity=grand)
#### B. Memanggil Reusable Workflow
Dibuat repository bernama `magang-devops`. Pada branch `dev/ade`, dibuat file workflow `.github/workflows/deployment.yml` dengan isi sebagai berikut:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ dev/ade ]

jobs:
  call-reusable-build:
    # call reusable workflow
    uses: solutionlabs-group/cicd-reusable/.github/workflows/deployment-build.yml@dev/ade
    
    # input parameters
    with:
      node-version: '22.x'
```
pada workflow di atas, kita memanggil reusable workflow dari repository `cicd-reusable` pada branch `dev/ade` dan mengirimkan input `node-version` dengan nilai `22.x`. Workflow ini akan berjalan setiap kali ada push ke branch `dev/ade` di repository `magang-devops`.

**Hasil Pipeline pada Repository `magang-devops`:**
![enter image description here](https://i.imgur.com/7WbdyZq_d.webp?maxwidth=1520&fidelity=grand)

### 4. Kesimpulan
Dengan reusable workflows, implementasi CI/CD di GitHub Actions menjadi lebih terstruktur, standar, dan mudah dipelihara. Reusable workflows sangat cocok digunakan pada organisasi dengan banyak repository yang membutuhkan pipeline serupa. Walaupun ada tantangan dalam manajemen dependensi dan secrets, manfaat yang diperoleh jauh lebih besar, terutama dari sisi konsistensi dan efisiensi.