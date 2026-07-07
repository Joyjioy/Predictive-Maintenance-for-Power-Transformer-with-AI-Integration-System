# PRODUCT REQUIREMENT DOCUMENT (PRD)
## Digital Twin & Predictive Maintenance Dashboard (MVP Edition: Time-to-Fault & Asynchronous Data)

**Doc Version:** 1.0 (Pragmatic MVP)
**Author:** Mahasiswa Kerja Praktik Teknik Elektro/Sistem Kontrol ITB
**Target Audience:** Mentor Lapangan, Reliability Engineer, & Management PT Chandra Asri Pacific Tbk
**Date:** Juli 2026

---

### 1. EXECUTIVE SUMMARY & PROBLEM STATEMENT

#### 1.1 Latar Belakang
Transformator daya utama (*Main Substation Transformer*) menuntut keandalan operasional tingkat tinggi. Tantangan terbesar di lapangan saat ini adalah perbedaan frekuensi ketersediaan data. Data *Dissolved Gas Analysis* (DGA) sering kali lebih rutin dipantau atau disimulasikan ketimbang data *Oil Analysis* (OA) kelistrikan-fisik (seperti Tegangan Tembus/BDV dan Keasaman) yang mengandalkan uji laboratorium eksternal setiap 6 hingga 12 bulan sekali. Hal ini menciptakan celah visibilitas yang menghambat transisi dari perawatan preventif menjadi prediktif.

#### 1.2 Pernyataan Masalah (*Problem Statement*)
1. **Asinkronisasi Data (Data Gap):** Ketidaksesuaian waktu antara pengambilan sampel DGA dan pengujian OA membuat sistem pemantauan konvensional kesulitan memberikan gambaran yang utuh pada satu titik waktu (bulan berjalan).
2. **Kesesatan Prediksi Fisik:** Memaksa algoritma untuk meramal nilai parameter fisik (BDV, IFT) di masa depan tanpa data suhu dan pembebanan harian adalah cacat secara fundamental teknik.
3. **Ilusi RUL Berbasis Oli:** Memprediksi Sisa Umur (*Remaining Useful Life* / RUL) trafo murni dari data oli menghasilkan fluktuasi angka palsu ("Presisi Palsu"), karena oli dapat dipurifikasi, sementara umur sejati trafo berada pada kertas isolasinya.

#### 1.3 Solusi yang Diusulkan (Pragmatic MVP)
Membangun purwarupa *Minimum Viable Product* (MVP) **Digital Twin Dashboard di Power BI** yang beroperasi selaras dengan realitas operasional ketersediaan data di lapangan. Sistem MVP ini berfokus pada:
* **Metode Penahanan Data (Jangkar Realita):** Membekukan status parameter OA terakhir hingga ada *update* lab terbaru.
* **Prognosis Waktu Menuju Kegagalan (*Time-to-Fault*):** Berbasis deret waktu (historis) evolusi gas DGA.
* **Kalkulasi RUL Sejati:** Berbasis ekstraksi molekul Furan (2-FAL) dari kertas selulosa.

---

### 2. OBJECTIVES & PROJECT SCOPE

#### 2.1 Tujuan Proyek (*Objectives*)
* Mengakomodasi kesenjangan jadwal uji lab (DGA vs OA) secara elegan tanpa menghasilkan alarm palsu atau metrik kosong di dasbor.
* Menerjemahkan laju tren historis DGA menjadi informasi zona waktu peringatan dini (misal: "Proyeksi D1 dalam 1-3 Bulan").
* Memberikan kepastian metrik degradasi aset menggunakan perhitungan standar internasional (IEC 61198) untuk kertas trafo.

#### 2.2 Batasan Ruang Lingkup (*Scope Boundary*)
| Kategori | In-Scope (MVP) | Out-of-Scope (Fase Selanjutnya) |
| :--- | :--- | :--- |
| **Aset Target** | 1 unit Trafo Kritis pabrik (*Single-Asset PoC*). | Ekspansi massal multi-aset. |
| **Integrasi Data** | Pemrosesan longitudinal data CSV/Excel (DGA, Furan, OA). | *Real-time Streaming* sensor IoT (Suhu, Beban). |
| **Target Prediksi** | Prediksi Deret Waktu DGA (Time-to-Fault) & Furan. | Prediksi harian degradasi fisik (BDV/Acidity). |

---

### 3. ARSITEKTUR SISTEM & ALUR KERJA (*WORKFLOW*)

MVP ini menggunakan Python sebagai *backend prognostic engine* dan DAX Power BI sebagai pengendali antarmuka (*Asynchronous Data Handler*).

