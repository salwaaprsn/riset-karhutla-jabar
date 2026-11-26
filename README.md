# Data Pendukung

### Pengembangan Sistem Informasi Peringatan Dini Titik Panas Karhutla di Jawa Barat Berbasis Web dengan *Isolation Forest* untuk Deteksi Anomali *Spatio-Temporal*

Repository ini berisi dataset, data pendukung, hasil analisis, dan kode pemrograman yang digunakan dalam penelitian mengenai deteksi anomali titik panas (*hotspot*) kebakaran hutan dan lahan (Karhutla) di Provinsi Jawa Barat.

---

## 👩🏻‍💻 Identitas Peneliti

| Keterangan | Informasi |
|---|---|
| Nama | Salwa Aprilia Santi |
| NIM | 20230040141 |
| Program Studi | Teknik Informatika |
| Tahun Penelitian | 2026 |

---

## 📁 1. Struktur Repository

Repository ini disusun ke dalam beberapa direktori untuk memisahkan data berdasarkan tahapan dan fungsinya.

```text
riset-karhutla-jabar/
│
├── 1_data_mentah/
│   
├── 2_data_olahan/
│   
├── 3_data_pendukung/
│
├── 4_hasil_analisis/
│   
├── 5_kode/
  
```

**Keterangan direktori:**

| Direktori | Deskripsi |
|---|---|
| `1_data_mentah` | Data asli dari NASA FIRMS yang belum mengalami perubahan. |
| `2_data_olahan` | Data titik panas Jawa Barat yang telah melalui proses pemotongan wilayah, penyaringan, dan penghapusan duplikasi. |
| `3_data_pendukung` | Data batas wilayah Jawa Barat dari GADM dan dokumentasi resmi FIRMS. |
| `4_hasil_analisis` | Keluaran analisis yang digunakan sebagai bahan penyusunan Bab IV. |
| `5_kode` | Notebook Google Colab untuk menjalankan kembali proses pengolahan dan analisis data. |

---

## 🛰️ 2. Sumber Data

### 2.1 Data Titik Panas (*Hotspot*)

Data titik panas diperoleh dari **NASA Fire Information for Resource Management System (FIRMS)** menggunakan produk VIIRS 375 m, Archive, versi C2.

- **Sumber:** [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov)
- **Produk:** VIIRS 375 m, Archive, C2
- **Metode pengambilan:** Archive Download dengan pilihan Custom Region
- **Koordinat wilayah:** 106,3; -8,2; 108,9; -5,9
- **Rentang tanggal permintaan:** 1 Januari 2020 – 31 Oktober 2026
- **Format data:** CSV
- **Jumlah sensor:** 3 satelit

**Daftar berkas data:**

| Nama Berkas | Satelit | ID Permintaan | Tanggal Unduh |
|---|---|---:|---|
| `fire_archive_J1V-C2_836775.csv` | NOAA-20 | 836775 | 5 November 2026 |
| `fire_archive_SV-C2_836777.csv` | S-NPP | 836777 | 5 November 2026 |
| `fire_archive_J2V-C2_836776.csv` | NOAA-21 | 836776 | 3 November 2026 |

**Catatan penggunaan data:**

Data archive yang digunakan mencakup periode 2 Januari 2020 sampai 31 Juli 2026. Sementara itu, data *Near Real-Time* (NRT) NOAA-21 untuk periode Agustus sampai Oktober 2026 tidak dimasukkan dalam analisis utama karena belum memperoleh versi standar dari FIRMS.

### 2.2 Data Batas Wilayah

Data batas administrasi wilayah digunakan untuk membatasi analisis spasial agar hanya mencakup Provinsi Jawa Barat.

