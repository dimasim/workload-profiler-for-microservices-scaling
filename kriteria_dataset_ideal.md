# Kriteria Dataset Ideal (Per-Detik) untuk Proactive Auto-Scaling Microservices

Dokumen ini menjelaskan kriteria, spesifikasi, dan contoh format dataset baru yang harus dicari untuk kebutuhan skripsi Anda. Dataset ini ditargetkan untuk mengganti dataset lama agar memiliki **resolusi waktu per-detik (RPS) asli** secara bawaan tanpa memerlukan rekayasa/sintesis data.

---

## 1. Karakteristik Utama Dataset yang Harus Dicari

Untuk memastikan validitas akademis dan kemudahan implementasi pada model LSTM/GRU serta load testing (Locust), dataset baru harus memenuhi kriteria berikut:

### A. Granularitas Waktu Tingkat Detik (*Second-Level Precision*)
* **Kriteria**: Timestamp data wajib memiliki presisi waktu hingga detik (format: `YYYY-MM-DD HH:MM:SS` atau Epoch Unix dengan presisi detik).
* **Alasan**: Menghilangkan tuduhan "manipulasi data" karena fluktuasi RPS diperoleh langsung dari log asli web server/gateway.

### B. Memiliki Informasi Endpoint/URI yang Jelas
* **Kriteria**: Log harus mencantumkan route API/web page yang diakses (misal: `/login`, `/search`, `/payment`, `/submit`).
* **Alasan**: Anda menguji arsitektur microservices (NestJS vs Go). Dataset harus memiliki minimal 2 rute berbeda untuk menyimulasikan beban kerja multi-layanan (misal: beban CPU-heavy vs I/O-heavy).

### C. Bersifat Dinamis dan *Bursty*
* **Kriteria**: Dataset harus menunjukkan pola trafik harian (*daily seasonality*), tren mingguan, serta lonjakan trafik mendadak (*bursty traffic*).
* **Alasan**: Auto-scaling proaktif berbasis LSTM/GRU baru bisa membuktikan keunggulannya dibanding reaktif threshold jika diuji pada pola trafik yang fluktuatif dan sulit diprediksi secara instan.

### D. Ukuran Baris Log yang Cukup
* **Kriteria**: Memiliki rentang waktu minimal 1 minggu hingga 1 bulan data mentah (minimal 50.000+ baris data agregat per detik).
* **Alasan**: Algoritma deep learning (LSTM/GRU) membutuhkan data latih yang cukup besar agar dapat mengenali pola tren secara akurat tanpa overfitting.

---

## 2. Contoh Format File CSV yang Ideal

Saat mengunduh log dari repositori publik (seperti Kaggle atau Github), pastikan data log mentah minimal memiliki kolom-kolom berikut:

```csv
timestamp,endpoint,method,status_code,response_time_ms
2026-07-04 08:00:01,/api/v1/auth/login,POST,200,85
2026-07-04 08:00:01,/api/v1/quiz/submit,POST,201,12
2026-07-04 08:00:02,/api/v1/auth/login,POST,401,90
2026-07-04 08:00:02,/api/v1/quiz/submit,POST,201,15
```

Atau jika dataset sudah dalam bentuk agregat hasil ekstraksi, formatnya harus seperti ini (RPS per detik):

```csv
timestamp,auth_service_rps,quiz_service_rps,total_rps
2026-07-04 08:00:01,12,150,162
2026-07-04 08:00:02,15,188,203
2026-07-04 08:00:03,8,95,103
```

---

## 3. Rekomendasi Sumber & Tempat Pencarian Dataset Publik

Berikut adalah repositori dataset open-source yang paling sering digunakan untuk riset load testing dan auto-scaling cloud:

### A. Repositori Klasik Log Server (Sangat Direkomendasikan)
1. **NASA Kennedy Space Center HTTP Logs**:
   * *Deskripsi*: Log akses HTTP server NASA selama dua bulan. Memiliki timestamp tingkat detik.
   * *Pencarian*: Cari di Kaggle dengan keyword `"NASA HTTP Web Server Logs"`.
2. **ClarkNet HTTP Server Logs**:
   * *Deskripsi*: Rekaman trafik web server komersial selama 2 minggu dengan stempel waktu detik.
3. **Saskatchewan HTTP Server Logs**:
   * *Deskripsi*: Dataset log server akademik berskala detik.

### B. Repositori Benchmark Cloud Modern
1. **Wikipedia Pageviews Dataset (Hourly/Per-Second Sample)**:
   * *Deskripsi*: Log lalu lintas akses halaman Wikipedia dengan granularitas sangat halus.
2. **Seco-Cloud / Azure Public Dataset (Workload Logs)**:
   * *Deskripsi*: Dataset performa VM/Container yang dirilis Microsoft Azure atau Google Cluster (Google Cluster Data 2011/2019). Biasanya berupa log utilisasi CPU/RAM per detik/menit.

---

## 4. Tips Kata Kunci Pencarian di Google Scholar / Kaggle
* `"web server access logs csv second precision"`
* `"http request logs timestamp dataset"`
* `"nasa web server access logs csv"`
* `"microservice workload trace dataset second level"`
