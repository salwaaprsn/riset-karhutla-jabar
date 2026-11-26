# Data dan Kode Pendukung Riset

### Pengembangan Sistem Informasi Pemantauan Titik Panas Karhutla di Jawa Barat Berbasis Web dengan *Isolation Forest* untuk Deteksi Anomali *Spatio-Temporal*

Repository ini berisi data penelitian, hasil analisis, serta kode program yang digunakan dalam penelitian deteksi anomali titik panas (*hotspot*) kebakaran hutan dan lahan (karhutla) di wilayah Jawa Barat periode 2020–2025.

---

## 📁 1. Struktur Repository

```text
├── 1_data_mentah/       # Data asli NASA FIRMS (tidak diubah)
├── 2_data_olahan/       # Data titik panas Jawa Barat hasil prapemrosesan
├── 3_data_pendukung/    # Batas wilayah GADM dan README FIRMS
├── 4_hasil_analisis/    # Hasil deteksi, perbandingan metode, dan validasi
└── 5_kode/              # Notebook Google Colab
```

---

## 🛰️ 2. Sumber Data

| Data | Sumber | Keterangan |
|---|---|---|
| Titik panas | NASA FIRMS, VIIRS 375 m, Archive C2 | Satelit S-NPP, NOAA-20, dan NOAA-21. Periode 1 Januari 2020–31 Desember 2025. |
| Batas wilayah | GADM 4.1, Level 1 | Poligon wilayah Jawa Barat (`jabar_batas.zip`). |
| Luas karhutla | SiPongi+ Kementerian Kehutanan | Data tahunan Jawa Barat periode 2020–2025 untuk keperluan validasi. |
| Jumlah penduduk | BPS, *Jawa Barat dalam Angka 2026*, Tabel 3.1.3 | Digunakan sebagai data pendukung latar belakang penelitian. |

**Catatan:** Berkas *archive* yang diunduh mencakup data hingga Juli 2026. Namun, penelitian ini hanya menggunakan data sampai dengan 31 Desember 2025. Data *Near Real-Time* (NRT) tidak digunakan.

---

## ⚙️ 3. Tahapan Prapemrosesan Data

Prapemrosesan dilakukan secara bertahap untuk memperoleh data titik panas yang sesuai dengan cakupan wilayah dan kebutuhan penelitian.

| Tahapan | Jumlah Titik |
|---|---:|
| Data mentah | 32.986 |
| Penyaringan periode hingga 31 Desember 2025 | 29.881 |
| Pemotongan berdasarkan wilayah Jawa Barat | 22.898 |
| Penyaringan (`type = 0`, `confidence = n/h`) | 15.877 |
| Penghapusan data duplikat | **12.435** |

Total data akhir yang digunakan dalam proses analisis adalah **12.435 titik panas**.

---

## 🧠 4. Metode Penelitian

### 4.1 Fitur yang Digunakan

Fitur yang digunakan dalam proses deteksi anomali meliputi:

- `latitude`: Koordinat lintang titik panas.
- `longitude`: Koordinat bujur titik panas.
- `FRP`: Nilai *Fire Radiative Power*.
- `brightness`: Nilai kecerahan titik panas.
- `bright_t31`: Nilai kecerahan kanal termal T31.
- `count_7d`: Jumlah titik panas dalam sel grid 0,1° selama 7 hari.
- `daynight_d`: Indikator waktu deteksi pada siang atau malam hari.

### 4.2 Metode Deteksi Anomali

**Metode utama: Isolation Forest**

Parameter yang digunakan:

| Parameter | Nilai |
|---|---:|
| `n_estimators` | 200 |
| `contamination` | 0,05 |
| `random_state` | 42 |

**Metode pembanding:**

1. Persentil 95 berdasarkan fitur `count_7d`.
2. Z-score per sel dengan ambang batas `z > 2`.
3. Local Outlier Factor (LOF) dengan `n_neighbors = 35`.

---

## 📊 5. Hasil Penelitian

### 5.1 Hasil Deteksi Anomali

Berdasarkan 12.435 titik panas yang dianalisis, diperoleh hasil perbandingan deteksi anomali sebagai berikut.

