# **Tugas Instalasi dan Konfigurasi Layanan pada Debian 12**

## **A. Instalasi dan Konfigurasi NTP Client**
### **1. Instalasi dan Pengujian NTP Client**
Agar host memiliki waktu yang sinkron dengan server NTP di Indonesia, lakukan langkah-langkah berikut:

#### **Instalasi (Dilakukan di VM Debian 12)**
1. **Perbarui repositori paket**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Instal paket NTP yang diperlukan**
   ```bash
   sudo apt install ntp ntpsec -y
   ```

*(Tempat untuk screenshot instalasi paket NTP)*

#### **Konfigurasi dan Pengujian NTP Client**
1. **Edit file konfigurasi NTP**
   ```bash
   sudo nano /etc/ntp.conf
   ```
2. **Tambahkan atau sesuaikan baris berikut untuk menggunakan server NTP Indonesia:**
   ```plaintext
    server 0.id.pool.ntp.org
    server 1.id.pool.ntp.org
    server 3.id.pool.ntp.org
    server 2.id.pool.ntp.org
   ```
3. **Simpan perubahan dan keluar dari editor.**
4. **Restart layanan NTP untuk menerapkan konfigurasi**
   ```bash
   sudo systemctl restart ntp
   ```
5. **Pastikan layanan NTP berjalan**
   ```bash
   sudo systemctl status ntp
   ```

*(Tempat untuk screenshot status layanan NTP)*

6. **Verifikasi sinkronisasi waktu**
   ```bash
   ntpq -p
   timedatectl
   date
   ```

*(Tempat untuk screenshot hasil verifikasi waktu)*

### **Referensi**
- [Server World - NTP pada Debian 12](https://www.server-world.info/en/note?os=Debian_12&p=ntp&f=1)
- [NTP Pool - Indonesia](https://www.ntppool.org/en/zone/id)

---

## **B. Instalasi dan Konfigurasi Samba**

### **1. Instalasi dan Pengujian Samba (Dilakukan di VM Debian 12)**
1. **Perbarui repositori dan instal paket Samba**
   ```bash
   sudo apt update && sudo apt install samba smbclient cifs-utils -y
   ```

*(Tempat untuk screenshot instalasi paket Samba)*

2. **Pastikan layanan berjalan**
   ```bash
   sudo systemctl status smbd
   ```

*(Tempat untuk screenshot status layanan Samba)*

### **2. Konfigurasi dan Pengujian Public Shared Folder (Dilakukan di VM Debian 12)**
1. **Buat direktori untuk shared folder**
   ```bash
   sudo mkdir -p /srv/samba/public
   ```
2. **Atur izin akses**
   ```bash
   sudo chmod 777 /srv/samba/public
   ```
3. **Edit konfigurasi Samba**
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Tambahkan konfigurasi berikut:
   ```plaintext
   [Public]
   path = /srv/samba/public
   browsable = yes
   writable = yes
   guest ok = yes
   read only = no
   ```
4. **Restart layanan Samba**
   ```bash
   sudo systemctl restart smbd
   ```

*(Tempat untuk screenshot restart layanan Samba)*

5. **Uji akses dari Fedora Host melalui Dolphin:**
   - Buka Dolphin File Manager.
   - Klik **Network** > **Samba Shares**.
   - Masukkan alamat: `smb://server-ip/Public`.
   - Pastikan bisa mengakses folder tanpa autentikasi.

*(Tempat untuk screenshot akses Dolphin ke Public Shared Folder)*

### **3. Konfigurasi dan Pengujian Limited Shared Folder (Dilakukan di VM Debian 12)**
1. **Buat direktori untuk limited shared folder**
   ```bash
   sudo mkdir -p /srv/samba/limited
   ```
2. **Buat grup khusus untuk akses terbatas**
   ```bash
   sudo groupadd sambashare
   ```
3. **Tambahkan pengguna ke grup**
   ```bash
   sudo usermod -aG sambashare $USER
   ```
4. **Atur izin dan kepemilikan direktori**
   ```bash
   sudo chown -R root:sambashare /srv/samba/limited
   sudo chmod 770 /srv/samba/limited
   ```
5. **Edit konfigurasi Samba**
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Tambahkan:
   ```plaintext
   [Limited]
   path = /srv/samba/limited
   browsable = yes
   writable = yes
   valid users = @sambashare
   read only = no
   ```
6. **Setel password Samba untuk pengguna**
   ```bash
   sudo smbpasswd -a $USER
   ```
7. **Restart layanan Samba**
   ```bash
   sudo systemctl restart smbd
   ```

*(Tempat untuk screenshot restart layanan Samba setelah konfigurasi Limited Shared Folder)*

8. **Uji akses dari Fedora Host melalui CLI:**
   ```bash
   smbclient -L //server-ip -U username
   ```

*(Tempat untuk screenshot hasil akses CLI ke Limited Shared Folder)*

9. **Uji akses dari Fedora Host melalui Dolphin:**
   - Buka Dolphin File Manager.
   - Klik **Network** > **Samba Shares**.
   - Masukkan alamat: `smb://server-ip/Limited`.
   - Masukkan username dan password.
   - Pastikan bisa mengakses folder dengan autentikasi.

*(Tempat untuk screenshot akses Dolphin ke Limited Shared Folder)*

### **Referensi**
- [Server World - Samba pada Debian 12](https://www.server-world.info/en/note?os=Debian_12&p=samba&f=1)

---

## **C. Rangkuman tentang Package Management di Debian 12**

### **1. Pengertian Package Management**
Package management adalah sistem untuk menginstal, menghapus, dan memperbarui perangkat lunak di Debian menggunakan alat seperti `apt`, `dpkg`, dan `snap`.

### **2. Alat-alat Package Management**
1. **APT (Advanced Package Tool)**
2. **DPKG (Debian Package Manager)**
3. **Snap**

### **Referensi**
- Materi **Ethol Debian 12 SysAdmin**

---

Dokumen ini berisi langkah-langkah terstruktur untuk instalasi dan konfigurasi NTP Client, Samba, serta rangkuman tentang package management di Debian 12. Tersedia ruang untuk dokumentasi berupa screenshot.

