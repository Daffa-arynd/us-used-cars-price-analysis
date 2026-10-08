# US Used Cars: Faktor yang Menggerakkan Resale Value
> Faktor apa yang paling menurunkan atau menjaga harga jual mobil bekas di AS, dan seberapa akurat harga bisa diprediksi?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/us-used-cars-price-analysis/blob/main/us-used-cars-analysis.ipynb)

## Business Problem
Dealer mobil bekas perlu tahu stok mana yang paling menjaga nilai, pembeli perlu tahu kapan waktu terbaik membeli, dan keduanya butuh patokan harga yang objektif. Proyek ini menjawab ketiganya dari data listing mobil bekas di AS.

## Dataset
- **Sumber:** [US Used Cars Dataset (Kaggle)](https://www.kaggle.com/datasets/ananaymital/us-used-cars-dataset)
- **Ukuran asli:** 3,000,040 baris, 66 kolom (28 kolom dibaca, 21 fitur dipakai di model)
- **Mobil bekas:** 1,529,003 baris
- **Sampel acak 10%:** 152,900 baris, **setelah cleaning: 149,715 baris**
- **Sampling:** file asli ±9 GB melebihi RAM Colab gratis. Data dibaca per chunk (500 ribu baris), difilter hanya mobil bekas, lalu disampel acak 10% per chunk (`random_state=42`) supaya representatif dan reproducible.
- **Data dibuang (total 3,185 baris, 2.1% dari sampel):**
  - 0 duplikat VIN
  - 1,670 baris tanpa price/mileage/year
  - 1,515 baris outlier (harga di luar $1,000 sampai persentil 99.5; mileage di luar 100 sampai 300,000; tahun produksi < 1995)
  - Nilai tidak masuk akal pada horsepower (di luar 50-1000) dan fuel economy (≤ 0) diubah jadi kosong lalu diisi median di pipeline model, bukan dibuang barisnya.
- **Asumsi:** umur mobil = tahun listing − tahun produksi, karena data adalah snapshot sekitar 2020.

## Tools
Python, pandas, NumPy, scikit-learn, XGBoost, matplotlib, seaborn (Google Colab)

## Pendekatan
1. **Cleaning:** deduplikasi VIN, penanganan missing value, trimming outlier harga dan mileage, konversi tipe data.
2. **Analisis/modeling:** EDA, korelasi Spearman, Linear Regression (baseline) vs XGBoost pada skala log harga, evaluasi MAE dan R² di skala dolar, permutation importance.
3. **Visualisasi:** kurva depresiasi, harga per brand dan body type, heatmap korelasi, feature importance.

## Temuan Utama
- **Depresiasi:** harga turun paling tajam di tahun pertama (**-25.9%**, dari $32,385 ke $23,999) dan di tahun ke-4 (**-18.2%**). Setelah umur 8, mobil kehilangan sekitar $1,000 atau kurang per tahun.
- **Retensi nilai** (median harga umur 5-7 tahun vs umur 0-2 tahun): Toyota dan Hyundai menyimpan **64.4%**, sedangkan Chrysler hanya **44.2%** dan BMW **44.8%**. Pickup truck menyimpan 65.7% dan sedan 63.1%, sedangkan SUV/crossover 55.9% dan wagon 48.8%.
- **Faktor harga** (regresi log, faktor lain ditahan konstan): tiap +1 tahun umur ≈ **-6.3%** harga, tiap +10,000 mil ≈ **-4.0%**, riwayat kecelakaan ≈ **-9.0%** (setara kehilangan ±1.4 tahun umur).
- **Model:** XGBoost MAE **$1,901** (9.8% dari median harga $19,325) dan R² **0.941**, dibanding Linear Regression MAE $3,426 dan R² 0.804. Error turun ±44.5%.
- **Fitur terpenting** (permutation importance): age (0.398), horsepower (0.375), mileage (0.220), brand (0.067), wheel system (0.049).

![Kurva depresiasi](figures/figuresdepreciation_curve.png)
![Harga per brand](figures/figuresprice_by_brand.png)
![Feature importance](figures/figuresfeature_importance.png)

## Rekomendasi Keputusan
- **Dealer/penjual:** prioritaskan stok Toyota, Hyundai, GMC, dan pickup truck (retensi 64-66%). Ambil BMW, Audi, atau Chrysler (retensi 44-47%) hanya kalau harga belinya cukup murah untuk menutup penurunan nilai yang lebih cepat. Dalam dolar, BMW kehilangan sekitar $22,095 dari umur 0-2 ke umur 5-7, sedangkan Toyota sekitar $8,186.
- **Pembeli:** hindari umur 0-1 (penurunan 25.9%). Beli di **umur 4-5** untuk mobil relatif baru (median $16,000-18,000), atau di **umur 8-11** untuk yang hemat (median $8,000-11,000, kehilangan nilai sekitar $1,000 atau kurang per tahun).
- **Penetapan harga:** pakai XGBoost sebagai harga referensi awal (±$1,901). Listing yang menyimpang lebih dari itu perlu dicek ulang.
- **Faktor kunci:** umur adalah penurun nilai utama dan tidak bisa dikontrol. Yang bisa dijaga penjual adalah mileage (tiap 10,000 mil = -4.0%) dan riwayat kecelakaan (-9.0%, jadi tawar minimal ±9% lebih murah saat membeli mobil dengan riwayat kecelakaan).

## Screenshot / Demo
Lihat folder [`figures/`](figures/) dan notebook `us-used-cars-analysis.ipynb` (klik badge Colab di atas).

## Cara Menjalankan
1. Buka notebook lewat badge Colab.
2. Buat API token di Kaggle, simpan di Colab Secrets dengan nama `KAGGLE_API_TOKEN` (aktifkan Notebook access). Pastikan sudah menyetujui syarat dataset di halaman Kaggle.
3. Mount Google Drive saat diminta, lalu jalankan cell berurutan dari atas (**Runtime → Run all**). Download dan sampling memakan waktu beberapa menit.
4. Dependensi: lihat `requirements.txt`.

## Keterbatasan & Next Steps
- Hanya sampel 10% (±150 ribu baris) dari data mobil bekas; hasil bisa sedikit bergeser dengan data penuh.
- Data snapshot sekitar 2020, jadi harga dan tren bukan kondisi pasar sekarang.
- Tidak ada fitur kondisi fisik, trim, dan lokasi, yang kemungkinan menjelaskan sebagian error model. Horsepower kemungkinan menjadi penanda kelas/segmen mobil, bukan penyebab harga naik.
- Kurva depresiasi dan retensi membandingkan mobil berbeda di tiap umur (bukan satu mobil yang dipantau dari waktu ke waktu), sehingga komposisi model ikut memengaruhi hasil.
- Koefisien regresi bersifat asosiasi, bukan kausal, dan hanya mengontrol tiga variabel.
- **Next steps:** dashboard Streamlit, hyperparameter tuning, analisis per model mobil, validasi di data periode lain.
