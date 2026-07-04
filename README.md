# Workload Profiler for Microservices Scaling

Repository ini berisi perkakas (*tools*), rancangan arsitektur microservices virtual, dan Jupyter Notebook untuk menganalisis berbagai dataset log HTTP akses publik guna mendukung riset **Proactive Auto-scaling** berbasis deep learning (LSTM/GRU).

Tujuan utama proyek ini adalah mengurai log trafik server web dunia nyata menjadi profil beban kerja tingkat detik (**Requests Per Second - RPS**) per virtual microservice. Hasil agregasinya dapat digunakan langsung sebagai umpan beban kerja (*workload replay*) dalam stress testing menggunakan Locust.

---

## 📂 Struktur Repositori

```text
├── dataset/                         # Folder lokal hasil agregasi (di-ignore oleh git)
│   ├── aggregated_worldcup98_rps.csv
│   ├── aggregated_clarknet_rps.csv
│   └── ...
├── analysis_worldcup98.ipynb        # Analisis Log WorldCup98 (Biner - 24 Jam)
├── analysis_clarknet.ipynb          # Analisis Log ClarkNet (ASCII - 7 Hari)
├── analysis_nasa.ipynb              # Analisis Log NASA (CLF - 28 Hari)
├── analysis_saskatchewan.ipynb      # Analisis Log U-Saskatchewan (ASCII - 7 Bulan)
├── analysis_epa.ipynb               # Analisis Log EPA (ASCII - 24 Jam)
├── analysis_sdsc.ipynb              # Analisis Log SDSC (ASCII - 24 Jam)
├── analysis_calgary.ipynb           # Analisis Log Calgary (ASCII - 353 Hari)
├── analysis_clean_log.ipynb         # Analisis Log Kaggle Terfilter (Anti-Cyberattack)
├── rancangan_fitur_worldcup98.md   # Cetak biru microservices WorldCup98
├── rancangan_fitur_clarknet.md     # Cetak biru microservices ClarkNet
├── panduan_analisis_dan_skenario.md# Panduan umum formulasi Locust & autoscaling
└── README.md                        # Dokumentasi utama repositori
```

---

## 📊 Ringkasan Komparasi Dataset Hasil Analisis

Seluruh dataset telah di-parsing secara efisien dan di-resample ke interval **1 detik (RPS)**. Berikut adalah komparasi statistik hasil agregasi untuk referensi skripsi Anda:

| Dataset | Rentang Waktu Sampel | Rata-rata RPS | Puncak (Peak) RPS | Karakteristik & Kelayakan |
| :--- | :--- | :--- | :--- | :--- |
| **WorldCup98 (TR2)** | 24 Jam | **14.61 RPS** | **71.00 RPS** | **Rekomendasi Utama**. Trafik sangat dinamis dan *bursty* akibat gol pertandingan bola. Sangat menantang untuk model prediksi. |
| **ClarkNet-HTTP** | **7 Hari Penuh** | 2.73 RPS | 45.00 RPS | **Sangat Bagus**. Memiliki pola mingguan (*weekly seasonality* weekdays vs weekend) yang sangat kaya dan rute dinamis cgi/pl melimpah. |
| **website-log-access** | 24 Jam (Cleaned) | 1.91 RPS | **9.396 RPS** | Trafik asli dibersihkan dari *cyber attack* (83.5% dibuang). Memiliki rute microservice modern seperti `/usr/login` dan `/usr/register`. Puncaknya sangat ekstrem. |
| **NASA HTTP** | 28 Hari | 0.79 RPS | 20.00 RPS | Trafik bersih, stabil, dan musiman harian berulang yang bagus untuk baseline. |
| **Saskatchewan-HTTP** | 3 Hari Tersibuk | 0.23 RPS | 15.00 RPS | Log akademik universitas jangka panjang, cenderung tenang di luar peak. |
| **EPA-HTTP** | 24 Jam | 0.55 RPS | 13.00 RPS | Sangat ringan (~47k baris), namun durasinya terlalu pendek untuk model peramalan berkala. |
| **SDSC-HTTP** | 24 Jam | 0.32 RPS | 11.00 RPS | Sangat ringan (~28k baris) dengan format log kustom, namun durasi pendek. |
| **Calgary-HTTP** | 3 Hari Tersibuk | 0.07 RPS | 9.00 RPS | Sangat sepi karena merepresentasikan server riset akademik tahun 1994. |

---

## 🚀 Cara Menjalankan Analisis

### 1. Prasyarat
Pastikan Anda sudah menginstal dependensi Python berikut:
```bash
pip install pandas numpy matplotlib jupyter
```

### 2. Jalankan Jupyter Notebook
Jalankan server jupyter di workspace Anda:
```bash
jupyter notebook
```
Buka salah satu berkas notebook (misalnya [analysis_clarknet.ipynb](analysis_clarknet.ipynb)) untuk mengeksplorasi langkah-langkah pembersihan data, konversi timestamp, pemetaan microservices, dan plotting visualisasinya.
