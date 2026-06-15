# Optimasi Bobot Skor Prioritas Pengiriman Amazon Menggunakan Genetic Algorithm Berdasarkan Data Historis Last-Mile Delivery

Dokumen ini menjelaskan output bagian **Dataset dan Formulasi Masalah** untuk proyek akhir Algoritma Evolusi dan Kecerdasan Berkoloni.

Dataset yang digunakan:

<https://www.kaggle.com/datasets/sujalsuthar/amazon-delivery-dataset>

## Tujuan

Tujuan bagian ini adalah menyiapkan dataset Amazon Delivery agar siap digunakan untuk implementasi algoritma metaheuristik, khususnya **Genetic Algorithm (GA)**.

Formulasi akhir yang digunakan adalah:

> Mencari kombinasi bobot terbaik untuk membentuk skor prioritas pengiriman, sehingga skor tersebut paling sesuai dengan pola `delivery_time` historis.

Formulasi ini dipilih karena dataset bersifat **per order**, bukan data rute kendaraan lengkap. Jadi proyek tidak dipaksakan menjadi Vehicle Routing Problem penuh.

## Struktur Output

```text
.
|-- README.md
|-- requirements.txt
|-- 02_dataset_formulation_preprocessing.ipynb
|-- data
|   |-- raw
|   |   |-- amazon_delivery.csv
|   |-- processed
|       |-- amazon_delivery_cleaned.csv
|       |-- data_quality_summary.csv
|       |-- priority_risk_delivery_time_metrics.csv
|       |-- priority_risk_delivery_time_quintile_summary.csv
```

## Daftar Output

| Output | Lokasi | Fungsi |
|---|---|---|
| Notebook preprocessing | `02_dataset_formulation_preprocessing.ipynb` | Proses download, cek data, cleaning, feature engineering, penyimpanan data, dan formulasi masalah |
| Requirements | `requirements.txt` | Daftar package Python yang dibutuhkan notebook |
| Dataset mentah | `data/raw/amazon_delivery.csv` | Dataset asli dari Kaggle tanpa modifikasi |
| Dataset bersih | `data/processed/amazon_delivery_cleaned.csv` | Dataset siap pakai untuk implementasi GA |
| Ringkasan kualitas data | `data/processed/data_quality_summary.csv` | Jumlah baris/kolom sebelum dan sesudah preprocessing |
| Metrik evaluasi baseline | `data/processed/priority_risk_delivery_time_metrics.csv` | Korelasi baseline equal-weight terhadap `delivery_time` |
| Ringkasan quintile baseline | `data/processed/priority_risk_delivery_time_quintile_summary.csv` | Rata-rata `delivery_time` per kelompok skor risiko |
| Dokumentasi | `README.md` | Penjelasan output, preprocessing, formulasi, dan hasil evaluasi awal |

## Dataset

Dataset ini berisi data historis pengiriman Amazon. Satu baris data merepresentasikan satu order/pengiriman.

Kolom awal dataset:

| Kolom | Keterangan |
|---|---|
| `Order_ID` | ID pesanan |
| `Agent_Age` | Usia kurir |
| `Agent_Rating` | Rating kurir |
| `Store_Latitude` | Latitude toko/gudang |
| `Store_Longitude` | Longitude toko/gudang |
| `Drop_Latitude` | Latitude tujuan pengiriman |
| `Drop_Longitude` | Longitude tujuan pengiriman |
| `Order_Date` | Tanggal pemesanan |
| `Order_Time` | Waktu pemesanan |
| `Pickup_Time` | Waktu pickup |
| `Weather` | Kondisi cuaca |
| `Traffic` | Kondisi lalu lintas |
| `Vehicle` | Jenis kendaraan |
| `Area` | Area pengiriman |
| `Delivery_Time` | Waktu pengiriman historis |
| `Category` | Kategori barang |

## Kualitas Data Awal

Hasil pengecekan dataset mentah:

| Informasi | Nilai |
|---|---:|
| Jumlah baris awal | 43.739 |
| Jumlah kolom awal | 16 |
| Missing value pada `Weather` | 91 |
| Missing value pada `Agent_Rating` | 54 |
| Outlier jarak tidak realistis | 188 baris |

