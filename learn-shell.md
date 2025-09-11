### Shell, ZSH, & Shell Script
###### Shell
Shell adalah program yang menjadi penghubung antara pengguna dan sistem operasi. dengan shell, dapat diberikan perintah (command) yang kemudian diterjemahkan untuk dijalankan oleh sistem. jenis-jenis shell dapat berupa sh, bash, zsh, atau fish. shell yang sering digunakan adalah `bash`
###### ZSH
zsh (Z shell) adalah salah satu jenis shell. yang membedakan zsh dengan shell lainnya adalah terdapat fitur tambahan seperti tab-completion, auto-suggest, customisasi prompt dan theme.

salah satu framework untuk zsh adalah ohmyzsh, instalasi ohmyzsh dapat dilakukan dengan:
```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

misal tab-completion:
```sh
➜  ~ ch
chacl                   chat                    chdir                   chgrp                   chown                   chruby_prompt_info    
chage                   chattr                  check-language-support  chmem                   chpasswd                chsh                  
chardet                 chcon                   chfn                    chmod                   chroot                  chvt                  
chardetect              chcpu                   chgpasswd               choom                   chrt    
```
ketika karakter yang diketikkan pada zsh adalah ch dan menekan tombol `tab`, zsh akan memberikan auto-suggest dan tab-completion.

pada folder repository git, zsh dapat menampilkan informasi branch seperti berikut:
```sh
➜  testing git:(dev/ade) ✗ ls -a -l
total 12
drwxrwxr-x  3 ade ade 4096 Sep 11 09:17 .
drwxr-x--- 17 ade ade 4096 Sep 11 09:18 ..
drwxrwxr-x  7 ade ade 4096 Sep 11 09:17 .git
-rw-rw-r--  1 ade ade    0 Sep 11 09:17 testing-zsh.sh
-rw-rw-r--  1 ade ade    0 Sep 11 09:17 text-again.txt
-rw-rw-r--  1 ade ade    0 Sep 11 09:17 text-manipulation.txt
```

###### Shell Script
Shell script adalah file teks yang berisi sekumpulan perintah shell yang biasanya dalam extension file `.sh` yang dijalankan otomatis, bukan diketik satu per satu.

contoh shell script `testing-zsh.sh`:
```sh
#!/bin/zsh
echo "coba shell script"
touch "new-text.txt"
echo "selesai"
```
file tersebut dijalankan dengan `./testing-zsh.sh`. baris pertama file tersebut adalah jenis shell yang akan menjalankan kumpulan perintah dalam file sh tersebut. pada contoh tersebut adalah `#!/bin/zsh`, sehingga yang menjalankan script tersebut adalah zsh.
### Text Manipulation
1. cat
cat digunakan untuk menampilkan text yang barada pada sebuah textfile. misal:
```sh
➜  ~ cat ade.txt        
"this is a text inside ade.txt"
```
2. nano atau vim
nano atau vim merupakan perintah untuk membuka editor text (biak nano maupun vim), digunakan untuk membuat, melihat, dan mengedit file text.

3. grep
perintah `grep` digunakan untuk mencari pola/teks pada sebuah file atau output perintah lain.
```sh
➜  ~ grep "ade" number.txt
35. ade
```
contoh mencari pola/texks pada output perintah lain:
```sh
➜  ~ docker ps -a | grep wordpress
25efe47facd6   wordpress                     "docker-entrypoint.s…"   40 hours ago   Exited (0) 24 hours ago                                                   my-wordpress-site
```
4. sed
perintah sed digunakan untuk melakukan manipulasi teks secara otomatis pada aliran data atau file, seperti mencari dan mengganti kata, menghapus baris tertentu, menampilkan baris tertentu, maupun menyisipkan teks baru, tanpa perlu membuka file di editor.
```sh
➜  ~ sed -i 's/ade/kurniawan/g' number.txt
➜  ~ cat number.txt
3. abc
6. def
2. ghi
7. jkl
1. mno
9. pqr
19. kurniawan
35. kurniawan
39.
```
pada contoh perintah sed tersebut, dengan argumen `-i`, dilakukan edit file langsung tanpa membuat salinan baru, lalu instruksi `s/ade/kurniawan/` memberi tahu sed untuk mengganti teks `ade` dengan `kurniawan`, dan tambahan `g` di akhir membuat penggantian dilakukan secara global di setiap baris, sehingga jika ada lebih dari satu kata ade dalam satu baris, semuanya akan diganti sekaligus.

