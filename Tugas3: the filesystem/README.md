# Bab 5: Sistem Berkas

![ikon-sistem-berkas](https://miro.medium.com/v2/resize:fit:752/1*quw0WvsLLCxad3WC6fjQ1Q.png)

Tujuan dasar dari sistem berkas adalah untuk merepresentasikan dan mengorganisir sumber daya penyimpanan sistem.

Sistem berkas dapat dianggap sebagai komponen yang terdiri dari empat bagian utama:

1. Sebuah ruang nama (namespace) - cara untuk menamai dan mengorganisir dalam hierarki
2. Sebuah API - sekumpulan system call untuk menavigasi dan memanipulasi objek
3. Model keamanan - skema untuk melindungi, menyembunyikan, dan berbagi objek
4. Sebuah implementasi - perangkat lunak yang menghubungkan model logis dengan perangkat keras

Sistem berkas berbasis disk yang dominan adalah ext4, XFS, dan UFS, bersama dengan ZFS dari Oracle dan Btrfs. Banyak sistem berkas lain juga tersedia, termasuk VxFS dari Veritas dan JFS dari IBM.

Kita juga memiliki beberapa sistem berkas asing seperti FAT dan NTFS yang digunakan oleh Windows dan sistem berkas ISO 9660 yang digunakan oleh CD dan DVD.

Sebagian besar sistem berkas modern berusaha mengimplementasikan fungsionalitas sistem berkas tradisional dengan cara yang lebih cepat dan lebih andal, atau menambahkan fitur tambahan sebagai lapisan di atas semantik sistem berkas standar.

## Nama Path

Kata "folder" hanyalah kebocoran linguistik dari dunia Windows dan macOS. Ini berarti hal yang sama dengan **direktori**, yang lebih teknis. Jangan gunakan istilah folder dalam konteks teknis kecuali Anda siap menerima tatapan aneh!!

Daftar direktori yang mengarah ke sebuah berkas disebut **pathname** (nama path). Pathname adalah string yang menjelaskan lokasi berkas dalam hierarki sistem berkas.
Pathname dapat berupa **absolut** (misalnya `/home/username/file.txt`) atau **relatif** (misalnya `./file.txt`).

## Pemasangan dan Pelepasan Sistem Berkas (Mounting dan Unmounting)

Sistem berkas terdiri dari potongan-potongan kecil--juga disebut "sistem berkas"--yang masing-masing terdiri dari satu direktori dan subdirektori serta berkas-berkasnya. Kita menggunakan istilah **pohon berkas** untuk merujuk pada tata letak keseluruhan dan menyimpan kata **sistem berkas** untuk cabang-cabang yang terpasang pada pohon.

Dalam kebanyakan situasi, sistem berkas dipasang ke pohon dengan perintah `mount`. Perintah `mount` memetakan direktori dalam pohon berkas yang ada, disebut **titik pemasangan** (mount point), ke root dari sistem berkas baru.

Contoh:

```bash
# Memasang sistem berkas di /dev/sda4 ke /users
mount /dev/sda4 /users
```

Linux memiliki opsi pelepasan malas (**umount -l**) yang menghapus sistem berkas dari hierarki penamaan tetapi tidak benar-benar melepaskannya sampai sistem berkas tersebut tidak lagi digunakan.

**umount -f** adalah pelepasan paksa, yang berguna ketika sistem berkas sedang sibuk.

Alih-alih menggunakan **umount -f**, Anda dapat menggunakan **lsof** atau **fuser** untuk mengetahui proses mana yang menggunakan sistem berkas dan kemudian mematikannya.

Contoh:

```bash
# Mencari tahu proses mana yang menggunakan sistem berkas

abdou@debian:~$ lsof /home/abdou

COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
bash     1000 abdou  cwd    DIR    8,1     4096  131073 /home/abdou
bash     1000 abdou  rtd    DIR    8,1     4096  131073 /home/abdou
bash     1000 abdou  txt    REG    8,1   103752  131072 /bin/bash
bash     1000 abdou  mem    REG    8,1  1848400  131074 /lib/x86_64-linux-gnu/libc-2.28.so
bash     1000 abdou  mem    REG    8,1   170864  131075 /lib/x86_64-linux-gnu/ld-2.28.so
code     1234 abdou  cwd    DIR    8,1     4096  131073 /home/abdou
msedge   5678 abdou  cwd    DIR    8,1     4096  131073 /home/abdou
```

Untuk menyelidiki proses yang menggunakan sistem berkas, Anda dapat menggunakan perintah **ps**.

Contoh:

```bash
# Menyelidiki proses yang menggunakan sistem berkas

abdou@debian:~$ ps up "1234 5678 91011"

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
abdou     1234  0.0  0.0  12345  1234 ?        Ssl  00:00   0:00 code
abdou     5678  0.0  0.0  12345  1234 ?        Ssl  00:00   0:00 msedge
abdou     91011  0.0  0.0  12345  1234 ?        Ssl  00:00   0:00 chrome
```

## Organisasi Pohon Berkas

Sistem UNIX tidak pernah terorganisir dengan baik! Berbagai konvensi penamaan yang tidak kompatibel digunakan secara bersamaan, dan berbagai jenis berkas tersebar secara acak di seluruh ruang nama. **Itulah mengapa sulit untuk meningkatkan sistem operasi**.

Sistem berkas root mencakup setidaknya direktori root dan sekumpulan berkas dan subdirektori minimal. Berkas yang berisi kernel OS biasanya berada di bawah **/boot**, tetapi nama dan lokasinya yang tepat dapat bervariasi. Di BSD dan beberapa sistem UNIX lainnya, kernel bukan satu berkas melainkan sekumpulan komponen.

**/etc** berisi berkas sistem dan konfigurasi penting. **/sbin** dan **/bin** untuk utilitas penting, dan terkadang **/tmp** untuk berkas sementara. **/dev** secara tradisional merupakan bagian dari sistem berkas root, tetapi saat ini merupakan sistem berkas virtual yang dipasang secara terpisah.

Beberapa sistem menyimpan berkas pustaka bersama dan beberapa hal aneh lainnya, seperti preprosesor C, di direktori **/lib** atau **/lib64**. Yang lain telah memindahkan item-item ini ke **/usr/lib**, terkadang meninggalkan **/lib** sebagai tautan simbolis.

**/usr** dan **/var** juga sangat penting. **/usr** adalah tempat di mana sebagian besar program standar-tetapi-tidak-kritis-sistem disimpan, bersama dengan berbagai barang rampasan lainnya seperti manual daring dan sebagian besar pustaka. FreeBSD menyimpan cukup banyak konfigurasi lokal di bawah **/usr/local**. **/var** berisi direktori spool, berkas log, informasi akuntansi, dan berbagai item lain yang tumbuh atau berubah dengan cepat dan bervariasi pada setiap host. Baik **/usr** dan **/var** harus tersedia untuk memungkinkan sistem benar-benar naik ke mode multi-pengguna.

![pathnames](./data/pathnames.png)

## Jenis Berkas

Sebagian besar implementasi sistem berkas mendefinisikan tujuh jenis berkas:

1. Berkas biasa (regular files)
2. Direktori (directories)
3. Berkas perangkat karakter (character device files)
4. Berkas perangkat blok (block device files)
5. Soket domain lokal (local domain sockets)
6. Pipa bernama (named pipes - FIFOs)
7. Tautan simbolis (symbolic links)

Anda dapat menentukan jenis berkas dengan menggunakan perintah **file** (ketik **man file** untuk informasi lebih lanjut).

```bash
$ file /bin/bash
bin/bash: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=33a5554034feb2af38e8c75872058883b2988bc5, for GNU/Linux 3.2.0, stripped
```

Anda juga dapat menggunakan **ls -ld**, flag **-d** memaksa **ls** untuk menampilkan informasi tentang direktori daripada menampilkan isi direktori.

![pengkodean-jenis-berkas](./data/file-type-encoding.png)

**Berkas biasa** terdiri dari serangkaian byte; sistem berkas tidak memaksakan struktur pada isinya. Berkas teks, berkas data, program yang dapat dieksekusi, dan pustaka bersama semuanya disimpan sebagai berkas biasa.

**Direktori** adalah referensi bernama ke berkas lain.

**Tautan keras (hard links)** adalah cara untuk memberikan beberapa nama pada satu berkas. Perintah **ln** membuat tautan keras baru ke berkas yang ada. Opsi **-i** pada **ls** menyebabkannya menampilkan jumlah tautan keras ke setiap berkas.

Contoh

```bash
$ ln /etc/passwd /tmp/passwd
```

**Berkas perangkat karakter dan blok**

Berkas perangkat memungkinkan program berkomunikasi dengan perangkat keras dan periferal sistem. Kernel menyertakan (atau memuat) perangkat lunak driver untuk setiap perangkat sistem. Perangkat lunak ini mengurus detail rumit pengelolaan setiap perangkat sehingga kernel itu sendiri dapat tetap relatif abstrak dan independen dari perangkat keras.

Perbedaan antara perangkat karakter dan blok sangat halus dan tidak layak ditinjau secara detail.

Berkas perangkat dicirikan oleh nomor perangkat utama dan minornya. Nomor utama mengidentifikasi driver yang mengontrol perangkat, dan nomor minor biasanya memberi tahu driver unit fisik mana yang akan dialamatkan. Misalnya, nomor perangkat utama 4 pada sistem Linux menunjukkan driver serial. Port serial pertama (**/dev/tty0**) akan memiliki nomor perangkat utama 4 dan nomor perangkat minor 0, port serial kedua (**/dev/tty1**) akan memiliki nomor perangkat utama 4 dan nomor perangkat minor 1, dan seterusnya.

Di masa lalu, **/dev** adalah direktori generik dan perangkat dibuat dengan **mknod** dan dihapus dengan **rm**. Sayangnya, sistem yang kasar ini kurang mampu menangani lautan driver dan jenis perangkat yang tak ada habisnya yang telah muncul selama beberapa dekade terakhir. Ini juga memfasilitasi berbagai macam potensi ketidakcocokan konfigurasi: berkas perangkat yang tidak merujuk ke perangkat yang sebenarnya, perangkat yang tidak dapat diakses karena tidak memiliki berkas perangkat, dan sebagainya.

Saat ini, direktori **/dev** biasanya dipasang sebagai jenis sistem berkas khusus, dan isinya secara otomatis dikelola oleh kernel bersama dengan daemon tingkat pengguna.

**Soket domain lokal** adalah cara bagi proses untuk berkomunikasi satu sama lain. Mereka mirip dengan soket jaringan, tetapi terbatas pada host lokal.

Syslog dan X Window System adalah contoh fasilitas standar yang menggunakan soket domain lokal.

**Pipa bernama** seperti soket domain lokal memungkinkan proses yang berjalan untuk berkomunikasi satu sama lain dalam host yang sama.

**Tautan simbolis** atau tautan lunak menunjuk ke berkas berdasarkan nama. Mereka adalah cara untuk memberikan beberapa nama pada satu berkas, tetapi lebih fleksibel daripada tautan keras. Mereka dapat menunjuk ke berkas di sistem berkas yang berbeda, dan mereka dapat menunjuk ke direktori.

Misalnya, direktori **/usr/bin** seringkali merupakan tautan simbolis ke **/bin**. Ini adalah cara untuk menjaga sistem berkas root tetap kecil dan memudahkan berbagi perangkat lunak yang sama di antara beberapa host.

```bash
$ ln -s /bin /usr/bin

$ ls -l /usr/bin
lrwxrwxrwx 1 root root 4 Mar  1  2020 /usr/bin -> /bin
```

## Atribut Berkas

Dalam model sistem berkas Unix dan Linux, setiap berkas memiliki seperangkat sembilan bit izin, yang menentukan siapa yang dapat membaca, menulis, dan mengeksekusi berkas. Bersama dengan tiga bit lainnya yang terutama memengaruhi operasi program yang dapat dieksekusi, bit-bit ini membentuk mode berkas.

Dua belas bit mode ini disimpan bersama dengan empat bit informasi tipe berkas. Empat bit tipe berkas diatur ketika berkas dibuat dan tidak dapat diubah, tetapi pemilik berkas dan superuser dapat memodifikasi dua belas bit mode dengan perintah **chmod**.

![atribut-berkas](https://cdn.storyasset.link/nlFtWFR5rySdmletT0jhDUQ0tXl2/ms-yxhcfoletf.jpg)

### Bit izin

Bit izin dibagi menjadi tiga kelompok dengan tiga bit masing-masing. Kelompok pertama dari tiga bit adalah untuk pemilik berkas, kelompok kedua adalah untuk grup berkas, dan kelompok ketiga adalah untuk semua orang lainnya.

Anda dapat menggunakan nama **Hugo** untuk mengingat urutan kelompok: **u** untuk pemilik (user), **g** untuk grup, dan **o** untuk yang lain (others).

Dimungkinkan juga untuk menggunakan notasi oktal (basis 8) karena setiap digit dalam notasi oktal mewakili tiga bit. Tiga bit teratas (dengan nilai oktal 400, 200, dan 100) mewakili pemilik berkas, tiga bit tengah (dengan nilai oktal 40, 20, dan 10) mewakili grup berkas, dan tiga bit terbawah (dengan nilai oktal 4, 2, dan 1) mewakili semua orang lainnya.

Pada berkas biasa, bit baca memungkinkan berkas untuk dibaca, bit tulis memungkinkan berkas untuk dimodifikasi atau dipotong; namun, kemampuan untuk menghapus atau mengganti nama (atau menghapus dan kemudian membuat ulang!) berkas dikendalikan oleh izin pada direktori induknya, di mana pemetaan nama-ke-ruang-data dipertahankan.

Bit eksekusi memungkinkan berkas untuk dieksekusi. Ada dua jenis berkas yang dapat dieksekusi: binari, yang dijalankan langsung oleh CPU, dan skrip, yang harus ditafsirkan oleh program seperti shell atau Python. Secara konvensi, skrip dimulai dengan baris **shebang** yang memberi tahu kernel interpreter mana yang akan digunakan.

```bash
#!/usr/bin/perl
```

Berkas yang dapat dieksekusi non-binari yang tidak menentukan interpreter diasumsikan sebagai skrip **sh**.

Kernel memahami sintaks *#!* (shebang) dan bertindak langsung. Namun, jika interpreter tidak ditentukan secara lengkap dan benar, kernel akan menolak berkas tersebut. Shell kemudian mengambil alih dan mencoba menafsirkan berkas sebagai skrip shell.

Untuk direktori, bit eksekusi (sering disebut bit pencarian atau pemindaian) memungkinkan direktori untuk dimasuki atau dilewati saat pathname dievaluasi, tetapi tidak untuk daftar isinya. Kombinasi bit baca dan eksekusi memungkinkan direktori untuk dibaca dan isinya dicantumkan. Kombinasi bit tulis dan eksekusi memungkinkan berkas dibuat, dihapus, dan diganti namanya dalam direktori.

### Bit setuid dan setgid

Bit dengan nilai oktal 4000 dan 2000 adalah bit setuid dan setgid. Ketika bit setuid diatur pada berkas, pemilik berkas sementara diubah menjadi pemilik berkas ketika berkas dieksekusi. Ketika bit setgid diatur pada berkas, grup berkas sementara diubah menjadi grup berkas ketika berkas dieksekusi.

Ketika diatur pada direktori, bit setgid menyebabkan berkas yang baru dibuat dalam direktori mengambil kepemilikan grup dari direktori daripada grup default dari pengguna yang membuat berkas. Ini memudahkan berbagi berkas di antara sekelompok pengguna.

### Bit sticky

Bit dengan nilai oktal 1000 adalah bit sticky. Ketika diatur pada direktori, bit sticky mencegah pengguna menghapus atau mengganti nama berkas yang bukan milik mereka. Ini berguna untuk direktori seperti **/tmp** yang dibagikan di antara banyak pengguna.

### ls: daftar dan periksa berkas

Perintah **ls** mencantumkan berkas dan direktori. Ini juga dapat digunakan untuk memeriksa atribut berkas dan direktori.

Opsi **-l** menyebabkan **ls** menampilkan format panjang, yang mencakup mode berkas, jumlah tautan keras ke berkas, pemilik berkas, grup berkas, ukuran berkas dalam byte, waktu modifikasi berkas, dan nama berkas.

Semua direktori memiliki setidaknya dua tautan keras: satu dari direktori itu sendiri (entri **..**) dan satu dari direktori induknya (entri **.**).

Output **ls** sedikit berbeda untuk berkas perangkat. Misalnya:

```bash
$ ls -l /dev/tty0
crw--w---- 1 root tty 4, 0 Mar  1  2020 /dev/tty0
```

**c** di awal baris menunjukkan bahwa berkas tersebut adalah berkas perangkat karakter. **4, 0** di akhir baris adalah nomor perangkat utama dan minor.

### chmod: ubah izin

Perintah **chmod** mengubah mode berkas. Anda dapat menggunakan notasi oktal atau notasi simbolis.

![pengkodean-izin](./data/permissions-encoding.png)

Contoh sintaks mnemonik chmod:

| Spesifikasi | Arti                                                                        |
| ----------- | --------------------------------------------------------------------------- |
| u+w         | Tambahkan izin tulis untuk pemilik berkas                                   |
| ug=rw,o=r   | Memberikan izin r/w kepada pemilik dan grup, dan izin r kepada orang lain   |
| a-x         | Hapus izin eksekusi untuk semua pengguna                                    |
| ug=srx, o=  | Atur bit setuid, setgid, dan sticky untuk pemilik dan grup (r/x)           |
| g=u         | Jadikan izin grup sama dengan izin pemilik                                  |

**Tips**: Anda juga dapat menentukan mode yang akan ditetapkan dengan menyalin mode dari berkas lain dengan opsi **--reference** (misalnya **chmod --reference=berkas_sumber berkas_target**).

### chown: ubah kepemilikan

Perintah **chown** mengubah pemilik dan grup berkas. Opsi **-R** menyebabkan **chown** mengubah kepemilikan isi berkas secara rekursif.

```bash
$ chown -R abdou:users /home/abdou
```

### chgrp: ubah grup

Perintah **chgrp** mengubah grup berkas. Opsi **-R** menyebabkan **chgrp** mengubah grup isi berkas secara rekursif.

```bash
$ chgrp -R users /home/abdou
```

### umask: atur izin default

Perintah **umask** mengatur izin default untuk berkas dan direktori baru. Perintah **umask** adalah bit mask yang dikurangkan dari izin default untuk menentukan izin sebenarnya.

Contoh:

```bash
$ umask 022
```

| Oktal | Biner | Izin | Oktal | Biner | Izin |
| ----- | ----- | ---- | ----- | ----- | ---- |
| 0     | 000   | rwx  | 4     | 100   | -wx  |
| 1     | 001   | rw-  | 5     | 101   | -w-  |
| 2     | 010   | r-x  | 6     | 110   | --x  |
| 3     | 011   | r--  | 7     | 111   | ---  |

Misalnya, **umask 027** memungkinkan rwx untuk pemilik, rx untuk grup, dan tidak ada izin untuk orang lain.

## Daftar Kendali Akses (Access Control Lists)

Model izin Unix tradisional sederhana dan efektif, tetapi memiliki keterbatasan. Misalnya, sulit untuk memberikan berkas beberapa pemilik, dan sulit untuk memberikan sekelompok pengguna izin yang berbeda pada berkas yang berbeda.

Daftar Kendali Akses (ACL) adalah cara untuk memperluas model izin Unix tradisional. ACL memungkinkan Anda memberikan beberapa pemilik berkas dan memberikan sekelompok pengguna izin yang berbeda pada berkas yang berbeda.

Setiap aturan dalam ACL disebut **entri kendali akses** (ACE). ACE terdiri dari **penentu pengguna atau grup**, **topeng izin**, dan **jenis**. Penentu pengguna atau grup bisa berupa nama pengguna, nama grup, atau kata kunci khusus seperti **pemilik** atau **lainnya**. Topeng izin adalah sekumpulan izin, dan jenisnya adalah **izinkan** atau **tolak**.

Perintah **getfacl** menampilkan ACL berkas, dan perintah **setfacl** mengatur ACL berkas.

```bash
$ getfacl /etc/passwd
```

```bash
$ setfacl -m u:abdou:rw /etc/passwd
```

Ada dua jenis ACL: **ACL POSIX** dan **ACL NFSv4**. ACL POSIX adalah ACL Unix tradisional, dan ACL NFSv4 adalah jenis ACL yang lebih baru dan lebih kuat.

### Implementasi ACL

Secara teori, tanggung jawab untuk memelihara dan menegakkan ACL dapat diberikan kepada beberapa komponen sistem yang berbeda. ACL dapat diimplementasikan oleh kernel atas nama semua sistem berkas sistem, oleh sistem berkas individual, atau mungkin oleh perangkat lunak tingkat yang lebih tinggi seperti server NFS dan SMB.

### ACL POSIX

ACL POSIX adalah ACL Unix tradisional. Mereka didukung oleh sebagian besar sistem operasi mirip Unix, termasuk Linux, FreeBSD, dan Solaris.

**Entri yang dapat muncul dalam ACL POSIX**

| Format                | Contoh          | Mengatur izin untuk         |
| --------------------- | --------------- | ---------------------------- |
| user::perms           | user:rw-        | Pemilik berkas               |
| user:username:perms   | user:abdou:rw-  | Pengguna bernama username    |
| group::perms          | group:r-x       | Grup berkas                  |
| group:groupname:perms | group:users:r-x | Grup bernama groupname       |
| mask::perms           | mask::rwx       | Izin maksimum                |
| other::perms          | other::r--      | Semua orang lainnya          |

Contoh:

```bash
$ setfacl -m user:abdou:rwx,group:users:rwx,other::r /home/abdou

$ getfacl --omit-header /home/abdou

user::rwx
user:abdou:rwx
group::r-x
group:users:r-x
mask::rwx
other::r--
```

### ACL NFSv4

ACL NFSv4 adalah jenis ACL yang lebih baru dan lebih kuat. Mereka didukung oleh beberapa sistem operasi mirip Unix, termasuk Linux dan FreeBSD.

ACL NFSv4 mirip dengan ACL POSIX, tetapi mereka memiliki beberapa fitur tambahan. Misalnya, ACL NFSv4 memiliki **ACL default** yang digunakan untuk mengatur ACL berkas dan direktori baru.

**Izin berkas NFSv4**

![nsfv4](./data/nfs4.png)