- **Sumber:** [GADM](https://gadm.org)
- **Versi:** 4.1
- **Level:** 1 (Provinsi)
- **Nama berkas:** `jabar_batas.zip`

Berkas ZIP berisi komponen *shapefile* berupa `.shp`, `.shx`, `.dbf`, dan `.prj` dari `gadm41_IDN_1`. Dalam penelitian ini, hanya poligon wilayah Jawa Barat yang digunakan.

### 2.3 Dokumentasi Pendukung

Dokumentasi pendukung berupa `Readme.txt`, yaitu dokumentasi resmi FIRMS yang diunduh bersama dataset untuk memberikan informasi mengenai struktur dan atribut data.

---

## ⚙️ 3. Tahapan Pengolahan Data

Pengolahan data dilakukan secara bertahap untuk memperoleh data titik panas yang sesuai dengan kebutuhan penelitian.

### 3.1 Jumlah Data pada Setiap Tahapan

| Tahapan Pengolahan | Jumlah Titik |
|---|---:|
| Data mentah dari tiga satelit | 32.986 |
| Setelah pemotongan batas Jawa Barat | 25.093 |
| Setelah filter `type = 0` dan `confidence = n atau h` | 17.469 |
| Setelah penghapusan duplikasi antar satelit | **13.687** |

### 3.2 Penjelasan Tahapan

1. **Pemotongan batas wilayah (*spatial filtering*)**

   Data titik panas dipotong berdasarkan batas administrasi Provinsi Jawa Barat. Proses ini dilakukan sebelum perhitungan fitur agar analisis hanya mencakup titik yang berada di dalam wilayah penelitian.

2. **Penyaringan jenis deteksi (*type filtering*)**

   Data difilter menggunakan nilai `type = 0` yang menunjukkan kebakaran vegetasi. Kode lainnya tidak disertakan, yaitu `type = 1` (gunung api aktif), `type = 2` (sumber panas darat statis seperti industri), dan `type = 3` (lepas pantai).

3. **Penyaringan tingkat kepercayaan (*confidence filtering*)**

   Penelitian menggunakan tingkat kepercayaan `n` (nominal) dan `h` (tinggi). Sementara itu, tingkat kepercayaan `l` (rendah) dikeluarkan untuk mengurangi kemungkinan positif palsu.

4. **Penghapusan duplikasi (*deduplication*)**

   Penghapusan duplikasi antar satelit dilakukan dengan membulatkan koordinat menjadi dua angka desimal dan menyamakan tanggal deteksi. Dari setiap kelompok duplikasi, titik dengan nilai FRP tertinggi dipertahankan.

Hasil akhir dari seluruh tahapan pengolahan adalah **13.687 titik panas** yang digunakan sebagai data analisis penelitian.

---

## 🧠 4. Ekstraksi Fitur dan Model

### 4.1 Fitur Penelitian

Penelitian menggunakan tujuh fitur sebagai masukan model *Isolation Forest*.

| No. | Fitur | Keterangan |
|---:|---|---|
| 1 | `latitude` | Koordinat lintang titik panas. |
| 2 | `longitude` | Koordinat bujur titik panas. |
| 3 | `frp` | Fire Radiative Power atau daya radiasi kebakaran. |
| 4 | `brightness` | Nilai kecerahan yang terdeteksi sensor. |
| 5 | `bright_t31` | Nilai kecerahan kanal termal 31. |
| 6 | `count_7d` | Jumlah titik panas dalam satu sel grid berukuran 0,1 derajat selama tujuh hari terakhir, termasuk hari pengamatan. |
| 7 | `daynight_d` | Indikator waktu deteksi, dengan nilai 1 untuk siang dan 0 untuk malam. |

### 4.2 Metode Utama: Isolation Forest

Metode utama yang digunakan adalah **Isolation Forest**, yaitu algoritma *unsupervised learning* untuk mendeteksi data yang memiliki karakteristik berbeda dari sebagian besar data lainnya.

Implementasi menggunakan pustaka `scikit-learn` dengan konfigurasi berikut:

| Parameter | Nilai |
|---|---:|
| `n_estimators` | 200 |
| `contamination` | 0,05 |
| `random_state` | 42 |

### 4.3 Metode Pembanding

Untuk mengevaluasi hasil deteksi anomali, penelitian menggunakan tiga metode pembanding:

1. **Persentil 95 pada `count_7d`**

   Menandai 5% nilai tertinggi berdasarkan jumlah titik panas dalam tujuh hari terakhir.

2. **Z-score per sel**

   Mengidentifikasi nilai yang memiliki Z-score lebih dari 2 berdasarkan riwayat titik panas pada sel grid yang sama.

3. **Local Outlier Factor (LOF)**

   Menggunakan algoritma deteksi anomali berbasis kepadatan lokal dengan konfigurasi `n_neighbors = 35` dan `contamination = 0,05`.

---

## 📊 5. Hasil Analisis

### 5.1 Perbandingan Hasil Deteksi Anomali

Hasil deteksi anomali dari setiap metode dibandingkan untuk mengetahui jumlah anomali, proporsi pada periode Agustus–Oktober, serta tingkat kesamaan hasil terhadap *Isolation Forest*.

| Metode | Jumlah Anomali | Agustus–Oktober | Kesamaan dengan IF |
|---|---:|---:|---:|
| Isolation Forest | 685 | 73,9% | 100,0% |
| Persentil 95 `count_7d` | 586 | 93,0% | 21,2% |
| Z-score per sel > 2 | 650 | 83,1% | 10,0% |
| Local Outlier Factor | 685 | 70,8% | 21,7% |

**Catatan:** Proporsi titik panas pada periode Agustus sampai Oktober terhadap keseluruhan data adalah 63,5%. Nilai tersebut digunakan sebagai basis pembanding untuk mengevaluasi distribusi temporal hasil deteksi anomali.

### 5.2 Hasil Utama

Berdasarkan hasil pengujian, metode *Isolation Forest* menghasilkan 685 titik anomali dari keseluruhan data yang dianalisis. Hasil tersebut kemudian dibandingkan dengan tiga metode pembanding untuk melihat perbedaan pola deteksi anomali berdasarkan karakteristik masing-masing metode.

---

## 🚀 6. Cara Menjalankan Kembali Penelitian

Seluruh proses pengolahan dan analisis data dapat dijalankan kembali menggunakan notebook Google Colab yang tersedia pada direktori kode.

**Langkah-langkah menjalankan penelitian:**

1. Buka notebook `karhutla_jabar_isolation_forest.ipynb` yang terdapat pada direktori `5_kode`.
2. Buka notebook menggunakan Google Colab.
3. Unggah empat berkas yang diperlukan melalui panel *Files*, yaitu:
   - Tiga berkas data archive dari direktori `1_data_mentah`.
   - Berkas `jabar_batas.zip` dari direktori `3_data_pendukung`.
4. Pilih menu **Runtime**, kemudian klik **Run all**.
5. Tunggu hingga seluruh tahapan pengolahan dan analisis selesai dijalankan.
6. Bandingkan jumlah titik hasil pengolahan dan tabel hasil analisis dengan informasi pada Bagian 3 dan Bagian 5.

Apabila hasil yang diperoleh berbeda, periksa kembali kesesuaian nama berkas, kelengkapan data, dan urutan proses pengolahan.

---

## ⚠️ 7. Keterbatasan Penelitian

Penelitian ini memiliki beberapa keterbatasan yang perlu diperhatikan dalam menginterpretasikan hasil analisis:

1. Data titik panas FIRMS merupakan hasil deteksi suhu permukaan oleh sensor satelit dan tidak secara langsung menjadi konfirmasi terjadinya kebakaran.
2. Validasi terhadap data luas kebakaran resmi belum dilakukan. Validasi yang tersedia masih berupa pemeriksaan konsistensi pola musiman dan pemeriksaan manual.
3. Parameter `contamination`, ukuran sel grid, dan jendela waktu tujuh hari belum melalui pengujian sensitivitas secara menyeluruh.
4. Data NRT NOAA-21 periode Agustus sampai Oktober 2026 belum dimasukkan dalam analisis utama.
5. Penghapusan duplikasi dilakukan menggunakan pembulatan koordinat, bukan berdasarkan perhitungan jarak geografis.

Keterbatasan tersebut menjadi pertimbangan dalam memahami hasil deteksi anomali dan pengembangan penelitian selanjutnya.

---

<p align="center">
  <sub>📚 Repository ini disusun sebagai dokumentasi data dan kode pendukung penelitian Riset tahun 2026.</sub>
</p>