### Networking Tools
```sh
ping <ip-address/hostname>
```
perintah ping digunakan untuk menguji apakah ip address/hostname tujuan dapat dijangkau. dapat juga menampilkan reply response time.
```sh
host <ip-address/hostname>
```
perintah host digunakan untuk melakukan DNS lookup, yaitu menerjemahkan hostname menjadi alamat IP atau sebaliknya.

```sh
mtr <ip-address/hostname>
```
perintah mtr (my traceroute) digunakan untuk menggabungkan fungsi traceroute dan ping, sehingga bisa melihat jalur paket dan kualitas koneksi secara realtime.

```sh
whois <ip-address/hostname>
```
perintah whois digunakan untuk mendapatkan informasi kepemilikan domain atau ip address, termasuk registrar, kontak admin, dan detail pendaftaran.

```sh
dig <hostname>
```
perintah dig (Domain Information Groper) digunakan untuk query DNS secara detail, misalnya untuk melihat A record, MX record, atau NS record dari sebuah domain.

```sh
curl <ip-address/hostname:port>
```
perintah curl digunakan untuk mengirim permintaan HTTP/HTTPS ke server dan menampilkan responsenya. Cocok untuk menguji API atau konektivitas web service pada port tertentu.

```sh
ifconfig
```
perintah ifconfig digunakan untuk menampilkan dan mengatur konfigurasi jaringan (alamat IP, netmask, interface)

```sh
ip a
```
perintah ip a (atau ip addr) adalah perintah modern pengganti ifconfig untuk menampilkan alamat IP dan status semua interface jaringan.

```sh
ssh <user@ip-address/hostname>
```
perintah ssh digunakan untuk melakukan remote login ke server/komputer lain secara aman melalui protokol SSH.

