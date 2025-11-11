### Velero Backup and Restore
Velero adalah tools yang digunakan untuk melakukan backup dan restore pada cluster k8s. Velero dapat digunakan untuk melindungi data dan aplikasi yang berjalan di dalam cluster k8s dengan cara membuat salinan data secara berkala dan menyimpannya di object storage.

#### 1. Instalasi Velero
Pada instalasi, Velero diinstal pada CLI terlebih dahulu dengan perintah:
```sh
brew install velero
```
Setelah itu, pastikan object storage sudah dibuat dan file `credentials-velero` sudah berisi akses key dan secret key untuk mengakses object storage.
Contoh isi file `credentials-velero`:
```txt
[default]
aws_access_key_id = 5XxfyJet8Egypedtu1CW
aws_secret_access_key = secretaccesskey
```
Velero diinstal pada cluster k8s dengan perintah:
```sh
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket velero \
  --secret-file ./credentials-velero \
  --use-node-agent \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=https://s3.adekurniawan.me \
  --snapshot-location-config region=minio
```
pada perintah tersebut digunakan s3 minio sebagai object storage untuk menyimpan backup pada bucket `velero`, dengan force path style agar dapat digunakan pada s3 compatible storage seperti minio.

perintah tersebut juga pada dasarnya membuat yaml dan langsung dilakukan apply pada cluster k8s, sehingga terbentuk namespace `velero` yang berisi resource-resource yang dibutuhkan oleh Velero.
```sh
kubectl get all -n velero
NAME                          READY   STATUS    RESTARTS      AGE
pod/node-agent-pvz44          1/1     Running   1             24h
pod/velero-75fdbc58dc-g2l8s   1/1     Running   1 (19h ago)   29h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-agent   1         1         1       1            1           kubernetes.io/os=linux   29h

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/velero   1/1     1            1           29h

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/velero-75fdbc58dc   1         1         1       29h

```
jika tidak langsung diapply dapat menambahkan flag `--dry-run -o yaml` untuk melihat hasil yaml yang akan diapply.

#### 2. Membuat Backup pada sebuah Namespace
Backup dapat dilakukan pada sebuah namespace dengan perintah:
```sh
velero backup create demo-app-backup --include-namespaces demo-app
```
Perintah tersebut akan membuat backup pada namespace `demo-app` dan menyimpannya pada object storage yang sudah dikonfigurasi sebelumnya. Untuk melihat status backup dapat menggunakan perintah:
```sh
velero backup get
NAME              STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
demo-app-backup   Completed   0        0          2025-11-10 10:34:59 +0700 WIB   28d       default            <none>
```
detail backup dapat dilihat dengan perintah:
```sh
 velero backup describe demo-app-backup
Name:         demo-app-backup
Namespace:    velero
Labels:       velero.io/storage-location=default
Annotations:  velero.io/resource-timeout=10m0s
              velero.io/source-cluster-k8s-gitversion=v1.34.0
              velero.io/source-cluster-k8s-major-version=1
              velero.io/source-cluster-k8s-minor-version=34

Phase:  Completed


Namespaces:
  Included:  demo-app
  Excluded:  <none>

Resources:
  Included cluster-scoped:    <none>
  Excluded cluster-scoped:    volumesnapshotcontents.snapshot.storage.k8s.io
  Included namespace-scoped:  *
  Excluded namespace-scoped:  volumesnapshots.snapshot.storage.k8s.io

Label selector:  <none>

Or label selector:  <none>

Storage Location:  default

Velero-Native Snapshot PVs:    auto
File System Backup (Default):  false
Snapshot Move Data:            false
Data Mover:                    velero

TTL:  720h0m0s

CSISnapshotTimeout:    10m0s
ItemOperationTimeout:  4h0m0s

Hooks:  <none>

Backup Format Version:  1.1.0

Started:    2025-11-10 10:34:59 +0700 WIB
Completed:  2025-11-10 10:35:04 +0700 WIB

Expiration:  2025-12-10 10:34:59 +0700 WIB

Total items to be backed up:  21
Items backed up:              21

Backup Volumes:
  Velero-Native Snapshots: <none included>

  CSI Snapshots: <none included>

  Pod Volume Backups: <none included>

HooksAttempted:  0
HooksFailed:     0

```
contoh hasil backup pada storage object:
![enter image description here](https://i.imgur.com/7lRjW6F_d.webp?maxwidth=1520&fidelity=grand)
#### 3. Melakukan Restore dari Backup
Restore dapat dilakukan pada server cluster k8s yang sudah terinstal velero dan memiliki akses ke object storage yang sama, pada percobaan ini dilakukan penghapusan namespace yang akan di-restore. Restore dilakukan dengan perintah:
```sh
velero restore create --from-backup demo-app-backup
```

hasil perintah tersebut:
```sh
ade@AdeHomeLabServer:~/k-deployment/deployment$ velero restore create --from-backup demo-app-backup
Restore request "demo-app-backup-20251111161355" submitted successfully.
Run `velero restore describe demo-app-backup-20251111161355` or `velero restore logs demo-app-backup-20251111161355` for more details.
ade@AdeHomeLabServer:~/k-deployment/deployment$ velero restore describe demo-app-backup-20251111161355
Name:         demo-app-backup-20251111161355
Namespace:    velero
Labels:       <none>
Annotations:  <none>

Phase:                       Completed
Total items to be restored:  21
Items restored:              21

Started:    2025-11-11 16:13:55 +0700 WIB
Completed:  2025-11-11 16:14:26 +0700 WIB

Warnings:
  Velero:     <none>
  Cluster:    <none>
  Namespaces:
    demo-app:  could not restore, ConfigMap:kube-root-ca.crt already exists. Warning: the in-cluster version is different than the backed-up version

Backup:  demo-app-backup

Namespaces:
  Included:  all namespaces found in the backup
  Excluded:  <none>

Resources:
  Included:        *
  Excluded:        nodes, events, events.events.k8s.io, backups.velero.io, restores.velero.io, resticrepositories.velero.io, csinodes.storage.k8s.io, volumeattachments.storage.k8s.io, backuprepositories.velero.io
  Cluster-scoped:  auto

Namespace mappings:  <none>

Label selector:  <none>

Or label selector:  <none>

Restore PVs:  auto

CSI Snapshot Restores: <none included>

Existing Resource Policy:   <none>
ItemOperationTimeout:       4h0m0s

Preserve Service NodePorts:  auto

Uploader config:


HooksAttempted:   0
HooksFailed:      0
```
setelah proses restore selesai, namespace yang dilakukan restore akan kembali seperti semula seperti ketika dilakukan backup.

![enter image description here](https://i.imgur.com/Yhp1e3k_d.webp?maxwidth=1520&fidelity=grand)