Outlier jarak ditemukan setelah menghitung jarak antara lokasi toko dan tujuan. Karena konteks proyek adalah last-mile delivery, data dengan jarak di atas 100 km dianggap tidak realistis dan dihapus.

## Keputusan Preprocessing

| Keputusan | Alasan |
|---|---|
| Standarisasi nama kolom | Agar konsisten dan mudah dipakai di Python |
| Menghapus duplikasi `order_id` jika ada | Satu order seharusnya hanya muncul satu kali |
| Mengubah kolom numerik ke tipe angka | Agar bisa dihitung untuk fitur optimasi |
| Memperbaiki `agent_rating` | Rating kosong/tidak valid diisi median |
| Memperbaiki `agent_age` | Umur tidak realistis diisi median |
| Mengisi kategori kosong dengan `Unknown` | Agar data tidak terlalu banyak terhapus |
| Menghitung `distance_km` | Jarak penting dalam risiko pengiriman |
| Menghapus jarak lebih dari 100 km | Tidak realistis untuk last-mile delivery |
| Membuat penalti traffic, cuaca, kendaraan, dan kurir | Mengubah faktor operasional menjadi nilai numerik |
| Membuat feature score 0 sampai 1 | Agar GA dapat mengoptimasi bobot secara adil |

## Dataset Bersih

File:

```text
data/processed/amazon_delivery_cleaned.csv
```

Ringkasan:

| Informasi | Nilai |
|---|---:|
| Baris data mentah | 43.739 |
| Kolom data mentah | 16 |
| Baris data bersih | 43.551 |
| Kolom data bersih | 28 |
| Baris dihapus | 188 |

## Kolom Penting Dataset Bersih

Dataset bersih memiliki 28 kolom. Kolom yang paling penting untuk formulasi GA adalah:

| Kolom | Keterangan |
|---|---|
| `order_id` | ID order |
| `delivery_time` | Waktu pengiriman historis, hanya untuk evaluasi |
| `distance_km` | Jarak toko ke tujuan |
| `traffic_penalty` | Penalti kondisi lalu lintas |
| `weather_penalty` | Penalti kondisi cuaca |
| `vehicle_penalty` | Penalti jenis kendaraan |
| `agent_penalty` | Penalti dari rating kurir |
| `distance_score` | Normalisasi `distance_km` ke 0-1 |
| `traffic_score` | Normalisasi `traffic_penalty` ke 0-1 |
| `weather_score` | Normalisasi `weather_penalty` ke 0-1 |
| `vehicle_score` | Normalisasi `vehicle_penalty` ke 0-1 |
| `agent_score` | Normalisasi `agent_penalty` ke 0-1 |
| `optimization_cost` | Baseline cost tanpa `delivery_time` |
| `priority_risk_score` | Baseline skor risiko dengan bobot sama rata |

## Fitur Baru

### `distance_km`

Jarak antara toko/gudang dan tujuan pengiriman dihitung dari koordinat latitude-longitude menggunakan rumus Haversine.

### Penalti Operasional

Beberapa kolom kategori diubah menjadi penalti numerik:

| Faktor | Kolom Penalti | Makna |
|---|---|---|
| Traffic | `traffic_penalty` | Traffic lebih padat mendapat penalti lebih besar |
| Cuaca | `weather_penalty` | Cuaca buruk mendapat penalti lebih besar |
| Kendaraan | `vehicle_penalty` | Jenis kendaraan diberi penalti sesuai asumsi operasional |
| Kurir | `agent_penalty` | Rating kurir lebih rendah mendapat penalti lebih besar |

Rumus `agent_penalty`:

```text
agent_penalty = 6 - agent_rating
```

### Feature Score

Setiap faktor utama dinormalisasi ke rentang 0 sampai 1:

```text
distance_score = normalize(distance_km)
traffic_score  = normalize(traffic_penalty)
weather_score  = normalize(weather_penalty)
vehicle_score  = normalize(vehicle_penalty)
agent_score    = normalize(agent_penalty)
```

Feature score ini menjadi input utama untuk Genetic Algorithm.

## Batasan Dataset

