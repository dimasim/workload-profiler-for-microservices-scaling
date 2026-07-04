# Rancangan Layanan & Fitur Microservices (WorldCup98 Dataset)

Dokumen ini berisi spesifikasi perancangan rute akses dari dataset log akses **WorldCup98 (TR2 - sampel 24 jam)** menjadi 3 layanan microservices virtual untuk kebutuhan pengujian proactive auto-scaling Anda.

---

## 1. Pemetaan Rute ke Virtual Microservices

Berdasarkan metadata biner log WorldCup98 (kolom `filetype` dan `method`), trafik didekomposisi menjadi 3 jenis layanan dengan karakteristik beban kerja (*workload profile*) yang berbeda:

| Nama Layanan (Microservice) | Rute Log WorldCup98 | Karakteristik Beban Kerja | Perilaku Auto-Scaling |
| :--- | :--- | :--- | :--- |
| **1. DynamicAPI_Service** | Rute tipe `DYNAMIC` (kode `6`) ATAU HTTP `POST` (kode `2`) | **CPU-Heavy & Computational Bound** | Memproses logika backend dinamis (seperti skrip CGI, input form). Sangat sensitif terhadap lonjakan trafik mendadak (*bursty traffic*). |
| **2. Content_Service** | Rute tipe `HTML` (kode `0`) ATAU `DIRECTORY` (kode `10`) | **I/O-Heavy & Database-Bound** | Menyajikan halaman web berita/jadwal pertandingan. Membaca data dari penyimpanan disk/database (Read-Heavy). |
| **3. Media_Service** | Rute tipe `IMAGE` (kode `1`) | **Network-Heavy & Memory Bound** | Menyajikan file gambar statis. Memiliki jumlah request paling dominan (~88% trafik). Biasanya dibantu oleh RAM cache. |
| **Others** | Rute tipe lainnya (Audio, Video, Java, dll.) | **General / Diabaikan** | Trafik umum yang tidak dipetakan ke layanan utama. |

---

## 2. Catatan Historis & Konseptual (Penting untuk Sidang Skripsi)

### A. Kenapa `Media_Service` berupa Gambar Statis, bukan Video Streaming?
* **Teknologi Era 1998**: Pada tahun 1998, mayoritas pengguna mengakses internet menggunakan koneksi **Dial-up** (lewat kabel telepon rumah) dengan kecepatan sangat rendah (33.6 Kbps - 56 Kbps).
* **Ketiadaan Live Streaming**: Infrastruktur internet saat itu belum mampu melayani streaming video siaran langsung secara massal. YouTube baru diluncurkan pada tahun 2005.
* **Aset Statis**: Oleh karena itu, aset media pada situs *www.france98.com* hampir sepenuhnya berupa file gambar berukuran kecil (format `.gif` dan `.jpg`) seperti cuplikan foto gol setelah pertandingan selesai, logo sponsor, dan aset antarmuka.

### B. Kelayakan Trafik untuk Project Auto-Scaling
* **Sifat Bursty**: Trafik log WorldCup98 memiliki pola fluktuasi harian yang sangat ekstrem dan tidak terduga (misalnya lonjakan drastis saat gol terjadi atau pertandingan sedang berlangsung).
* **Multi-Service Testing**: Dengan memisahkan beban kerja dinamis (`DynamicAPI_Service`) dari statis, Anda dapat memvalidasi kinerja auto-scaler proaktif berbasis LSTM/GRU Anda dalam memicu scaling pada microservice yang tepat secara independen (*decoupled scaling*).