```text
[Dataset Historis CSV: DGA, Furan, OA] 
           │
           ▼
[Python Engine - Data Pre-Processing & ML]
           ├── 1. Forecasting DGA (ARIMA/Prophet) ─► Prediksi Profil Gas Masa Depan
           ├── 2. Diagnosis Masa Depan ────────────► Eksekusi Duval Triangle AI dari Data Proyeksi
           └── 3. Kalkulasi DP Kertas via Furan ───► True RUL (Sisa Umur Valid)
           │
           ▼
[Power BI Dashboard - Presentasi Visual]
           ├── Panel DGA Forecasting (Grafik Tren + Confidence Interval)
           ├── Panel Peringatan (Zona Waktu Menuju Fault D1/D2/T2 dll.)
           └── Panel OA Terakhir (Gauge Meter Statis dengan DAX LASTNONBLANKVALUE)
```
### 4. TECHNICAL SPECIFICATIONS & CORE CAPABILITIES

#### 4.1 Lapis 1: Asynchronous Data Fusion (Penahanan Data OA)
Mengingat data *Oil Analysis* (BDV, Acidity, Water Content, IFT) hanya diperbarui secara sporadis (misal: setahun sekali), MVP ini menggunakan pendekatan **Jangkar Realita (*Reality Anchor*)**.
* Dasbor Power BI menggunakan fungsi DAX `LASTNONBLANKVALUE` untuk parameter fisik.
* **Mekanisme:** Jika lab mengeluarkan hasil BDV 60 kV pada bulan Januari, visualisasi *Gauge Meter* untuk BDV akan menahan dan menampilkan angka 60 kV secara statis pada bulan Februari, Maret, hingga ada input data lab terbaru di bulan Juli.
* **Tujuan:** Menghindari manipulasi matematis tebakan untuk besaran fisik yang membingungkan operator, sembari tetap menjaga *readiness* dasbor untuk menampilkan data terbaru kapan pun lab merilis hasilnya.

#### 4.2 Lapis 2: Sisa Umur Kertas (Furan - IEC 61198)
Umur trafo tidak diukur dari pelumasnya, melainkan direpresentasikan oleh kekuatan kertas isolasi (*Degree of Polymerization* / DP). MVP ini mengekstrak kadar 2-Furfural (2-FAL) dan menghitung nilai DP menggunakan Persamaan Chendong:

$$DP = \frac{\log_{10}(\text{2-FAL}) - 1.51}{-0.0035}$$

Nilai DP dikonversi menjadi persentase Indeks Kesehatan Kertas (RUL Sejati). Jika DP mendekati batas kegagalan mekanis (angka 200), dasbor akan memicu alarm peringatan dini.

#### 4.3 Lapis 3: Time-to-Fault Forecasting (Prediksi DGA)
Alih-alih memberikan satu angka "Sisa Waktu" yang rentan berfluktuasi, MVP ini menggunakan sistem **Zona Waktu Proyeksi**:
* AI (model *Time-Series*) memproyeksikan laju pembentukan gas pembakaran utama (seperti Ethylene dan Acetylene) ke masa depan.
* Titik proyeksi tersebut disilangkan ke dalam batas Segitiga Duval (Otak 1 Klasifikasi).
* Keluaran diubah menjadi rentang peringatan proaktif. Contoh output pada dasbor: **"Proyeksi D2: Zona Waktu Menengah (3 - 6 Bulan Ke Depan)."** Hal ini memberikan kenyamanan UI/UX sekaligus akurasi teknis tanpa presisi palsu.

---

### 5. DATA REQUIREMENTS (KEBUTUHAN DATA)

MVP ini dikembangkan dan diuji menggunakan kumpulan dataset longitudinal (riwayat DGA, Furan, dan OA) dengan kebutuhan struktur spesifik:

| Kategori Parameter | Kolom Wajib (*Mandatory*) | Kegunaan dalam MVP |
| :--- | :--- | :--- |
| **Sumbu Waktu** | `Tanggal_Uji` / `Bulan_Ke` | Sumbu temporal mutlak untuk eksekusi algoritma *forecasting*. |
| **DGA (Gas Utama)** | `H2`, `CH4`, `C2H6`, `C2H4`, `C2H2` | Bahan baku model proyeksi mesin waktu & klasifikasi Duval. |
| **Parameter Kertas** | `Furan_2FAL` | Parameter tunggal penghitung RUL (Kalkulasi Chendong). |
| **Kualitas Fisik (OA)**| `BDV`, `Acidity`, `Water`, `IFT` | Disimpan dan ditahan sebagai Jangkar Realita (Status Lab Terakhir). |

---

### 6. TECHNOLOGY STACK & DEPENDENCIES

* **Python (Scikit-Learn, Statsmodels, Pandas):** Untuk pembersihan data temporal, penarikan trendline/forecasting, dan penghitungan DP secara *batch processing*.
* **Microsoft Power BI:** Untuk implementasi lapisan antarmuka pengguna, pengaturan filter asinkron (DAX), dan penyajian metrik prediksi secara visual.
* **Standar Industri Referensi:** IEC 61198 (Metode Uji Senyawa Furanic) & IEEE C57.104 (Interpretasi DGA).
