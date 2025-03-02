# Process Control di UNIX/Linux: Panduan Praktikum

## Daftar Isi
- [Pendahuluan](#pendahuluan)
- [Persiapan Lab](#persiapan-lab)
- [Praktikum 1: Memonitor Proses](#praktikum-1-memonitor-proses)
- [Praktikum 2: Mengelola Proses](#praktikum-2-mengelola-proses)
- [Praktikum 3: Prioritas Proses](#praktikum-3-prioritas-proses)
- [Praktikum 4: Menggunakan Sinyal](#praktikum-4-menggunakan-sinyal)
- [Praktikum 5: Menjadwalkan Proses dengan cron](#praktikum-5-menjadwalkan-proses-dengan-cron)
- [Praktikum 6: Debugging Proses dengan strace](#praktikum-6-debugging-proses-dengan-strace)
- [Latihan Mandiri](#latihan-mandiri)

## Pendahuluan

Proses adalah komponen fundamental dalam sistem operasi Unix/Linux. Setiap proses terdiri dari:
- Address space (kode, data, stack)
- Struktur data kernel untuk melacak status
- Sumber daya yang dialokasikan

Dokumen ini menyediakan panduan praktis untuk memahami dan mengelola proses di lingkungan Unix/Linux.

## Persiapan Lab

1. Masuk ke direktori `/unix-and-linux-sysadmin-notes/`
2. Jalankan container lab:
   ```bash
   docker-compose up -d --build
   ```
3. Verifikasi container dan network sudah berhasil dibuat
4. Masuk ke container lab:
   ```bash
   docker exec -it lab-debian bash
   ```

## Praktikum 1: Memonitor Proses

### 1.1. Menggunakan ps

1. Jalankan perintah berikut untuk melihat semua proses yang sedang berjalan:
   ```bash
   ps aux
   ```

2. Amati kolom output dan catat informasi yang ditampilkan:
   - USER: pemilik proses
   - PID: ID proses
   - %CPU: persentase CPU yang digunakan
   - %MEM: persentase memori yang digunakan
   - VSZ: ukuran virtual memori
   - RSS: ukuran memori fisik yang digunakan
   - TTY: terminal yang terkait dengan proses
   - STAT: status proses
   - START: waktu proses dimulai
   - TIME: waktu CPU yang digunakan
   - COMMAND: perintah yang dijalankan

3. Gunakan perintah berikut untuk melihat struktur hierarki proses:
   ```bash
   ps f
   ```

4. Gunakan `ps lax` untuk melihat informasi teknis lebih lanjut tentang proses:
   ```bash
   ps lax
   ```

### 1.2. Menggunakan top dan htop

1. Jalankan `top` untuk melihat proses secara real-time:
   ```bash
   top
   ```
   
2. Pelajari kontrol interaktif di top:
   - `q`: keluar dari top
   - `h`: menampilkan bantuan
   - `k`: mengirim sinyal ke proses
   - `r`: mengubah prioritas proses
   - `F`: konfigurasikan kolom yang ditampilkan
   - `<`: sortir berdasarkan kolom sebelumnya
   - `>`: sortir berdasarkan kolom berikutnya

3. Jika tersedia, coba gunakan htop untuk tampilan yang lebih interaktif:
   ```bash
   htop
   ```
   (Jika tidak tersedia, instal dengan `apt-get install htop`)

### 1.3. Memeriksa /proc Filesystem

1. Lihat informasi tentang CPU dari /proc:
   ```bash
   cat /proc/cpuinfo
   ```

2. Periksa informasi memori:
   ```bash
   cat /proc/meminfo
   ```

3. Pilih sebuah proses (misalnya bash) dan periksa informasinya:
   ```bash
   # Temukan PID dari bash
   pidof bash
   
   # Ganti PID dengan nilai yang sebenarnya
   PID=$(pidof bash | awk '{print $1}')
   
   # Lihat command line proses
   cat /proc/$PID/cmdline
   
   # Lihat environment variables
   cat /proc/$PID/environ | tr '\0' '\n'
   
   # Lihat status proses
   cat /proc/$PID/status
   
   # Lihat file descriptor yang dibuka
   ls -l /proc/$PID/fd
   ```

## Praktikum 2: Mengelola Proses

### 2.1. Membuat Proses Background

1. Buat script sederhana yang berjalan lama:
   ```bash
   cat > infinite.sh << 'EOL'
   #!/bin/bash
   count=0
   while true; do
       echo "Loop $count"
       count=$((count+1))
       sleep 5
   done
   EOL
   
   chmod +x infinite.sh
   ```

2. Jalankan script di background:
   ```bash
   ./infinite.sh &
   ```

3. Lihat proses yang berjalan di background:
   ```bash
   jobs
   ```

4. Bawa proses ke foreground:
   ```bash
   fg %1
   ```

5. Tekan Ctrl+Z untuk menghentikan sementara, kemudian kirim kembali ke background:
   ```bash
   bg %1
   ```

### 2.2. Menggunakan kill, killall, dan pkill

1. Buat beberapa proses background:
   ```bash
   ./infinite.sh > log1.txt &
   ./infinite.sh > log2.txt &
   ./infinite.sh > log3.txt &
   ```

2. Identifikasi PID dari proses:
   ```bash
   ps aux | grep infinite.sh
   ```

3. Hentikan salah satu proses dengan kill:
   ```bash
   # Ganti PID dengan nilai sebenarnya dari output ps
   kill PID
   ```

4. Hentikan semua proses infinite.sh dengan killall:
   ```bash
   killall infinite.sh
   ```

5. Mulai ulang beberapa proses, kemudian gunakan pkill:
   ```bash
   ./infinite.sh > log1.txt &
   ./infinite.sh > log2.txt &
   
   # Hentikan semua proses dengan nama tertentu
   pkill -f infinite.sh
   ```

### 2.3. Menemukan PID Proses

1. Gunakan pgrep untuk menemukan PID berdasarkan nama:
   ```bash
   ./infinite.sh > logfile.txt &
   pgrep -f infinite.sh
   ```

2. Gunakan pidof untuk menemukan PID berdasarkan path:
   ```bash
   pidof bash
   ```

## Praktikum 3: Prioritas Proses

### 3.1. Memahami nice dan renice

1. Jalankan proses dengan prioritas nice rendah (nilai tinggi):
   ```bash
   nice -n 10 ./infinite.sh &
   ```

2. Jalankan proses dengan prioritas nice tinggi (nilai rendah):
   ```bash
   # Perlu hak root
   sudo nice -n -10 ./infinite.sh &
   ```

3. Periksa nilai nice dari proses:
   ```bash
   ps -o pid,ppid,ni,cmd -p $(pgrep -f infinite.sh)
   ```

4. Ubah prioritas proses yang sudah berjalan:
   ```bash
   # Temukan PID
   PID=$(pgrep -f infinite.sh | head -1)
   
   # Ubah nice value (sebagai root)
   sudo renice -n 15 -p $PID
   
   # Verifikasi perubahan
   ps -o pid,ppid,ni,cmd -p $PID
   ```

### 3.2. Melihat Dampak Prioritas

1. Buat script CPU-intensif:
   ```bash
   cat > cpu_intensive.sh << 'EOL'
   #!/bin/bash
   end=$((SECONDS+60))
   while [ $SECONDS -lt $end ]; do
       for i in {1..1000000}; do
           echo "Scale=5000; 4*a(1)" | bc -l > /dev/null
       done
   done
   echo "Done"
   EOL
   
   chmod +x cpu_intensive.sh
   ```

2. Jalankan dua instance dengan prioritas berbeda:
   ```bash
   time nice -n 19 ./cpu_intensive.sh &
   time nice -n 0 ./cpu_intensive.sh &
   ```

3. Monitor dengan top dan perhatikan penggunaan CPU.

## Praktikum 4: Menggunakan Sinyal

### 4.1. Mempelajari Sinyal

1. Lihat daftar semua sinyal:
   ```bash
   kill -l
   ```

2. Buat script yang menangani sinyal:
   ```bash
   cat > signal_handler.sh << 'EOL'
   #!/bin/bash
   
   echo "PID: $$"
   
   trap 'echo "Received SIGHUP (1)"' SIGHUP
   trap 'echo "Received SIGINT (2)"; exit' SIGINT
   trap 'echo "Received SIGTERM (15)"' SIGTERM
   trap 'echo "Received SIGUSR1 (10)"' SIGUSR1
   
   echo "Script running. Send signals with:"
   echo "kill -HUP $$ (Hang up)"
   echo "kill -INT $$ or Ctrl+C (Interrupt)"
   echo "kill -TERM $$ (Terminate)"
   echo "kill -USR1 $$ (User defined 1)"
   
   while true; do
       sleep 2
       echo "."
   done
   EOL
   
   chmod +x signal_handler.sh
   ```

3. Jalankan script:
   ```bash
   ./signal_handler.sh
   ```

4. Dari terminal lain, kirim sinyal ke proses:
   ```bash
   # Ganti PID dengan nilai yang ditampilkan oleh script
   kill -HUP PID
   kill -USR1 PID
   kill -TERM PID
   ```

5. Kembali ke terminal pertama dan tekan Ctrl+C untuk mengirim SIGINT.

### 4.2. Membuat Zombie Process

1. Buat script yang menghasilkan zombie process:
   ```bash
   cat > zombie.sh << 'EOL'
   #!/bin/bash
   
   # Parent process
   echo "Parent PID: $$"
   
   # Create child process
   for i in {1..3}; do
       bash -c "echo Child process $i with PID: \$\$; sleep 1; exit" &
   done
   
   echo "Parent sleeping and not waiting for children..."
   sleep 30
   echo "Parent exiting"
   EOL
   
   chmod +x zombie.sh
   ```

2. Jalankan script:
   ```bash
   ./zombie.sh &
   ```

3. Periksa status zombie:
   ```bash
   ps aux | grep defunct
   ```

## Praktikum 5: Menjadwalkan Proses dengan cron

### 5.1. Membuat Crontab

1. Edit crontab untuk pengguna saat ini:
   ```bash
   crontab -e
   ```

2. Tambahkan entri untuk menjalankan script setiap menit:
   ```
   # Run script every minute
   * * * * * echo "Cron executed at $(date)" >> /tmp/cron_log.txt
   ```

3. Simpan dan keluar dari editor.

4. Verifikasi crontab:
   ```bash
   crontab -l
   ```

5. Tunggu beberapa menit dan periksa hasilnya:
   ```bash
   cat /tmp/cron_log.txt
   ```

### 5.2. Format Crontab Lanjutan

1. Edit crontab lagi:
   ```bash
   crontab -e
   ```

2. Tambahkan beberapa entri dengan format yang berbeda:
   ```
   # Format: minute hour day month day_of_week command
   
   # Run at 2:30 AM every day
   30 2 * * * echo "2:30 AM task" >> /tmp/cron_schedule.txt
   
   # Run every 5 minutes
   */5 * * * * echo "Every 5 minutes" >> /tmp/cron_schedule.txt
   
   # Run at 10:30 PM on weekdays (Monday-Friday)
   30 22 * * 1-5 echo "Weekday night" >> /tmp/cron_schedule.txt
   
   # Run on the 1st of every month at 5:00 AM
   0 5 1 * * echo "Monthly task" >> /tmp/cron_schedule.txt
   ```

3. Simpan dan keluar dari editor.

## Praktikum 6: Debugging Proses dengan strace

### 6.1. Menggunakan strace

1. Instal strace jika belum tersedia:
   ```bash
   apt-get update && apt-get install -y strace
   ```

2. Buat script sederhana:
   ```bash
   cat > hello.sh << 'EOL'
   #!/bin/bash
   echo "Hello, world!"
   cat /etc/hostname
   sleep 2
   date
   EOL
   
   chmod +x hello.sh
   ```

3. Trace system calls:
   ```bash
   strace ./hello.sh
   ```

4. Trace hanya system calls tertentu:
   ```bash
   strace -e open,read,write ./hello.sh
   ```

5. Trace process yang sudah berjalan:
   ```bash
   # Jalankan di background
   ./infinite.sh &
   
   # Ambil PID
   PID=$(pgrep -f infinite.sh)
   
   # Trace
   strace -p $PID
   ```

### 6.2. Menganalisis Output strace

1. Simpan output strace ke file:
   ```bash
   strace -o trace.log ./hello.sh
   ```

2. Analisis statistik system calls:
   ```bash
   strace -c ./hello.sh
   ```

3. Periksa file descriptor yang dibuka:
   ```bash
   strace -e open,openat ./hello.sh
   ```

## Latihan Mandiri

Setelah menyelesaikan semua praktikum, cobalah latihan-latihan berikut:

1. **Runaway Process**: 
   - Buat script yang menggunakan CPU secara intensif
   - Jalankan beberapa instance
   - Gunakan `top`, `ps`, dan `kill` untuk mengidentifikasi dan menghentikan proses tersebut

2. **Monitoring Resources**:
   - Buat script monitoring sederhana yang mencatat penggunaan CPU dan memori
   - Jadwalkan agar berjalan setiap 15 menit menggunakan cron
   - Analisis hasil untuk mengidentifikasi tren penggunaan sumber daya

3. **Simulasi Skenario Masalah**:
   - Buat script yang menyimulasikan proses yang kehabisan memori
   - Gunakan `strace` untuk mengidentifikasi system calls yang gagal
   - Gunakan tools monitoring untuk mengidentifikasi masalah

4. **Daemon Process**:
   - Buat script sederhana yang berjalan sebagai daemon
   - Gunakan sinyal untuk mengendalikannya (reload config, restart, shutdown)
   - Praktikkan mengelola daemon dengan various systemd commands

5. **Optimalisasi Proses**:
   - Identifikasi proses yang berjalan dengan prioritas default
   - Bereksperimen dengan berbagai nilai nice
   - Ukur dan bandingkan kinerja dengan prioritas yang berbeda