<div align="center">

# LAPORAN PRAKTIKUM


*dns*  

**Workshop Administrasi Jaringan**  

![logo pens](media/image1.jpg)  

**Nama Dosen Pengampu**:  
Bapak Dr. Ferry Astika Saputra ST, M.Sc  
<br>
**Dikerjakan oleh**:  
Nama: Adrian Mendienta Tumanggor  
Kelas: 2 D4 IT A  
NRP: 3123600012

**DEPARTEMEN TEKNIK INFORMATIKA DAN KOMPUTER**  
**POLITEKNIK ELEKTRONIKA NEGERI SURABAYA**  
**2025**  
</div>

# Workshop DNS (Domain Name System)

## Gambaran Umum
Dokumen ini mencakup workshop praktis tentang administrasi DNS (Domain Name System), termasuk konsep dasar, implementasi, dan konfigurasi menggunakan BIND9 pada sistem Debian.

## Daftar Isi
- [Latar Belakang Teoritis](#latar-belakang-teoritis)
  - [Apa itu DNS?](#apa-itu-dns)
  - [Fitur Utama](#fitur-utama)
  - [Protokol dan Port](#protokol-dan-port)
  - [Proses Resolusi DNS](#proses-resolusi-dns)
  - [Komponen DNS](#komponen-dns)
- [Implementasi Praktis](#implementasi-praktis)
  - [Instalasi BIND9](#instalasi-bind9)
  - [Konfigurasi BIND9](#konfigurasi-bind9)
  - [Konfigurasi Jaringan](#konfigurasi-jaringan)
  - [Pengujian Konfigurasi](#pengujian-konfigurasi)
  - [Pengamanan DNS dengan Aturan Firewall](#pengamanan-dns-dengan-aturan-firewall)

## Latar Belakang Teoritis

### Apa itu DNS?
Domain Name System (DNS) adalah sistem penamaan terdesentralisasi yang menerjemahkan nama domain yang mudah dibaca manusia (seperti www.example.com) menjadi alamat IP (seperti 192.0.2.1) yang digunakan komputer untuk saling berkomunikasi dalam jaringan.

### Fitur Utama

#### Server DNS Multiple dengan Manajemen Terdistribusi
DNS menggunakan server yang tersebar secara global dan dikelola oleh berbagai operator untuk mengurangi latensi dan meningkatkan redundansi sistem secara keseluruhan.

#### Sistem yang Loosely Coherent
Meskipun bersifat terdistribusi, DNS tetap terintegrasi dalam satu sistem global, memastikan konsistensi data secara bertahap (*eventual consistency*) di seluruh node.

#### Arsitektur yang Scalable
Sistem DNS dirancang untuk menangani pertumbuhan dengan mudah melalui penskalaan vertikal dan horizontal, mengakomodasi ekspansi internet yang berkelanjutan.

#### Keandalan Tinggi
Sebagai komponen infrastruktur internet yang kritis, DNS dilengkapi dengan mekanisme failover, redundansi, dan perlindungan DDoS untuk memastikan ketersediaan tinggi (*high availability*).

#### Pembaruan Dinamis
DNS mendukung pembaruan record secara real-time (misalnya melalui DDNS) tanpa menyebabkan downtime, memungkinkan penambahan domain atau perubahan konfigurasi secara fleksibel.

### Protokol dan Port

#### UDP Port 53
Mayoritas kueri dan respons DNS menggunakan UDP karena:
- Overhead yang rendah
- Cocok untuk pertukaran data kecil yang membutuhkan kecepatan
- Efisien dalam menangani paket DNS yang biasanya berukuran kecil

#### TCP Port 53
Digunakan dalam kasus khusus:
- Transfer data besar (misalnya: *zone transfers* antar server DNS untuk sinkronisasi data zona)
- Ketika respons DNS melebihi ukuran maksimum paket UDP (512 byte)

### Proses Resolusi DNS

1. **Permintaan Pengguna**
   - Pengguna memasukkan domain (misalnya, `www.example.com.au`) di browser
   - Browser pertama kali memeriksa **cache lokal** untuk alamat IP yang sebelumnya pernah diakses

2. **Menghubungi DNS Resolver**
   - Jika tidak ada di cache, browser menghubungi **DNS Resolver** (biasanya disediakan oleh ISP atau layanan seperti Google DNS)
   - Resolver bertanggung jawab untuk mencari alamat IP dari domain yang diminta

3. **Pencarian Hierarkis**
   - Resolver mulai dengan menanyakan **Root Server** untuk menentukan server mana yang mengelola ekstensi domain (misalnya, `.com`)
   - Root Server mengarahkan ke **TLD Server** (Top-Level Domain Server) yang sesuai

   **Jenis-jenis TLD:**
   - **Country-code TLDs (ccTLDs)**  
     Kode domain dua huruf yang mewakili negara atau wilayah tertentu  
     Contoh: `.id` (Indonesia), `.us` (Amerika Serikat), `.jp` (Jepang)  
     Digunakan untuk situs yang terkait dengan wilayah geografis tertentu
   
   - **Generic TLDs (gTLDs)**  
     Domain tingkat atas umum yang tidak terikat pada lokasi geografis, dikategorikan berdasarkan tujuan atau industri  
     Contoh: `.com` (komersial), `.org` (organisasi), `.gov` (pemerintah AS), `.app` (aplikasi)  
     Mengelompokkan situs berdasarkan jenis entitas, layanan, atau tema tertentu
   
   - **Infrastructure TLD**  
     TLD khusus yang digunakan untuk keperluan teknis infrastruktur internet  
     Contoh: `.arpa` (digunakan untuk manajemen alamat dan routing jaringan)  
     Mendukung operasi teknis jaringan seperti reverse DNS lookup
   
   - **Internationalized TLDs (IDN)**  
     TLD yang menggunakan karakter non-ASCII (aksara lokal atau simbol khusus)  
     Contoh: `.中国` (Tiongkok), `.рф` (Rusia), `.مصر` (Mesir)  
     Memungkinkan penggunaan nama domain dalam bahasa/aksara lokal untuk meningkatkan aksesibilitas pengguna global

4. **Pencarian Server Otoritatif**
   - TLD Server memberikan informasi tentang **Authoritative Server** (server yang menyimpan data lengkap untuk domain `example.com.au`)
   - Authoritative Server merespons dengan alamat IP tujuan (misalnya, `m.n.o.p`)

5. **Pengiriman Hasil ke Pengguna**
   - Resolver menerima alamat IP dan menyimpannya di cache untuk mempercepat akses berikutnya
   - Browser akhirnya mengakses alamat IP tersebut (`m.n.o.p`) untuk menampilkan situs web

### Komponen DNS

#### 1. Namespace
- **Definisi:**  
  Namespace DNS adalah struktur hierarkis global yang mengatur penamaan domain, menyerupai "pohon terbalik" dengan:
  - **Root** (.) di puncak
  - **Top-Level Domains (TLD)** (misalnya, `.com`, `.id`)
  - **Second-Level Domains** (misalnya, `example.com`)
  - **Subdomains** (misalnya, `blog.example.com`)

- **Domain dan Zona:**
  - **Domain:** Nama unik yang diidentifikasi dalam hierarki (misalnya, `example.com`)
  - **Zona:** Bagian dari namespace yang dikelola oleh otoritas tertentu (misalnya, zona `example.com` mencakup semua subdomain dan record di bawahnya)

#### 2. Nameserver
- **Definisi:**  
  Server yang menyimpan dan mengelola data DNS untuk zona tertentu

- **Fungsi:**
  - Menyimpan catatan DNS (A, MX, CNAME, dll.) untuk zona yang diotorisasi
  - Merespons permintaan DNS resolver terkait record dalam zona tersebut

- **Jenis:**
  - **Authoritative Nameserver:** Berisi data resmi untuk zona tertentu
  - **Recursive Nameserver:** Mencari data dari nameserver lain ketika tidak memiliki jawabannya

#### 3. Resolver atau Klien
- **Definisi:**  
  Perangkat atau layanan yang mengirimkan kueri ke nameserver untuk menerjemahkan nama domain menjadi alamat IP

- **Contoh:**
  - **Layanan Resolver:** Google DNS (8.8.8.8) atau Cloudflare (1.1.1.1)
  - **Klien:** Browser, aplikasi, atau sistem operasi yang memulai permintaan DNS

- **Proses Kerja:**
  1. Klien meminta resolver untuk menerjemahkan domain
  2. Resolver mencari jawaban dengan bertanya ke nameserver (root → TLD → authoritative)
  3. Hasil dikirim kembali ke klien

## Implementasi Praktis

### Instalasi BIND9

BIND (Berkeley Internet Name Domain) adalah perangkat lunak server DNS yang paling banyak digunakan. Instalasi pada Debian:

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc
```

### Konfigurasi BIND9

Konfigurasi untuk domain "kelompok07.com" memerlukan pengeditan beberapa file:

#### 1. `/etc/bind/named.conf`
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
include "/etc/bind/named.conf.internal-zones";  # Ditambahkan untuk zona jaringan internal
```
**Tujuan:** Menyertakan file konfigurasi zona internal

#### 2. `/etc/bind/named.conf.options`
```
acl internal-network {
    192.168.200.0/24;
};

options {
    directory "/var/cache/bind";
    allow-query { localhost; internal-network; };
    allow-transfer { localhost; };
    recursion yes;
};
```
**Tujuan:** Mendefinisikan pengaturan global server DNS
- **acl internal-network:** Mendefinisikan 192.168.200.0/24 sebagai "jaringan internal"
- **directory:** Menentukan lokasi cache DNS
- **allow-query:** Mengizinkan komputer dari localhost dan jaringan internal untuk mengirim kueri DNS
- **allow-transfer:** Hanya mengizinkan transfer zona dari localhost (langkah keamanan)
- **recursion yes:** Mengizinkan server mencari jawaban di server DNS lain

#### 3. `/etc/bind/named.conf.internal-zones`
```
zone "com" {
    type master;
    file "/etc/bind/com.lan";
    allow-update { none; };
};

zone "200.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/200.168.192.db";
    allow-update { none; };
};
```
**Tujuan:** Mendefinisikan zona DNS untuk domain "com" dan DNS balik
- **zone "com":** Mendefinisikan zona forward untuk menerjemahkan nama domain "com" menjadi alamat IP
- **type master:** Server ini adalah sumber utama untuk zona ini
- **file:** Lokasi file detail zona
- **allow-update:** Tidak mengizinkan pembaruan dinamis (langkah keamanan)
- **zone "200.168.192.in-addr.arpa":** Zona reverse untuk menerjemahkan alamat IP menjadi nama domain

#### 4. `/etc/bind/com.lan`
```
$TTL 86400
@   IN  SOA ns.com. admin.com. (
        2023011301  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400       ; Minimum TTL
)
@   IN  NS  ns.com.
@   IN  A   192.168.200.2
ns  IN  A   192.168.200.2
kelompok07  IN  A   192.168.200.2
```
**Tujuan:** Mendefinisikan detail untuk zona "com", termasuk catatan DNS
- **$TTL:** Time To Live - berapa lama record boleh di-cache (86400 detik = 1 hari)
- **SOA:** Start of Authority - informasi otoritatif tentang zona:
  - **ns.com.:** Name server utama untuk zona
  - **admin.com.:** Email admin (@ diganti dengan .)
  - **Serial:** Nomor versi konfigurasi (biasanya format tanggal YYYYMMDDNN)
  - **Refresh:** Berapa sering server sekunder harus memeriksa pembaruan (3600 detik)
  - **Retry:** Berapa lama menunggu jika pembaruan gagal (1800 detik)
  - **Expire:** Berapa lama data dianggap valid jika server utama tidak tersedia (604800 detik)
  - **Minimum TTL:** Default TTL minimum (86400 detik)
- **NS:** Name Server record - mendefinisikan ns.com sebagai name server untuk zona
- **A:** Address record - memberikan alamat IP untuk nama domain:
  - **@ IN A:** IP untuk domain dasar (com)
  - **ns IN A:** IP untuk ns.com
  - **kelompok07 IN A:** IP untuk kelompok07.com (192.168.200.2)

#### 5. `/etc/bind/200.168.192.db`
```
$TTL 86400
@   IN  SOA ns.com. admin.com. (
        2023011301  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400       ; Minimum TTL
)
@   IN  NS  ns.com.
2   IN  PTR kelompok07.com.
```
**Tujuan:** Menerjemahkan alamat IP menjadi nama domain (pencarian balik)
- **2 IN PTR kelompok07.com.:** Memetakan IP 192.168.200.2 ke nama domain kelompok07.com

### Konfigurasi Jaringan

Konfigurasikan resolver untuk menggunakan server DNS Debian student (192.168.200.2):

Edit `/etc/resolv.conf`:
```
nameserver 192.168.200.2
```

### Pengujian Konfigurasi

Uji konfigurasi DNS dengan alat seperti `dig` atau `nslookup`:

```bash
dig kelompok07.com
nslookup kelompok07.com
```

### Pengamanan DNS dengan Aturan Firewall

Konfigurasikan iptables untuk mengamankan lalu lintas DNS:

```bash
# Izinkan koneksi masuk ke port DNS (53) melalui TCP
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Izinkan koneksi masuk ke port DNS (53) melalui UDP
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Konfigurasikan NAT/masquerading untuk lalu lintas keluar
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
```

**Tujuan:**
- Mengizinkan koneksi masuk ke port 53 (DNS) melalui TCP dan UDP
- Port 53 digunakan oleh DNS, biasanya menggunakan UDP, tetapi juga dapat menggunakan TCP untuk transfer data besar
- Konfigurasi NAT: Menyiapkan IP masquerading untuk lalu lintas keluar melalui antarmuka enp0s8
- Semua lalu lintas dari LAN (yang diteruskan) akan disamarkan menggunakan IP publik dari enp0s8

## Kesimpulan

Workshop ini mendemonstrasikan dasar-dasar teoritis DNS dan implementasi praktisnya menggunakan BIND9 pada sistem Debian. Memahami DNS sangat penting bagi administrator jaringan karena DNS membentuk tulang punggung konektivitas internet, menerjemahkan nama domain yang mudah dibaca manusia menjadi alamat IP yang dapat digunakan oleh mesin.

## Referensi
- Dokumentasi BIND9: https://bind9.readthedocs.io/
- Internet Systems Consortium: https://www.isc.org/bind/