Dataset tidak menyediakan semua informasi operasional pengiriman. Batasan ini penting agar formulasi masalah tidak berlebihan.

| Batasan Dataset | Dampak pada Formulasi |
|---|---|
| Satu baris merepresentasikan satu order/pengiriman | Model tidak dianggap sebagai rute kendaraan penuh |
| Tidak ada kapasitas kendaraan | `vehicle` tidak dipakai sebagai kapasitas angkut |
| Tidak ada berat/volume paket | Tidak ada constraint kapasitas barang |
| Tidak ada jumlah paket dalam satu kendaraan | Tidak bisa menyimpulkan van membawa banyak paket dalam satu trip |
| Tidak ada daftar order dalam satu trip | Tidak membentuk multi-drop route aktual |
| Tidak ada jumlah kendaraan tersedia | Tidak melakukan assignment kendaraan |
| Tidak ada urutan drop-off aktual | Tidak memodelkan rute delivery nyata |

Karena itu, proyek ini **bukan Vehicle Routing Problem penuh**. Formulasi yang dipakai adalah **optimasi bobot skor prioritas order**.

## Formulasi Masalah

### Masalah Optimasi

Masalah yang diselesaikan:

> Menentukan bobot optimal dari faktor jarak, traffic, cuaca, kendaraan, dan rating kurir untuk membentuk skor prioritas order yang paling sesuai dengan pola keterlambatan historis.

Skor prioritas ini dapat digunakan untuk mengurutkan order dari risiko tertinggi ke risiko terendah.

### Variabel Keputusan

Kromosom GA berisi lima bobot:

```text
X = [w_distance, w_traffic, w_weather, w_vehicle, w_agent]
```

Contoh:

```text
X = [0.30, 0.25, 0.15, 0.10, 0.20]
```

Artinya:

| Bobot | Makna |
|---|---|
| `w_distance` | Pengaruh jarak |
| `w_traffic` | Pengaruh traffic |
| `w_weather` | Pengaruh cuaca |
| `w_vehicle` | Pengaruh kendaraan |
| `w_agent` | Pengaruh rating kurir |

### Fungsi Skor Prioritas

Untuk setiap order ke-i:

```text
priority_score_i =
    w_distance * distance_score_i
  + w_traffic  * traffic_score_i
  + w_weather  * weather_score_i
  + w_vehicle  * vehicle_score_i
  + w_agent    * agent_score_i
```

Semakin tinggi `priority_score`, semakin tinggi risiko/prioritas order tersebut.

### Fungsi Objektif

Tujuan GA adalah mencari bobot yang membuat `priority_score` paling sesuai dengan `delivery_time` historis.

```text
Maximize Spearman(priority_score, delivery_time)
```

Spearman correlation dipilih karena yang penting adalah kesesuaian peringkat. Jika order dengan `delivery_time` historis tinggi juga mendapat `priority_score` tinggi, maka bobot dianggap baik.

### Fitness Function

Karena Spearman correlation berada pada rentang -1 sampai 1, nilai fitness dapat dibuat menjadi rentang 0 sampai 1:

```text
Fitness = (Spearman(priority_score, delivery_time) + 1) / 2
```

Semakin besar fitness, semakin baik kombinasi bobot.

### Kendala

```text
0 <= setiap bobot <= 1
sum(bobot) = 1
```

Kendala tambahan:

1. `delivery_time` tidak boleh dipakai sebagai input `priority_score`.
2. `delivery_time` hanya dipakai sebagai evaluasi historis.
3. Setiap baris dataset dianggap sebagai satu order independen.
4. Jarak pengiriman harus realistis untuk last-mile delivery, yaitu tidak lebih dari 100 km.
5. Kolom `vehicle` hanya digunakan sebagai faktor risiko, bukan kapasitas kendaraan.
6. Model tidak memasukkan kapasitas kendaraan, berat/volume paket, jumlah kendaraan, atau multi-drop route.

## Kenapa Formulasi Ini Kuat

Formulasi ini lebih kuat dibandingkan membuat skor manual karena:

