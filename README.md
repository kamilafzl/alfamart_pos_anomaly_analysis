# Capstone Project Modul Data Analysis
## Deteksi Anomali Transaksi POS & Perlindungan Margin Promo JSM di Jaringan Minimarket "Alfamart"

**Capstone Project Module 2 (Data Analysis) | Purwadhika Digital Technology School**
**Student:** Kamila Fazilatunisa

---

## 📌 Business Problem

> Bagaimana Alfamart dapat mengurangi kebocoran margin akibat penyalahgunaan Promo JSM (Jumat-Sabtu-Minggu) oleh tengkulak, sekaligus merapikan kualitas data POS (payment method, member ID, dan anomali sistem) agar pelaporan finance & operasional lebih akurat?

Pertanyaan ini dipecah menjadi 6 business questions:

1. Gerai mana yang punya volume transaksi JSM tinggi namun terindikasi dikuasai segelintir member (indikasi tengkulak)?
2. Kategori barang apa yang paling sering mengalami void/pembatalan transaksi?
3. Seberapa parah inkonsistensi penulisan `payment_method` di sistem?
4. Berapa persen transaksi tanpa ID Member, dan apa dampaknya ke visibilitas Customer Lifetime Value?
5. Apakah ada transaksi fiktif yang tercatat sebelum toko resmi buka (*logical error*)?
6. Berapa estimasi kerugian akibat outlier quantity (qty ekstrem/negatif) hasil salah scan kasir?

## 🗂️ Dataset

Data sintetis (± 300.000 baris log kasir), terdiri dari 3 tabel:

| File | Isi | Baris |
|---|---|---|
| `data/raw/alfa_stores.csv` | Dimensi gerai (kode, nama, kota, tanggal grand opening) | 1.000 |
| `data/raw/alfa_products.csv` | Master SKU (kode, nama produk, kategori, harga normal) | 500 |
| `data/raw/alfa_pos_transactions.csv` | Log transaksi kasir mentah | 300.000 |
| `data/clean/alfa_pos_transactions_clean.csv` | Dataset hasil cleaning, siap analisis & siap ditarik ke Tableau | 289.516 |

## 🧹 Ringkasan Data Cleaning

| Anomali | Skala | Keputusan |
|---|---|---|
| `payment_method` tidak konsisten (10 varian teks) | 300.000 baris | Distandarisasi → 6 kategori baku |
| `member_id` kosong | 40% (120.000 baris) | Dipertahankan, ditandai `is_member` (bukan error — transaksi non-member valid) |
| `qty` negatif (void/retur) | 0,49% (1.483 baris) | Ditandai `is_void`, dikeluarkan dari perhitungan revenue |
| `trx_time` sebelum `open_date` toko | 3,0% (9.000 baris) | Dihapus — data fiktif / bug sistem |
| `qty` = 999 atau 9999 (sentinel error) | 0,51% (1.517 baris) | Dihapus — bukan quantity asli |

Hasil akhir: **289.516 baris (96,5%)** dataset bersih siap pakai.

## 🔑 Key Findings

- **81 gerai** terindikasi punya pola volume JSM tinggi + konsentrasi qty-per-member di atas rata-rata — kandidat prioritas monitoring, meski tidak ada bukti 1 member tunggal mendominasi.
- **Food & Staples** paling sering di-void (±40% dari total void transaksi).
- **10 varian teks** `payment_method` ternyata cuma **6 metode bayar** yang sama, murni masalah standarisasi input.
- **40% transaksi** tidak menyertakan member ID — titik buta terbesar untuk visibilitas Customer Lifetime Value.
- **9.000 transaksi (±Rp4,67 M)** tercatat fiktif sebelum toko buka — indikasi bug sistem/ETL.
- **1.517 baris qty sentinel** mendistorsi laporan hingga ±Rp199,4 M jika tidak dibersihkan.

Rekomendasi lengkap & rencana aksi ada di notebook dan slide presentasi (lihat bagian *Deliverables*).

## 🛠️ Tools & Stack

- **Python** (pandas, numpy, matplotlib, seaborn) — data cleaning & exploratory/diagnostic analysis
- **Jupyter Notebook** — dokumentasi analisis end-to-end
- **Tableau** — dashboard interaktif untuk eksplorasi lebih lanjut oleh stakeholder
- **PowerPoint** — laporan hasil analisis untuk presentasi ke stakeholder

## 📁 Struktur Repository

```
├── README.md
├── notebook/
│   └── Capstone2_Alfamart_Analysis.ipynb      # analisis end-to-end (cleaning → EDA → insight → rekomendasi)
├── data/
│   ├── raw/                                   # data mentah/original
│   │   ├── alfa_pos_transactions.csv
│   │   ├── alfa_products.csv
│   │   └── alfa_stores.csv
│   └── clean/
│       └── alfa_pos_transactions_clean.csv    # data hasil cleaning
├── dashboard/
│   └── Alfamart_POS_Dashboard.twbx            # dashboard Tableau (packaged workbook)
└── presentation/
    └── Capstone2_Alfamart_Analysis.pptx       # slide presentasi hasil analisis
```

## ▶️ Cara Menjalankan Ulang Analisis

```bash
pip install pandas numpy matplotlib seaborn
jupyter notebook notebook/Capstone2_Alfamart_Analysis.ipynb
```

Jalankan seluruh cell secara berurutan (Run All) — notebook akan otomatis membaca 3 file mentah di `data/raw/`, melakukan cleaning, dan meng-export ulang `data/clean/alfa_pos_transactions_clean.csv`.

## ⚠️ Keterbatasan Analisis

- Tidak ada kolom eksplisit "flag promo JSM" di data mentah, sehingga deteksi periode JSM memakai proxy hari Jumat–Sabtu–Minggu.
- Tidak ada data harga promo aktual vs harga normal, sehingga estimasi kerugian margin dari promo baru sebatas potensi konsentrasi volume, bukan nilai rupiah margin yang presisi.
- Tidak ada data internal karyawan untuk membedakan void akibat pelanggan vs murni kesalahan kasir.

---

