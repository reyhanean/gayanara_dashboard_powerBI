# Dashboard Penjualan & Ulasan Pelanggan Gayanara (Power BI)

Dashboard interaktif dua halaman yang dibuat dengan **Power BI Desktop** untuk menganalisis performa penjualan toko pakaian online **Gayanara** dan kepuasan pelanggannya, dari Januari 2022 sampai Februari 2025.

> Proyek portofolio data analyst: pemodelan data, penulisan DAX, dan visualisasi dari lima tabel mentah sampai menjadi dashboard.

---

## Tujuan Analisis

Dashboard ini menjawab pertanyaan bisnis berikut:

1. Bagaimana tren omzet dari bulan ke bulan?
2. Subkategori produk mana yang paling menyumbang omzet, dan mana yang paling lemah?
3. Kota mana yang paling besar kontribusinya?
4. Berapa persen pesanan yang gagal (dibatalkan atau diretur)?
5. Seberapa puas pelanggan, dan subkategori mana yang ratingnya paling tinggi?
6. Metode pembayaran dan kurir apa yang paling sering dipakai?

---

## Dataset

Lima tabel dalam satu folder (`gayanara`):

| Tabel | Isi | Kolom penting |
|---|---|---|
| `customers` | Data pelanggan | `customer_id`, `gender`, `age_group`, `city`, `province` |
| `orders` | Data pesanan | `order_id`, `customer_id`, `order_date`, `order_status`, `payment_method`, `courier`, `promo_code`, `shipping_city` |
| `order_items` | Rincian barang per pesanan | `order_id`, `product_id`, `quantity`, `unit_price_idr`, `subtotal_idr` |
| `products` | Katalog produk | `product_id`, `category`, `sub_category`, `brand`, `price_idr` |
| `reviews` | Ulasan pelanggan | `review_id`, `product_id`, `customer_id`, `rating`, `review_date` |

Cakupan data: **3.000 pesanan**, Januari 2022 sampai Februari 2025.

---

## Tools

- **Power BI Desktop**: pemodelan data dan visualisasi
- **DAX**: pembuatan measure
- **Excel / CSV**: sumber data

---

## Proses Pengerjaan

### 1. Pemodelan data

Kelima tabel dihubungkan dengan relasi *one-to-many*:

- `customers` → `orders`
- `orders` → `order_items`
- `products` → `order_items`
- `products` → `reviews`
- `customers` → `reviews`

Semua measure disimpan dalam satu tabel khusus bernama `_MEASURE` supaya rapi.

### 2. Aturan bisnis

Pesanan berstatus **cancelled** (328) dan **returned** (151) **tidak dihitung sebagai omzet**. Hanya pesanan **delivered** (1.776), **shipped** (445), dan **processing** (300) yang dihitung, sehingga ada **2.521 pesanan** yang masuk analisis.

### 3. Measure DAX

```DAX
Total Omzet =
CALCULATE(
    SUM(order_items[subtotal_idr]),
    KEEPFILTERS(orders[order_status] IN {"delivered", "shipped", "processing"})
)

Jumlah Pesanan =
CALCULATE(
    DISTINCTCOUNT(orders[order_id]),
    KEEPFILTERS(orders[order_status] IN {"delivered", "shipped", "processing"})
)

Total Unit =
CALCULATE(
    SUM(order_items[quantity]),
    KEEPFILTERS(orders[order_status] IN {"delivered", "shipped", "processing"})
)

Jumlah Pelanggan =
CALCULATE(
    DISTINCTCOUNT(orders[customer_id]),
    KEEPFILTERS(orders[order_status] IN {"delivered", "shipped", "processing"})
)

AVG Order Value = DIVIDE([Total Omzet], [Jumlah Pesanan])

Cancel Rate =
DIVIDE(
    CALCULATE(DISTINCTCOUNT(orders[order_id]), orders[order_status] = "cancelled"),
    DISTINCTCOUNT(orders[order_id])
)

Return Rate =
DIVIDE(
    CALCULATE(DISTINCTCOUNT(orders[order_id]), orders[order_status] = "returned"),
    DISTINCTCOUNT(orders[order_id])
)

AVG Rating = AVERAGE(reviews[rating])

Jumlah Review = COUNTROWS(reviews)
```

`KEEPFILTERS` dipakai supaya filter status tidak menimpa filter dari visual atau slicer lain.

### 4. Visualisasi

**Halaman 1: Ringkasan Penjualan**

- Card: Total Omzet, Jumlah Pesanan, Total Unit, Jumlah Pelanggan, AVG Order Value, Cancel Rate, Return Rate
- Line chart: omzet per bulan (Jan 2022 sampai Feb 2025)
- Bar chart: 10 kota dengan omzet tertinggi
- Column chart: omzet per subkategori
- Slicer: tahun