1. GA memiliki variabel keputusan yang jelas, yaitu bobot.
2. Fungsi objektif dapat dihitung secara kuantitatif.
3. `delivery_time` tidak dipakai sebagai input, sehingga menghindari data leakage.
4. `delivery_time` tetap dimanfaatkan sebagai ground truth historis untuk evaluasi.
5. Formulasi sesuai dengan struktur dataset yang berbasis per order.
6. Hasil akhir mudah dianalisis melalui bobot terbaik, korelasi, dan perbandingan baseline.

## Evaluasi Baseline Equal-Weight

Sebelum GA dijalankan, notebook membuat baseline `priority_risk_score` dengan bobot sama rata:

```text
priority_risk_score =
    mean(distance_score, traffic_score, weather_score, vehicle_score, agent_score)
```

Baseline ini bukan hasil final GA, tetapi berguna sebagai pembanding awal.

Hasil baseline terhadap `delivery_time` historis:

| Metrik | Nilai |
|---|---:|
| Pearson correlation | 0.4109 |
| Spearman correlation | 0.4295 |
| Rata-rata `delivery_time` top 20% risiko tertinggi | 157.48 |
| Rata-rata `delivery_time` bottom 20% risiko terendah | 93.95 |
| Selisih top 20% dan bottom 20% | 63.53 |

Ringkasan per kelompok risiko:

| Kelompok | Avg Delivery Time |
|---|---:|
| Q1 sangat rendah | 93.96 |
| Q2 rendah | 112.93 |
| Q3 sedang | 123.87 |
| Q4 tinggi | 136.44 |
| Q5 sangat tinggi | 157.48 |

Interpretasi:

Baseline equal-weight sudah menunjukkan hubungan positif sedang dengan `delivery_time`. Artinya, fitur risiko yang dibuat cukup relevan. Target GA adalah mencari bobot yang lebih baik daripada baseline tersebut.

## Luaran yang Relevan dengan Rubrik

| Rubrik | Output yang Disediakan |
|---|---|
| Problem Formulation | Masalah optimasi, variabel keputusan, fungsi objektif, fitness function, dan kendala |
| Algorithm Design | Struktur kromosom bobot dan feature score yang siap dipakai |
| Implementation | Dataset bersih dalam CSV, kolom score numerik, dan target evaluasi |
| Experimentation & Tuning | Baseline tersedia untuk dibandingkan dengan hasil GA |
| Analysis & Reporting | README, metrik baseline, ringkasan quintile, dan justifikasi data leakage |

## Cara Menjalankan Notebook

Jalankan:

```text
02_dataset_formulation_preprocessing.ipynb
```

Library utama:

```text
pandas
numpy
kagglehub
```

Package tersebut sudah ditulis di:

```text
requirements.txt
```

Notebook akan membaca `requirements.txt`, mengecek package yang belum tersedia, lalu hanya menginstall package yang belum ada. Jika package sudah terinstall, notebook tidak akan menginstall ulang.

Alur notebook:

1. Download dataset Kaggle.
2. Simpan data mentah ke `data/raw`.
3. Cek struktur dan kualitas data.
4. Bersihkan data.
5. Hitung jarak dan penalti.
6. Buat feature score 0 sampai 1.
7. Simpan data processed.
8. Simpan evaluasi baseline historis.
9. Tampilkan formulasi masalah.

Jika download Kaggle gagal, jalankan notebook di Kaggle Notebook atau pastikan akses Kaggle tersedia di komputer lokal.

## Kesimpulan

Output bagian ini sudah menyiapkan dasar yang kuat untuk proyek metaheuristik. Dataset sudah dibersihkan, fitur risiko sudah dibuat, batasan dataset sudah dijelaskan, dan formulasi masalah sudah diarahkan ke optimasi bobot menggunakan Genetic Algorithm.

Output utama:

1. `02_dataset_formulation_preprocessing.ipynb`
2. `requirements.txt`
3. `data/raw/amazon_delivery.csv`
4. `data/processed/amazon_delivery_cleaned.csv`
5. `data/processed/data_quality_summary.csv`
6. `data/processed/priority_risk_delivery_time_metrics.csv`
7. `data/processed/priority_risk_delivery_time_quintile_summary.csv`
8. `README.md`
