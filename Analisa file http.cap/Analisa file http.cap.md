Saya akan merapikan dokumen markdown tersebut agar lebih terstruktur dan mudah dibaca:

# Tugas Workshop Administrasi Jaringan [REVIEW]

## Tugas 1: Analisa file http.cap

### 1. Download File
- Download `http.cap` di [SampleCaptures](https://wiki.wireshark.org/SampleCaptures)

### 2. Langkah-langkah Pengerjaan
- Buka Wireshark dan load file `http.cap`
- Gunakan filter untuk menampilkan paket HTTP
- Analisis setiap paket untuk mendapatkan informasi yang diminta
- Catat hasil analisis dalam dokumen

### 3. Eksplorasi dan Analisis di Wireshark

#### IP Server dan Client
- IP Server: 65.208.228.223
- IP Client: 145.254.160.237

#### Versi HTTP
- Menggunakan HTTP versi 1.1 (HTTP/1.1)

#### Analisis Waktu
- Waktu client mengirim request: 0.911310 detik
- Waktu server menerima request: Selisih antara baris 5 (1.472 sec) dan 4 (0.911 sec) = 0.561 detik
- Waktu transfer dan response total: 3.935659 detik
  - Waktu client mengirim request: 0.911310 detik (paket #4)
  - Waktu server mengirim response: 4.846969 detik (paket #38)
  - Total waktu: 4.846969 - 0.911310 = 3.935659 detik

## Tugas 2: Analisis Gambar Types of Data Deliveries

### 1. Node to Node (Data Link Layer)
- Komunikasi antara dua node/perangkat jaringan yang terhubung langsung
- Terjadi di level paling dasar (router-ke-router atau komputer-ke-router)
- Menggunakan protokol data link layer seperti Ethernet
- Menggunakan alamat MAC untuk identifikasi perangkat
- Contoh: Komunikasi antara laptop dengan router WiFi rumah

### 2. Host to Host (Network Layer)
- Komunikasi end-to-end antara komputer sumber dan tujuan
- Mencakup seluruh path melalui internet
- Menggunakan protokol IP (Internet Protocol)
- Contoh: Komunikasi antara komputer pengguna dengan server website

### 3. Process to Process (Transport Layer)
- Komunikasi antara aplikasi/proses spesifik di komputer sumber dan tujuan
- Level tertinggi, menangani komunikasi antar aplikasi
- Menggunakan protokol TCP/UDP
- Menggunakan port number untuk identifikasi aplikasi
- Contoh: Komunikasi antara browser dengan web server

## Tugas 3: Resume Tahapan TCP

### 1. TCP Establishment (Three-Way Handshake)
1. **Langkah 1 (SYN)**
   - Client mengirim paket SYN ke server
   - Berisi sequence number awal acak (SEQ=100)
   - Flag SYN = 1

2. **Langkah 2 (SYN-ACK)**
   - Server merespon dengan paket SYN-ACK
   - Berisi acknowledgment (ACK=101)
   - Sequence number server (SEQ=300)
   - Flag SYN dan ACK = 1

3. **Langkah 3 (ACK)**
   - Client mengirim ACK final
   - Berisi acknowledgment (ACK=301)
   - Koneksi siap untuk transfer data

### 2. Data Transfer

#### Segmentasi
- Data dipecah menjadi segmen-segmen kecil
- Setiap segmen memiliki sequence number
- Maximum Segment Size (MSS) menentukan ukuran maksimal

#### Flow Control
- Menggunakan sliding window
- Receiver window menunjukkan kapasitas buffer penerima
- Mencegah overloading pada penerima

#### Error Control
- Menggunakan checksums untuk deteksi error
- Retransmission untuk segmen hilang/rusak
- Acknowledgment untuk konfirmasi penerimaan

### 3. TCP Termination (Four-Way Handshake)

1. **Langkah 1 (FIN)**
   - Client mengirim FIN ke server
   - Menandakan client selesai mengirim data
   - Client masih bisa menerima data

2. **Langkah 2 (ACK)**
   - Server mengirim ACK untuk FIN client
   - Server masih bisa mengirim data
   - Client menunggu FIN dari server

3. **Langkah 3 (FIN)**
   - Server mengirim FIN ke client
   - Menandakan server juga selesai
   - Semua data sudah terkirim

4. **Langkah 4 (ACK)**
   - Client mengirim ACK terakhir
   - Koneksi terputus setelah timeout
   - Resources dibebaskan