**Halaman 2: Pelanggan & Ulasan**

- Card: AVG Rating, Jumlah Review
- Column chart: jumlah pesanan per kurir
- Pie chart: jumlah pesanan per metode pembayaran
- Bar chart: rata-rata rating per subkategori
- Column chart: distribusi rating (1 sampai 5)

---

## Insight Utama

Periode Januari 2022 sampai Februari 2025.

**Ringkasan angka**

| Metrik | Nilai |
|---|---|
| Total omzet | Rp1.296.768.000 |
| Jumlah pesanan | 2.521 |
| Total unit terjual | 5.442 |
| Jumlah pelanggan | 768 |
| Rata-rata nilai pesanan | Rp514.386 |
| Cancel rate | 10,93% |
| Return rate | 5,03% |
| Rata-rata rating | 4,02 dari 1.500 review |

**Temuan**

1. **Omzet tumbuh dari tahun ke tahun.** Omzet bulanan naik dari sekitar 10 juta (Juni 2022) menjadi sekitar 68 juta (Februari 2025), dengan lonjakan tajam di Januari 2023.
2. **Atasan Wanita adalah produk andalan.** Omzetnya sekitar 450 juta atau sekitar 35% dari total, hampir dua kali lipat subkategori terbesar berikutnya (Outwear dan Aksesoris, sekitar 220 juta).
3. **Bawahan paling lemah.** Bawahan Pria dan Bawahan Wanita digabung hanya sekitar 17% dari omzet, jadi ada peluang promo atau bundling.
4. **Pasar tersebar merata.** Semarang, Depok, dan Medan masing-masing sekitar 78 juta, dan kota kesepuluh masih sekitar 66 juta. Omzet tidak bergantung pada satu kota.
5. **Sekitar 16% pesanan gagal** (batal 10,93% dan retur 5,03%), dan pesanan ini tidak menjadi omzet.
6. **Pelanggan umumnya puas.** Sekitar 73% review bernilai 4 atau 5, dan hanya sekitar 11% bernilai 1 atau 2. Rating antar subkategori rata di kisaran 3,90 sampai 4,07, dengan Atasan Pria dan Bawahan Pria sedikit paling rendah.
7. **Dompet digital mendominasi.** GoPay, OVO, dan QRIS digabung sekitar 55% dari pesanan. Transfer Bank metode tunggal terbesar (30,35%).
8. **Pengiriman terpusat di tiga kurir.** J&T, JNE, dan SiCepat menangani sekitar 84% pesanan.

---

## Screenshot

| Halaman 1: Ringkasan Penjualan | Halaman 2: Pelanggan & Ulasan |
|---|---|
| ![Halaman 1](screenshots/halaman-1.png) | ![Halaman 2](screenshots/halaman-2.png) |

---

## Struktur Repositori

```
gayanara-dashboard/
├── gayanara_dashboard.pbix
├── data/
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── products.csv
│   └── reviews.csv
├── screenshots/
│   ├── halaman-1.png
│   └── halaman-2.png
└── README.md
```

## Cara Membuka

1. Unduh dan pasang [Power BI Desktop](https://powerbi.microsoft.com/desktop) (gratis, khusus Windows).
2. Unduh repositori ini, lalu buka `gayanara_dashboard.pbix`.
3. Jika muncul pesan data tidak ditemukan: **Home → Transform data → Data source settings → Change Source**, lalu arahkan ke folder `data/` di komputer kamu.

---

## Yang Dipelajari

- Membangun model data dari beberapa tabel dengan relasi satu ke banyak
- Memakai `CALCULATE` dan `KEEPFILTERS` untuk menerapkan aturan bisnis pada measure
- Memisahkan measure ke tabel khusus (`_MEASURE`)
- Menyusun tata letak dashboard yang rapi dan interaktif dengan slicer
- Mengubah angka menjadi insight yang bisa ditindaklanjuti

## Pengembangan Selanjutnya

- Analisis promo dan diskon (`promo_code`, `discount_amount_idr`)
- Analisis per wilayah atau provinsi
- Segmentasi pelanggan berdasarkan `age_group` dan `gender`
- Perbandingan tahun ke tahun (YoY growth)

---

## Pembuat

**Muhamad Rayhan**
Mahasiswa Teknik Informatika yang sedang mendalami analisis data.

- GitHub: [username-kamu](https://github.com/username-kamu)
- LinkedIn: [nama-kamu](https://linkedin.com/in/nama-kamu)
