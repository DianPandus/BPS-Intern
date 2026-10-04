# Forecasting Harga Komoditas Pangan & Inflasi — Magang BPS

Project magang di **Badan Pusat Statistik (BPS)** untuk memprediksi perubahan harga bulanan 20 komoditas pangan dan inflasi (MtM, YtD, YoY) menggunakan **SARIMAX**. Variabel eksogennya dipilih berdasarkan keterkaitan antar komoditas, misalnya cabai merah ↔ cabai rawit ↔ bawang, atau beras ↔ mie instan ↔ tepung terigu.

> Project ini dikerjakan bersama [@arthuritokeintjem](https://github.com/arthuritokeintjem) selama magang. Repo aslinya ada di [arthuritokeintjem/BPS-Intern](https://github.com/arthuritokeintjem/BPS-Intern).

---

## Data

| File | Isi |
| --- | --- |
| `commodity_price_dataset.csv` | Perubahan harga (%) 20 komoditas pangan, Januari 2020 – Agustus 2025 |
| `inflation_dataset.csv` | Inflasi Month to Month, Year to Date, dan Year on Year, Januari 2020 – Juli 2025 (67 bulan) |
| `evaluation_results.csv` | Hasil backtest per komoditas (Agustus 2024 – Januari 2025): actual vs forecast, error, MAPE |
| `forecast_results.csv` | Forecast 6 bulan ke depan beserta confidence interval |
| `notebook.ipynb` | Seluruh pipeline, dari preprocessing sampai export hasil |

## Alur Kerja

1. **Preprocessing**: ubah data komoditas dari format wide ke long, parsing periode dengan nama bulan Indonesia, ffill bulan yang kosong, lalu log transform dan differencing.
2. **EDA**: ACF/PACF, dekomposisi musiman, dan boxplot per komoditas.
3. **Pemilihan eksogen**: mapping manual berdasarkan hubungan substitusi/komplementer antar bahan pangan, lalu distandardisasi (z-score).
4. **Model**: SARIMAX `(1,1,1)(1,1,1,12)`. Inflasi dimodelkan tanpa eksogen.
5. **Evaluasi**: holdout 6 bulan terakhir, dihitung MAE dan MAPE per komoditas.
6. **Forecast**: 6 bulan ke depan, lalu diekspor ke CSV.

## Insight

**1. Inflasi 2022 jadi outlier.**
Total inflasi bulanan sepanjang 2022 mencapai **±6,3%**, sedangkan tahun lain hanya di kisaran 1,4–2,5%. Puncak YoY terjadi di **September 2022 (7,08%)**. Lonjakan seperti ini membuat model musiman sulit belajar pola yang "normal".

**2. Cabai dan bawang merah adalah komoditas paling bergejolak.**
Dilihat dari standar deviasi perubahan harga bulanan:

| Komoditas | Std. dev (%) |
| --- | --- |
| Cabai Rawit | 39,5 |
| Cabai Merah | 23,9 |
| Bawang Merah | 16,0 |
| Susu Bubuk Balita | 0,6 |
| Susu Bubuk | 0,9 |

Produk olahan/kemasan jauh lebih stabil dibanding hortikultura segar.

**3. MAPE menyesatkan untuk data perubahan persen.**
Karena nilai aktualnya sering mendekati 0, MAPE bisa meledak. Contohnya Minyak Goreng dengan MAPE **1.943%**, dan Gula Pasir 60% padahal error absolutnya cuma 0,22 poin. Untuk data seperti ini, **MAE lebih jujur** dipakai:

| Paling akurat (MAE) | | Paling sulit (MAE) | |
| --- | --- | --- | --- |
| Tepung Terigu | 0,19 | Minyak Goreng | 6,20 |
| Gula Pasir | 0,22 | Udang Basah | 5,26 |
| Tahu | 0,26 | Jeruk | 2,47 |
| Daging Sapi | 0,34 | Ikan Kembung | 2,04 |

**4. Gambaran forecast (Sep 2025 – Feb 2026).**
* Inflasi MtM diprediksi rendah (0,03–0,16%) sampai Desember 2025, lalu **turun ke −0,70% di Januari 2026**, pola deflasi awal tahun yang juga terlihat di Januari–Februari 2025.
* Bawang Merah (+0,33%/bulan) dan Jeruk (+0,25%/bulan) punya rata-rata kenaikan forecast tertinggi.
* Confidence interval inflasi masih lebar (sekitar ±1,2 poin), jadi forecast ini lebih cocok dipakai sebagai indikasi arah daripada angka pasti.

## Keterbatasan & Pengembangan

* Order SARIMAX masih fixed untuk semua series. Grid search AIC (`USE_GRID_SEARCH = True`) kemungkinan bisa meningkatkan akurasi per komoditas.
* Data 2025 komoditas tercatat per minggu (M1–M5), sehingga perlu agregasi bulanan yang konsisten.
* Bisa dibandingkan dengan model lain, misalnya Prophet atau LSTM, terutama untuk komoditas volatil seperti cabai.

## Cara Menjalankan

```bash
git clone https://github.com/DianPandus/BPS-Intern.git
cd BPS-Intern
pip install pandas numpy matplotlib statsmodels scikit-learn jupyter
jupyter notebook notebook.ipynb
```

> Notebook membaca `file_harga.csv` dan `file_inflasi.csv`. Rename `commodity_price_dataset.csv` → `file_harga.csv` dan `inflation_dataset.csv` → `file_inflasi.csv`, atau ubah `HARGA_PATH` / `INFLASI_PATH` di Cell 3.

## Tools

Python · Pandas · NumPy · Statsmodels · Scikit-learn · Matplotlib · Jupyter