| Metode | Jumlah Anomali | Agustus–Oktober (%) | Kesamaan dengan IF (%) |
|---|---:|---:|---:|
| Isolation Forest | 622 | 76,8 | 100,0 |
| Persentil 95 | 574 | 95,5 | 24,6 |
| Z-score per sel | 587 | 88,1 | 11,1 |
| LOF | 622 | 77,2 | 21,5 |

Isolation Forest menghasilkan **622 titik anomali** dari keseluruhan data yang dianalisis.

### 5.2 Analisis Sensitivitas Parameter

Pengujian sensitivitas dilakukan terhadap parameter `contamination` pada Isolation Forest untuk mengetahui pengaruh perubahan nilai parameter terhadap jumlah anomali yang terdeteksi.

| Contamination | Jumlah Anomali |
|---|---:|
| 0,01 | 125 |
| 0,05 | 622 |
| 0,10 | 1.244 |

### 5.3 Validasi Tahunan (2020–2025)

Validasi dilakukan dengan membandingkan hasil deteksi anomali terhadap data luas karhutla tahunan dari SiPongi+.

| Indikator | Spearman (ρ) | P-value | Hasil |
|---|---:|---:|---|
| Jumlah anomali dengan luas karhutla | 0,70 | 0,12 | Tidak signifikan |
| Proporsi anomali dengan luas karhutla | -0,03 | 0,96 | Tidak menunjukkan hubungan yang signifikan |

Hasil validasi menunjukkan adanya korelasi positif pada jumlah anomali, tetapi belum signifikan secara statistik. Sementara itu, proporsi anomali tidak menunjukkan hubungan yang berarti dengan luas karhutla.

**Catatan:** Hasil tersebut menunjukkan konsistensi awal berdasarkan data yang tersedia, bukan merupakan bukti akurasi deteksi anomali.

---

## 📦 6. Berkas Hasil Penelitian

Beberapa berkas utama yang dihasilkan dari proses analisis meliputi:

| Nama Berkas | Deskripsi |
|---|---|
| `titik_panas_jabar_2020_2025.json` | Data 12.435 titik panas beserta label dan skor anomali untuk kebutuhan visualisasi peta web. |
| `anomali_jabar_2020_2025.csv` | Data titik panas dalam format CSV beserta hasil deteksi anomali. |
| `perbandingan_metode_2020_2025.csv` | Hasil perbandingan metode deteksi anomali. |
| `validasi_tahunan_2020_2025.csv` | Hasil validasi tahunan periode 2020–2025. |

---

## 🚀 7. Panduan Menjalankan Ulang Penelitian

Untuk menjalankan kembali proses analisis, ikuti langkah-langkah berikut:

1. Buka notebook `5_kode/karhutla_jabar_isolation_forest.ipynb` menggunakan Google Colab.
2. Unggah tiga berkas *archive* dari folder `1_data_mentah/` serta berkas `jabar_batas.zip`.
3. Pastikan seluruh berkas pendukung telah tersedia.
4. Pilih menu **Runtime → Run all** untuk menjalankan seluruh tahapan analisis.

---

## ⚠️ 8. Keterbatasan Penelitian

Penelitian ini memiliki beberapa keterbatasan yang perlu diperhatikan dalam menginterpretasikan hasil analisis:

1. Data titik panas NASA FIRMS merupakan hasil deteksi anomali suhu permukaan, bukan konfirmasi langsung terjadinya kebakaran.
2. Validasi hanya menggunakan enam data tahunan. Data bulanan dan hasil verifikasi lapangan belum tersedia.
3. Data luas karhutla dari SiPongi+ tidak sepenuhnya independen dari data titik panas yang digunakan dalam penelitian.
4. Tidak tersedia label kejadian kebakaran berdasarkan verifikasi lapangan sehingga metrik akurasi klasifikasi belum dapat dihitung.
5. Pengujian parameter sel grid, jendela waktu, dan `contamination` masih terbatas.
6. Penghapusan data duplikat menggunakan pembulatan koordinat, bukan perhitungan jarak geografis.

---

## 📩 9. Kontak

Peneliti: Salwa Aprilia Santi

Email: salwa.aprilia_ti23@nusaputra.ac.id
