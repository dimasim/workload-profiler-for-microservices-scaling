# Metodologi Pemilihan & Slicing Data (WorldCup98 Dataset)

Dokumen ini mendokumentasikan pola-pola pemilihan berkas log dari dataset **WorldCup98** yang umum digunakan dalam penelitian ilmiah/jurnal untuk eksperimen *resource provisioning* dan *proactive auto-scaling*. Informasi ini sangat berguna untuk penyusunan **Bab 3 (Metodologi Penelitian)** skripsi Anda.

---

## 1. Pola Pemilihan Rentang Data (Slicing Patterns)

Ada 3 pola utama yang biasa digunakan peneliti untuk mengambil sampel data dari total 88 hari log WorldCup98:

### Pola 1: Mengambil Rentang Berurutan Periode Turnamen (2 Minggu)
* **Contoh Nyata**: Penelitian **PhoenixCloud** (sistem provisioning resource cloud) menggunakan trace World Cup dari tanggal **7 – 20 Juni 1998** sebagai beban kerja (*workload*) pengujian mereka.
* **Alasan & Konseptual**: Periode ini sengaja dipilih karena mencakup beberapa hari menjelang turnamen hingga beberapa hari pertama turnamen dimulai (pembukaan tanggal 10 Juni 1998). Pada rentang ini, rasio kenaikan trafik dari kondisi normal ke kondisi puncak terlihat sangat ekstrem, riil, dan sangat baik untuk menguji reaksi adaptif dari sistem provisioning.

### Pola 2: Mengambil Sampel Hari Terpisah (Rendah / Sedang / Tinggi) - *Sangat Direkomendasikan*
Pola ini digunakan oleh mirror dataset di **Zenodo** dengan memilih 3 hari spesifik yang mewakili 3 tingkat beban kerja berbeda:
1. **`wc_day9_1.gz` (4 Mei 1998)**: Representasi **Trafik Rendah** (baseline normal, jauh sebelum turnamen dimulai).
2. **`wc_day25_1.gz` (20 Mei 1998)**: Representasi **Trafik Sedang** (mulai terjadi peningkatan aktivitas).
3. **`wc_day28_1.gz` (23 Mei 1998)**: Representasi **Trafik Tinggi** (mendekati hari turnamen).

* **Kelebihan untuk Skripsi**:
  * Ukuran data jauh lebih ringan dan ramah untuk RAM laptop Anda dibanding mengolah rentang waktu berurutan yang sangat panjang.
  * Anda bisa menunjukkan perbandingan performa model prediksi auto-scaling Anda di tiga kondisi beban yang kontras secara adil dan terstruktur.

### Pola 3: Pemrosesan Global dan Pemotongan Bagian Puncak (Peak Filtering)
* **Pendekatan**: Memproses seluruh 88 hari data log menjadi data agregat harian terlebih dahulu, lalu secara otomatis memotong (*slice*) bagian puncak di sekitar pertandingan-pertandingan krusial (misal: Babak Pembukaan 10 Juni, Semifinal, atau Final 12 Juli) untuk dijadikan skenario pengujian beban ekstrem.

---

## 2. Cara Menghitung File Log (`wc_dayN`) ke Tanggal Riil

Penamaan berkas log biner asli WorldCup98 mengikuti urutan hari pengumpulan sejak tanggal **26 April 1998** (sebagai Hari ke-1):

* **`wc_day1`**  → 26 April 1998 (File log kosong / inisialisasi)
* **`wc_day5`**  → 30 April 1998
* **`wc_day9`**  → 4 Mei 1998 (Trafik Rendah)
* **`wc_day25`** → 20 Mei 1998 (Trafik Sedang)
* **`wc_day28`** → 23 Mei 1998 (Trafik Tinggi)
* **`wc_day46`** → 10 Juni 1998 (Upacara Pembukaan Piala Dunia)
* **`wc_day78`** → 12 Juli 1998 (Pertandingan Final)

> [!TIP]
> **Rumus Konversi Hari (`N`):**
> Jika Anda ingin mencari nomor file untuk tanggal tertentu, hitung jumlah hari selisih sejak tanggal **26 April 1998** sebagai baseline hari ke-1.
