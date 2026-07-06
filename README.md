# Predictive-Maintenance-for-Power-Transformer-with-AI-Integration-System

# PRODUCT REQUIREMENT DOCUMENT (PRD)
## Digital Twin & Predictive Maintenance Dashboard Transformator Daya Berbasis AI (Hybrid IEEE/IEC Standar)

**Doc Version:** 1.0  
**Author:** Mahasiswa Kerja Praktik Teknik Elektro/Sistem Kontrol ITB  
**Target Audience:** Mentor Lapangan, Reliability Engineer, & Management PT Chandra Asri Pacific Tbk  
**Date:** Juli 2026  

---

### 1. EXECUTIVE SUMMARY & PROBLEM STATEMENT

#### 1.1 Latar Belakang
Transformator daya utama (*Main Substation Transformer*) merupakan jantung distribusi listrik pabrik petrokimia yang menuntut keandalan (*reliability*) 100% dan *zero unplanned downtime*. Metode pemantauan rutin saat ini umumnya masih bersifat statis (*snapshot classification*), di mana analisis laboratorium minyak isolasi (Dissolved Gas Analysis/DGA & *Oil Analysis*) dievaluasi secara terpisah dan hanya menilai kondisi pada satu titik waktu saat sampel diambil.

#### 1.2 Pernyataan Masalah (*Problem Statement*)
1. **Analisis Terfragmentasi:** Evaluasi gas terlarut (DGA) sering kali dipisahkan dari analisis degradasi fisik-kimiawi minyak isolasi (*Oil Analysis*), sehingga teknisi kehilangan gambaran holistik kesehatan transformator.
2. **Keterbatasan Evaluasi Statis:** Nilai parameter di bawah ambang batas bahaya sering kali dianggap "Aman", padahal laju kenaikannya (*rate of change*) mungkin sedang melonjak eksponensial dalam rentang waktu singkat.
3. **Kebutuhan Preskriptif:** Manajemen dan teknisi tidak hanya membutuhkan diagnosis *jenis kerusakan saat ini*, tetapi membutuhkan perkiraan **kapan transformator berisiko mengalami kegagalan (*Remaining Useful Life / RUL*)** dan **tindakan perawatan preskriptif apa yang harus segera dieksekusi**.

#### 1.3 Solusi yang Diusulkan (*Proposed Solution*)
Membangun purwarupa (*Proof of Concept / PoC*) **Digital Twin & Predictive Dashboard terintegrasi Microsoft Power BI** untuk 1 unit Transformator Kritis pabrik. Sistem ini memadukan 3 lapis analisis (*Hybrid Intelligence*):
* **Rel 1 (Standard Deterministik):** Skrining batas kesehatan minyak isolasi berdasarkan IEEE C57.104 & IEC 60422 Edition 4.0.
* **Rel 2 (Geometri Diagnosis):** Pemetaan klasifikasi kegagalan aktif menggunakan Segitiga Duval Klasik & Rasio Rogers.
* **Rel 3 (Machine Learning Prognostics):** Prediksi laju degradasi bersambung (*Time-Series Forecasting*) dan estimasi sisa umur pakai (*Remaining Useful Life / RUL*).

---

### 2. OBJECTIVES & PROJECT SCOPE

#### 2.1 Tujuan Proyek (*Objectives*)
* **Membangun sistem pemantauan terpusat** di Power BI yang memproses data uji lab Excel secara otomatis tanpa intervensi manual yang rumit.
* **Mengurangi alarm palsu (*False Alarm*)** melalui validasi silang antara kondisi normal standar IEEE dengan kecerdasan buatan (*Machine Learning*).
* **Memberikan visibilitas masa depan (*Prognostics*)** mengenai kurva degradasi parameter kritis hingga 12–24 bulan ke depan.
* **Menghasilkan rekomendasi tindakan preskriptif** sesuai standar IEC 60422 Tabel 5 & 6 (*Reconditioning*, *Reclaiming*, atau *Passivation*).

#### 2.2 Batasan Ruang Lingkup (*Scope Boundary*)
| Kategori | In-Scope (Akan Dikerjakan) | Out-of-Scope (Bukan Prioritas Saat Ini) |
| :--- | :--- | :--- |
| **Aset Target** | Fokus pada **1 unit Trafo Kritis** pabrik (*Single-Asset PoC*). | Implementasi massal untuk seluruh trafo distribusi kecil di pabrik. |
| **Integrasi Data** | Impor historis file Excel (*Offline/Batch Processing*) ke Power BI. | Pemasangan sensor *real-time* langsung via IoT ke fisik trafo. |
| **Domain Analisis** | DGA (IEEE C57.104/Duval) + Oil Analysis (IEC 60422). | Analisis *Furanic compound* mendalam atau uji tegangan impuls fisik lab. |

---

### 3. ARSITEKTUR SISTEM & ALUR KERJA (*WORKFLOW*)

Sistem dirancang dengan alur pemrosesan data linier di dalam ekosistem Microsoft Power BI yang didukung oleh skrip *backend* Python di dalam Power Query:


[Data Lab Excel (Longitudinal)] 
           │
           ▼
[Power BI Power Query Editor] ────────► [Python Engine (joblib .pkl / script)]
           │                                      │
           │                                      ├── 1. Feature Engineering (Rasio Gas & GGR)
           │                                      ├── 2. Evaluasi Skrining IEEE C57.104 & IEC 60422
           │                                      ├── 3. Diagnosis AI (Random Forest 7 Kelas)
           │                                      └── 4. Time-Series Trend & RUL Estimation
           ▼
[Tabel Diperkaya (Enriched Dataset)]

'''text 