### Process Monitoring
1. `ps aux` => digunakan untuk menampilkan daftar proses berjalan
```sh
➜  ~ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.1  21940 12512 ?        Ss   11:04   0:02 /sbin/init
root           2  0.0  0.0   3060  1664 ?        Sl   11:04   0:00 /init
root           8  0.0  0.0   3060  1792 ?        Sl   11:04   0:00 plan9 --control-socket 7 --log-level 4 --serv
root          44  0.0  0.2  50356 15744 ?        S<s  11:04   0:00 /usr/lib/systemd/systemd-journald
root          90  0.0  0.0  25400  6400 ?        Ss   11:04   0:00 /usr/lib/systemd/systemd-udevd
root         110  0.1  0.1 676284 12448 ?        Ssl  11:04   0:05 snapfuse /var/lib/snapd/snaps/snapd_25202.sna
root         111  0.0  0.0 152936  1284 ?        Ssl  11:04   0:00 snapfuse /var/lib/snapd/snaps/snapd_24792.sna
root         116  0.3  0.1 526756 14044 ?        Ssl  11:04   0:15 snapfuse /var/lib/snapd/snaps/docker_3265.sna
root         117  0.0  0.1 526888 11472 ?        Ssl  11:04   0:00 snapfuse /var/lib/snapd/snaps/core22_2111.sna
root         118  0.0  0.0 152936  1284 ?        Ssl  11:04   0:00 snapfuse /var/lib/snapd/snaps/core22_2045.sna
systemd+     196  0.0  0.1  21584 12672 ?        Ss   11:04   0:00 /usr/lib/systemd/systemd-resolved
systemd+     201  0.0  0.0  91024  7680 ?        Ssl  11:04   0:00 /usr/lib/systemd/systemd-timesyncd
root         219  0.0  0.0   4236  2560 ?        Ss   11:04   0:00 /usr/sbin/cron -f -P
message+     220  0.0  0.0   9592  4736 ?        Ss   11:04   0:00 @dbus-daemon --system --address=systemd: --no
root         227  0.0  0.4 2514632 38644 ?       Ssl  11:04   0:01 /usr/lib/snapd/snapd
root         228  0.0  0.1  17960  8192 ?        Ss   11:04   0:00 /usr/lib/systemd/systemd-logind
root         230  0.0  0.1 1756096 12544 ?       Ssl  11:04   0:00 /usr/libexec/wsl-pro-service -vv
syslog       237  0.0  0.0 222508  5376 ?        Ssl  11:04   0:00 /usr/sbin/rsyslogd -n -iNONE
root         240  0.0  0.0   3160  1920 hvc0     Ss+  11:04   0:00 /sbin/agetty -o -p -- \u --noclear --keep-bau
root         257  0.0  0.0   3116  1792 tty1     Ss+  11:04   0:00 /sbin/agetty -o -p -- \u --noclear - linux
root         271  0.0  0.2 107012 22520 ?        Ssl  11:04   0:00 /usr/bin/python3 /usr/share/unattended-upgrad
root         427  0.0  1.0 2491440 77776 ?       Ssl  11:04   0:02 dockerd --group docker --exec-root=/run/snap.
root         495  0.0  0.0   3068   896 ?        Ss   11:04   0:00 /init
root         496  0.0  0.0   3084  1152 ?        S    11:04   0:00 /init
ade          497  0.0  0.0   6072  4992 pts/0    Ss+  11:04   0:00 -bash
root         498  0.0  0.0   6820  4096 pts/1    Ss   11:04   0:00 /bin/login -f
ade          578  0.0  0.1  20380 11264 ?        Ss   11:04   0:00 /usr/lib/systemd/systemd --user
ade          579  0.0  0.0  21152  3520 ?        S    11:04   0:00 (sd-pam)
ade          595  0.0  0.0   6072  5248 pts/1    S+   11:04   0:00 -bash
root         621  0.1  0.6 2171928 48364 ?       Ssl  11:04   0:07 containerd --config /run/snap.docker/containe
root         638  0.0  0.0   3068   896 ?        Ss   11:04   0:00 /init
root         639  0.0  0.0   3084  1156 ?        S    11:04   0:00 /init
ade          641  0.0  0.0   6852  5632 pts/2    Ss   11:04   0:00 -bash
polkitd     1128  0.0  0.1 308164  7808 ?        Ssl  11:05   0:00 /usr/lib/polkit-1/polkitd --no-debug
ade         1320  0.0  0.0   9292  4864 ?        Ss   11:06   0:00 /usr/bin/dbus-daemon --session --address=syst
root        1642  0.0  0.0   2800  1792 ?        Ss   11:18   0:00 /bin/sh /usr/lib/apt/apt.systemd.daily update
root        1646  0.0  0.0   2800  1792 ?        S    11:18   0:00 /bin/sh /usr/lib/apt/apt.systemd.daily lock_i
root        1686  0.0  0.1  15808 10496 ?        S    11:18   0:01 apt-get -qq -y update
_apt        1695  0.0  0.1  20292 10368 ?        S    11:18   0:00 /usr/lib/apt/methods/http
_apt        1696  0.0  0.1  20292 10240 ?        S    11:18   0:00 /usr/lib/apt/methods/http
_apt        1706  0.0  0.0  13096  6656 ?        S    11:18   0:00 /usr/lib/apt/methods/gpgv
_apt        1853  0.0  0.1  21480 14080 ?        S    11:19   0:00 /usr/lib/apt/methods/store
root        2416  0.0  0.1 1238104 14544 ?       Sl   11:23   0:00 /snap/docker/3265/bin/containerd-shim-runc-v2
root        2438  0.0  0.2  28160 20548 ?        Ss   11:23   0:01 /usr/local/bin/python3.9 /usr/local/bin/gunic
root        2483  0.0  0.0 1745440 4224 ?        Sl   11:23   0:00 /snap/docker/3265/bin/docker-proxy -proto tcp
root        2494  0.0  0.0 1745440 4480 ?        Sl   11:23   0:00 /snap/docker/3265/bin/docker-proxy -proto tcp
root        2509  0.0  0.4  42984 33316 ?        S    11:23   0:00 /usr/local/bin/python3.9 /usr/local/bin/gunic
ade         2610  0.0  0.1  11868  8136 pts/2    S    11:35   0:00 zsh
ade         3186  100  0.0   8284  4096 pts/2    R+   12:30   0:00 ps aux
```
2. top => digunakan untuk menampilkan daftar proses berjalan secara realtime
```sh
➜  ~ top
top - 12:31:23 up  1:26,  1 user,  load average: 0.01, 0.01, 0.00
Tasks:  49 total,   1 running,  48 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7558.2 total,   6607.6 free,    673.5 used,    439.4 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6884.7 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    621 root      20   0 2171928  48364  34560 S   0.3   0.6   0:07.14 containerd
      1 root      20   0   21940  12512   9312 S   0.0   0.2   0:02.66 systemd
      2 root      20   0    3060   1664   1664 S   0.0   0.0   0:00.01 init-systemd(Ub
      8 root      20   0    3060   1792   1792 S   0.0   0.0   0:00.00 init
     44 root      19  -1   50356  15744  14848 S   0.0   0.2   0:00.61 systemd-journal
     90 root      20   0   25400   6400   4864 S   0.0   0.1   0:00.44 systemd-udevd
    110 root      20   0  676284  12448   1408 S   0.0   0.2   0:05.43 snapfuse
    111 root      20   0  152936   1284   1152 S   0.0   0.0   0:00.00 snapfuse
    116 root      20   0  526756  14044   1280 S   0.0   0.2   0:15.73 snapfuse
    117 root      20   0  526888  11472   1280 S   0.0   0.1   0:00.80 snapfuse
    118 root      20   0  152936   1284   1152 S   0.0   0.0   0:00.00 snapfuse
    196 systemd+  20   0   21584  12672  10496 S   0.0   0.2   0:00.17 systemd-resolve
    201 systemd+  20   0   91024   7680   6784 S   0.0   0.1   0:00.27 systemd-timesyn
    219 root      20   0    4236   2560   2432 S   0.0   0.0   0:00.02 cron
    220 message+  20   0    9592   4736   4352 S   0.0   0.1   0:00.21 dbus-daemon
    227 root      20   0 2514632  38644  24064 S   0.0   0.5   0:01.31 snapd
    228 root      20   0   17960   8192   7296 S   0.0   0.1   0:00.14 systemd-logind
    230 root      20   0 1756096  12544  10240 S   0.0   0.2   0:00.23 wsl-pro-service
    237 syslog    20   0  222508   5376   4480 S   0.0   0.1   0:00.13 rsyslogd
    240 root      20   0    3160   1920   1792 S   0.0   0.0   0:00.01 agetty
    257 root      20   0    3116   1792   1664 S   0.0   0.0   0:00.01 agetty   
```
3. `htop` => versi lebih interaktif dan berwarna dari top
4. `pgrep` => digunakan untuk mencari PID berdasarkan nama proses
```sh
➜  ~ pgrep docker
427
2483
2494
```
5. `kill` => menghentikan proses berdasarkan PID
```sh
➜  ~ sudo kill 427
```
