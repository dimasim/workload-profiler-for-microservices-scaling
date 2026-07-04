# Panduan Analisis Dataset & Konsep Skenario Stress Testing

Dokumen ini adalah cetak biru (*blueprint*) dan panduan langkah demi langkah (*walkthrough*) untuk melakukan riset dataset baru, memformulasikan arsitektur microservice, dan merancang konsep pengujian beban kerja (*workload/stress testing*) menggunakan Locust pada repository baru Anda.

---

## TAHAP 1: Analisis Dataset pada Jupyter Notebook (`.ipynb`)

Tujuan dari tahap ini adalah memahami karakteristik beban kerja riil secara statistik menggunakan Python (Pandas, Matplotlib/Seaborn).

### Langkah-Langkah Analisis:
1. **Memuat Data (*Data Ingestion*)**:
   * Impor log server (format `.csv` atau `.log`) menggunakan Pandas.
   * Lakukan *parsing* timestamp mentah ke tipe data `datetime64`.
2. **Pembersihan Data (*Data Cleaning*)**:
   * Hapus baris data yang memiliki nilai kosong atau *corrupted*.
   * Filter endpoint statis seperti gambar (`.png`, `.jpg`), stylesheet (`.css`), atau script (`.js`) jika fokus pengujian adalah performa API backend.
3. **Agregasi Waktu ke Tingkat Detik (RPS)**:
   * Lakukan *resampling* data log berdasarkan interval waktu 1 detik.
   * Hitung frekuensi baris data yang masuk pada setiap detiknya untuk mendapatkan metrik **Requests Per Second (RPS)** yang asli.
4. **Analisis Deskriptif & Visualisasi**:
   * Buat grafik runtun waktu (*time-series plot*) dari fluktuasi RPS untuk mendeteksi lonjakan trafik (*burst/spike*).
   * Identifikasi statistik penting: **RPS Minimum**, **RPS Rata-rata**, dan **RPS Puncak (Peak)**.

---

## TAHAP 2: Formulasi Rencana Microservices

Setelah pola trafik dipahami, Anda perlu menerjemahkan rute-rute API tersibuk di dalam log menjadi layanan microservice yang relevan untuk diuji autoscaling-nya.

### Contoh Formulasi Layanan:
* **Service A (CPU-Heavy / Computational Bound)**:
  * *Rute Log Asli*: `/api/v1/auth/login` atau `/api/v1/checkout`.
  * *Tugas*: Melakukan proses enkripsi (misal bcrypt), penguraian tanda tangan JWT, atau pemrosesan matematika kompleks.
* **Service B (I/O-Heavy / Database Bound)**:
  * *Rute Log Asli*: `/api/v1/items/search` atau `/api/v1/analytics/log`.
  * *Tugas*: Melakukan query database intensif, pembacaan disk cache, atau penulisan data masal (*bulk write*).


---

## TAHAP 3: Konsep Skenario Pengujian Beban Kerja (Locust)

Pada tahap ini, Anda tidak perlu langsung menulis kode Python Locust, melainkan menyusun **konsep teoritis** dan **skenario pengujian** berdasarkan karakteristik dataset yang telah dianalisis.

### Komponen Konsep Skenario:

#### A. Pembagian Profil Beban Kerja (*Workload Profile Mix*)
Tentukan persentase distribusi panggilan API oleh user tiruan agar menyerupai pola log asli.
* *Contoh*:
  * **90% User** melakukan request ke **Service B** (Membaca data/kuis).
  * **10% User** melakukan request ke **Service A** (Login/Autentikasi).

#### B. Skenario Pengetesan (*Testing Skenarios*)
Rancang 3 jenis skenario pengujian beban untuk mengevaluasi responsivitas autoscaler:
1. **Skenario Trafik Normal (*Baseline Test*)**:
   * *Konsep*: Beban kerja dikirim secara konstan pada tingkat RPS rata-rata dataset historis untuk menguji kestabilan sistem dalam kondisi normal.
2. **Skenario Naik Bertahap (*Ramp-up Test*)**:
   * *Konsep*: Jumlah user dinaikkan secara bertahap (misal naik 10 user setiap 30 detik) hingga mencapai puncak trafik tertinggi pada dataset. Berguna untuk melihat seberapa mulus auto-scaling beradaptasi secara proaktif.
3. **Skenario Lonjakan Mendadak (*Spike/Stress Test*)**:
   * *Konsep*: Trafik mendadak dilipatgandakan melebihi kapasitas puncak normal dalam durasi waktu sangat singkat (misal melompat dari 10 RPS ke 300 RPS dalam 5 detik). Berguna untuk menguji apakah sistem mengalami *cascading failure* sebelum autoscaler sempat melakukan penyediaan kontainer baru (*provisioning latency*).
