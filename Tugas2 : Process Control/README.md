# Process Control

## Pendahuluan

Process control merupakan konsep penting dalam UNIX dan Linux yang menjelaskan bagaimana sistem operasi mengelola dan mengendalikan proses. Proses adalah program yang sedang dijalankan dan merupakan salah satu konsep dasar dalam sistem operasi. Bab 4 dari buku UNIX and Linux System Administration Handbook (Edisi 5) membahas secara mendalam tentang proses, bagaimana proses dibuat, bagaimana proses dihentikan, dan bagaimana sistem operasi menjadwalkan proses.

## Dasar-dasar Process Control

### Anatomi Proses

Proses di Unix/Linux memiliki beberapa karakteristik penting:

- **PID (Process ID)**: Pengenal unik untuk setiap proses
- **PPID (Parent Process ID)**: ID dari proses induk yang membuat proses tersebut
- **UID dan GID**: User ID dan Group ID yang menjalankan proses
- **Prioritas dan jadwal eksekusi**: Menentukan kapan dan seberapa sering proses mendapat jatah CPU
- **Working directory**: Direktori kerja dari proses
- **Environment variables**: Variabel lingkungan yang tersedia untuk proses
- **Handlers**: Rutinitas untuk menangani berbagai sinyal
- **File descriptor**: Pointer ke file, socket, dan resources lain

### Siklus Hidup Proses

1. **Pembuatan Proses**:
   - Proses dibuat melalui system call `fork()` yang membuat duplikat (child) dari proses yang memanggilnya (parent)
   - Child process biasanya melanjutkan dengan system call `exec()` untuk mengganti program yang dieksekusi

2. **Status Proses**:
   - **Running**: Proses aktif berjalan
   - **Sleeping**: Proses menunggu input/output atau sumber daya lain
   - **Stopped**: Proses dihentikan sementara (misalnya dengan SIGSTOP)
   - **Zombie**: Proses yang telah selesai tapi masih memiliki entry di tabel proses

3. **Terminasi Proses**:
   - Proses dapat berakhir normal dengan system call `exit()`
   - Proses dapat dihentikan paksa dengan sinyal seperti SIGKILL

## Process Control di Praktik

### Perintah-perintah Dasar

#### Melihat Proses

```bash
# Melihat semua proses dengan format lengkap
ps -ef
ps aux

# Melihat proses secara real-time
top
htop

# Melihat proses dengan format tree
pstree

# Mencari proses berdasarkan nama
pgrep [nama_program]
```

#### Mengelola Proses

```bash
# Menghentikan proses dengan sinyal default (SIGTERM)
kill [PID]

# Menghentikan proses secara paksa
kill -9 [PID]
pkill [nama_program]

# Mengirim sinyal tertentu ke proses
kill -s [sinyal] [PID]
```

#### Job Control

```bash
# Menjalankan program di background
command &

# Menghentikan proses sementara (SIGSTOP)
Ctrl+Z

# Melanjutkan proses di background
bg

# Membawa proses background ke foreground
fg

# Melihat job yang sedang berjalan
jobs
```

### Prioritas Proses

```bash
# Menjalankan program dengan prioritas tertentu (nilai nice dari -20 hingga 19)
nice -n [nilai] [command]

# Mengubah prioritas proses yang sudah berjalan
renice [nilai] -p [PID]
```

### Sinyal (Signals)

Sinyal adalah mekanisme komunikasi asinkron antarproses di UNIX/Linux. Beberapa sinyal penting:

- **SIGHUP (1)**: Hangup, biasanya untuk mereload konfigurasi
- **SIGINT (2)**: Interrupt (Ctrl+C)
- **SIGKILL (9)**: Menghentikan proses secara paksa (tidak dapat diblock)
- **SIGTERM (15)**: Terminasi normal (dapat dihandle)
- **SIGSTOP (19)**: Menghentikan proses sementara (tidak dapat diblock)
- **SIGCONT (18)**: Melanjutkan proses yang berhenti

## Daemon dan Service

Daemon adalah proses yang berjalan di background secara terus-menerus dan tidak terkait dengan terminal manapun. Contoh daemon:

- Web server (httpd, nginx)
- Database server (mysqld, postgresql)
- SSH server (sshd)

Manajemen service di sistem modern menggunakan systemd:

```bash
# Memulai service
systemctl start [nama_service]

# Menghentikan service
systemctl stop [nama_service]

# Merestart service
systemctl restart [nama_service]

# Melihat status service
systemctl status [nama_service]

# Mengaktifkan service saat boot
systemctl enable [nama_service]
```

## Latihan Praktik

### Latihan 1: Mengelola Proses

1. Jalankan beberapa proses seperti `sleep 1000 &`, `top`, dan `tail -f /var/log/syslog`
2. Gunakan perintah `ps aux` untuk melihat proses yang berjalan
3. Identifikasi PID dari proses tersebut
4. Hentikan proses dengan SIGTERM
5. Jika proses tidak berhenti, gunakan SIGKILL

### Latihan 2: Prioritas Proses

1. Jalankan proses CPU-intensive dengan nilai nice yang berbeda:
   ```bash
   nice -n 19 sha1sum /dev/zero &
   nice -n 0 sha1sum /dev/zero &
   nice -n -10 sha1sum /dev/zero &  # Memerlukan hak root
   ```
2. Amati perbedaan penggunaan CPU dengan `top`
3. Ubah prioritas proses dengan `renice`

### Latihan 3: Sinyal dan Handling

Buat script sederhana yang menangani sinyal:

```bash
#!/bin/bash
trap 'echo "Caught SIGINT"; exit 1' INT
trap 'echo "Caught SIGTERM"; exit 0' TERM
trap 'echo "Caught SIGHUP"; exit 0' HUP

echo "PID: $$"
while true; do
    sleep 1
done
```

Jalankan script dan kirim berbagai sinyal untuk melihat efeknya.

## Kesimpulan

Process control adalah komponen fundamental dalam administrasi sistem UNIX dan Linux. Memahami cara kerja proses dan bagaimana mengelolanya dengan efektif sangat penting untuk:

- Troubleshooting masalah performa
- Manajemen sumber daya sistem
- Otomatisasi tugas administrasi
- Pemeliharaan layanan yang handal

## Referensi

- UNIX and Linux System Administration Handbook (Edisi 5), Bab 4
- [Linux Process Management](https://www.kernel.org/doc/html/latest/admin-guide/pm/index.html)
- [systemd Documentation](https://systemd.io/)
- Manual pages (`man ps`, `man kill`, `man signal`, dll.)