# Panduan Setup Data ERP â€” Untuk Admin Sistem

> Dokumen ini menjelaskan **step-by-step** cara menyiapkan data di sistem ERP agar operasional bisa berjalan, mulai dari cabang baru sampai pengantaran ke customer.

---

## Daftar Isi

1. [Urutan Setup (Peta Ketergantungan)](#1-urutan-setup--peta-ketergantungan)
2. [Fase 1: Perusahaan / Cabang](#2-fase-1-perusahaan--cabang)
3. [Fase 2: User & Karyawan](#3-fase-2-user--karyawan)
4. [Fase 3: Gudang (Stock Location)](#4-fase-3-gudang-stock-location)
5. [Fase 4: Satuan (UOM)](#5-fase-4-satuan-uom)
6. [Fase 5: Produk](#6-fase-5-produk)
7. [Fase 6: Bill of Materials (BOM)](#7-fase-6-bill-of-materials-bom)
8. [Fase 7: Zona Pengantaran](#8-fase-7-zona-pengantaran)
9. [Fase 8: Customer](#9-fase-8-customer)
10. [Fase 9: Kendaraan (Fleet)](#10-fase-9-kendaraan-fleet)
11. [Fase 10: Harga & Pricing](#11-fase-10-harga--pricing)
12. [Contoh Kasus Lengkap: Setup Cabang Baru](#12-contoh-kasus-lengkap-setup-cabang-baru)
13. [Checklist Kesiapan Operasional](#13-checklist-kesiapan-operasional)

---

## 1. Urutan Setup â€” Peta Ketergantungan

Setup **HARUS** dilakukan berurutan karena ada ketergantungan antar data:

```
FASE 1  Perusahaan/Cabang
   â”‚
   â”œâ”€â”€â–º FASE 2  User & Karyawan (butuh: Perusahaan)
   â”‚
   â”œâ”€â”€â–º FASE 3  Gudang / Stock Location (butuh: Perusahaan)
   â”‚
   â”œâ”€â”€â–º FASE 4  Satuan / UOM (independen, biasanya sudah ada)
   â”‚       â”‚
   â”‚       â””â”€â”€â–º FASE 5  Produk (butuh: UOM)
   â”‚               â”‚
   â”‚               â””â”€â”€â–º FASE 6  BOM (butuh: Produk + UOM)
   â”‚
   â”œâ”€â”€â–º FASE 7  Zona Pengantaran (butuh: Perusahaan + Karyawan/Kurir)
   â”‚       â”‚
   â”‚       â””â”€â”€â–º FASE 8  Customer (butuh: Perusahaan + Zona)
   â”‚
   â””â”€â”€â–º FASE 9  Kendaraan/Fleet (butuh: Perusahaan + Karyawan)

FASE 10  Harga (butuh: Produk sudah ada)
```

> **Aturan emas**: Jangan loncat fase. Data di fase bawah butuh data di fase atasnya.

---

## 2. Fase 1: Perusahaan / Cabang

### Kapan Dibutuhkan?
Saat membuka cabang baru atau setup awal sistem.

### Data yang Diperlukan

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama Perusahaan** | âœ… | WAJIB PROLIMAN | Nama resmi cabang |
| **Kode** | âœ… | PROLIMAN | Kode singkat, huruf kapital |
| Alamat (Jalan) | - | Jl. Jend. A. Yani No. 123 | Alamat lengkap |
| Kota | - | Cilacap | Kota/Kabupaten |
| Provinsi | - | Jawa Tengah | Pilih dari daftar wilayah |
| Telepon | - | (0282) 123456 | Nomor kantor |
| Email | - | proliman@wajib.co.id | Email resmi |
| Koordinat GPS | - | -7.6850, 109.0125 | Untuk fitur absensi radius |
| **Radius Absensi (meter)** | - | 200 | Jarak maksimal karyawan bisa absen |
| **Biaya Antar per Item** | - | 0 | Rp surcharge per galon antar |
| **Induk Perusahaan** | âœ… | Wajib Indonesia | Cabang â†’ pilih kantor pusat |

### Apa yang Otomatis Terjadi Setelah Dibuat?
- Gudang default (STOCK, VENDOR, CUSTOMER) **harus dibuat manual** di Fase 3
- Produk **berlaku global** (semua cabang bisa lihat produk yang sama)

---

## 3. Fase 2: User & Karyawan

### Kapan Dibutuhkan?
Setelah cabang terdaftar, untuk menambah orang yang bisa login dan bekerja.

### Data User (untuk Login)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Budi Santoso | Nama lengkap |
| **Email** | âœ… | budi@wajib.co.id | Untuk login |
| **Password** | âœ… | (min 8 karakter) | Password awal |
| **Role** | âœ… | Staff Pengantaran | Menentukan akses menu & fitur |
| **Cabang** | âœ… | WAJIB PROLIMAN | Cabang tempat kerja |

### Daftar Role yang Tersedia

| Role | Akses | Cocok Untuk |
|------|-------|-------------|
| **Super Admin** | Semua fitur | Developer / IT |
| **Owner** | Lihat semua laporan, setting terbatas | Pemilik usaha |
| **Manager** | Semua operasional cabang | Kepala cabang |
| **Spv** | Monitoring + update stok/produksi | Supervisor lapangan |
| **Admin** | Inventori + produksi + stok | Admin gudang |
| **Staff Pengantaran** | Delivery + rute + absensi + fleet | Kurir |
| **Staff Produksi** | Stok + produksi + absensi | Operator produksi |

### Data Karyawan (HrEmployee â€” setelah user dibuat)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **User** | âœ… | (pilih user yang baru dibuat) | Link ke akun login |
| **Cabang** | âœ… | WAJIB PROLIMAN | Sama dengan user |
| **Nomor Karyawan** | âœ… | STF-PROL-001 | Unik per karyawan |
| **Nama** | âœ… | Budi Santoso | Nama display |
| **Jabatan** | âœ… | Kurir | Deskripsi jabatan |
| Departemen | - | Distribution | Unit kerja |
| Telepon | - | 081234567890 | Kontak HP |
| Tanggal Masuk | - | 2026-01-15 | Mulai bekerja |

> **Penting**: Untuk kurir, data karyawan **harus ada** sebelum bisa assign ke zona pengantaran (Fase 7).

---

## 4. Fase 3: Gudang (Stock Location)

### Kapan Dibutuhkan?
Setiap cabang baru butuh minimal 3 lokasi gudang.

### Data yang Diperlukan (per Cabang)

Untuk setiap cabang, buat 3 lokasi berikut:

| Nama | Kode | Tipe | Default? | Fungsi |
|------|------|------|:--------:|--------|
| **Stock** | STOCK | Internal | âœ… | Gudang utama (stok fisik) |
| **Vendor** | VENDOR | Vendor | - | Lokasi virtual masuk barang (PO) |
| **Customer** | CUSTOMER | Customer | - | Lokasi virtual keluar barang (SO) |

### Contoh Penamaan

```
PROLIMAN/STOCK    â†’ Gudang utama cabang Proliman
PROLIMAN/VENDOR   â†’ Sumber barang dari supplier
PROLIMAN/CUSTOMER â†’ Tujuan pengiriman ke pelanggan
```

> **Catatan**: Tanpa lokasi STOCK yang di-set sebagai default, sistem tidak bisa memproses penerimaan barang maupun pengiriman.

---

## 5. Fase 4: Satuan (UOM)

### Kapan Dibutuhkan?
Biasanya **sudah tersedia** dari setup awal. Hanya perlu ditambah jika ada satuan khusus.

### Satuan yang Sudah Ada di Sistem

| Kategori | Satuan | Rasio | Penggunaan |
|----------|--------|------:|------------|
| **Unit** | Unit | 1 | Galon, tutup, segel (satuan hitung) |
| | Dozen | 12 | 12 unit |
| | Dus 24 | 24 | Kemasan 24 gelas |
| | Dus 48 | 48 | Kemasan 48 gelas |
| **Volume** | Liter | 1 | Stok air baku |
| | Milliliter | 0.001 | Takaran kecil |
| | **Tangki** | **8000** | **Pembelian air baku (1 tangki = 8.000 L)** |
| | Cubic Meter | 1000 | 1 mÂ³ = 1.000 L |
| **Berat** | Kilogram | 1 | Es kristal, bahan lain |
| | Gram | 0.001 | Takaran kecil |

### Konsep Penting

```
Konversi otomatis terjadi berdasarkan KATEGORI yang sama:
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
  â”‚  Kategori: Volume                             â”‚
  â”‚  Liter (dasar) â† Tangki (8000x) â†’ otomatis   â”‚
  â”‚                                               â”‚
  â”‚  Beli: 2 Tangki  â†’  Stok masuk: 16.000 Liter â”‚
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

- **UOM Stok** (`uom_id`): Satuan penyimpanan di gudang
- **UOM Beli** (`uom_po_id`): Satuan saat beli dari supplier
- Dua UOM ini **boleh berbeda** selama satu kategori yang sama

### Kapan Perlu Tambah UOM Baru?
- Ada satuan beli khusus dari supplier (misal: "Drum" = 200 Liter)
- Ada kemasan baru (misal: "Karton 12" = 12 unit)

---

## 6. Fase 5: Produk

### Kapan Dibutuhkan?
Saat ada produk baru yang dijual, diproduksi, atau dibeli.

### Jenis-Jenis Produk di Sistem

| Jenis | Contoh | Dijual? | Dibeli? | Diproduksi? | Dilacak Stok? |
|-------|--------|:-------:|:-------:|:-----------:|:-------------:|
| **Galon Isi (Manufactured)** | Air Mineral Biasa 19L | âœ… | - | âœ… | âœ… |
| **Bahan Baku (Raw Material)** | Air Baku Gunung | - | âœ… | - | âœ… |
| **Komponen** | Galon Kosong, Tutup, Segel | - | âœ… | - | âœ… |
| **Service** | Biaya Galon Baru, Denda | âœ… | - | - | - |
| **Custom Refill** | Isi ulang wadah customer | âœ… | - | - | - |

### Data Produk â€” Field yang Harus Diisi

#### A. Produk Galon Isi (Manufactured)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Air Mineral Biasa 19L | Nama tampil |
| **Kode/SKU** | âœ… | AIR_MINERAL_BIASA | Kode unik, huruf kapital |
| **Tipe** | âœ… | Produk | "Produk" = dilacak stoknya |
| **UOM Stok** | âœ… | Unit | Satuan di gudang |
| **UOM Beli** | âœ… | Unit | Satuan saat PO (biasanya sama) |
| **Bisa Dijual?** | âœ… | Ya | Tampil di Sales Order |
| **Bisa Diproduksi?** | âœ… | Ya | Bisa dibuat Manufacturing Order |
| **Harga Jual** | âœ… | 8.000 | Harga per galon ke customer |
| **Harga Pokok** | - | 3.500 | HPP per galon (otomatis dari BOM) |
| Barcode | - | 8992761100001 | Untuk scan |
| Deskripsi | - | Air mineral... | Info tambahan |
| Kategori | - | Finished Goods | Pengelompokan |

#### B. Bahan Baku (Raw Material)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Air Baku Gunung | Nama tampil |
| **Kode/SKU** | âœ… | AIR-BAKU-GUNUNG | Kode unik |
| **Tipe** | âœ… | Produk | Dilacak stoknya |
| **UOM Stok** | âœ… | Liter | Stok dalam liter |
| **UOM Beli** | âœ… | **Tangki** | Beli per tangki (otomatis konversi) |
| **Bisa Dibeli?** | âœ… | Ya | Tampil di Purchase Order |
| **Bisa Dijual?** | âœ… | Tidak | Bahan baku, tidak dijual langsung |
| Kategori | - | Bahan Baku | Pengelompokan |

> **Contoh Konversi Pembelian**:
> - Input PO: 2 Tangki Ã— Rp 4.000.000/Tangki = Rp 8.000.000
> - Masuk gudang: 16.000 Liter, HPP Rp 500/Liter

#### C. Komponen (Galon Kosong, Tutup, Segel)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Galon Kosong 19L | Nama tampil |
| **Kode/SKU** | âœ… | GALON-001 | Kode unik |
| **Tipe** | âœ… | Produk | Dilacak stoknya |
| **UOM Stok** | âœ… | Unit | Per biji |
| **Bisa Dibeli?** | âœ… | Ya | Beli dari supplier kemasan |

#### D. Service (Jasa)

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Biaya Galon Baru | Nama tampil |
| **Kode/SKU** | âœ… | PROD-BTL | Kode unik |
| **Tipe** | âœ… | Service | **Tidak** dilacak stok |
| **Bisa Dijual?** | âœ… | Ya | Otomatis masuk SO saat needed |
| **Harga** | âœ… | 35.000 | Harga per event |

### Produk yang Biasa Ada di Sistem Ini

```
PRODUK JUAL (Galon 19L):
  â”œâ”€â”€ Air Mineral Biasa 19L     â†’ Rp 8.000/galon
  â”œâ”€â”€ Air Mineral Pegunungan 19L â†’ Rp 10.000/galon
  â”œâ”€â”€ Air RO 19L                â†’ Rp 12.000/galon
  â””â”€â”€ Air Alkali pH 9+ 19L     â†’ Rp 15.000/galon (opsional per cabang)

BAHAN BAKU:
  â”œâ”€â”€ Air Baku Gunung           â†’ Beli per Tangki, stok per Liter
  â””â”€â”€ Air Baku PAM              â†’ Beli per Tangki, stok per Liter

KOMPONEN:
  â”œâ”€â”€ Galon Kosong 19L          â†’ Beli baru
  â”œâ”€â”€ Galon Bekas 19L           â†’ Dari retur customer
  â”œâ”€â”€ Tutup <per jenis air>     â†’ 1 per galon
  â””â”€â”€ Segel <per jenis air>     â†’ 1 per galon

SERVICE:
  â”œâ”€â”€ Biaya Galon Baru          â†’ Rp 35.000 (customer beli galon baru)
  â”œâ”€â”€ Denda Galon Rusak         â†’ Rp 40.000
  â”œâ”€â”€ Biaya Pinjam Galon (Awal) â†’ Rp 5.000
  â””â”€â”€ Biaya Pinjam Galon (Bulanan) â†’ Rp 5.000/bulan
```

---

## 7. Fase 6: Bill of Materials (BOM)

### Kapan Dibutuhkan?
Setelah produk **manufactured** dan **bahan baku** dibuat. BOM menentukan resep produksi.

### Data BOM Header

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | BOM Air Mineral Biasa | Nama resep |
| **Produk Hasil** | âœ… | Air Mineral Biasa 19L | Produk yang dihasilkan |
| **Qty Hasil** | âœ… | 1 | Biasanya 1 (= 1 galon) |
| **UOM Hasil** | âœ… | Unit | Satuan output |

### Data BOM Line (Bahan per Galon)

| Urutan | Komponen | Qty | UOM | Fleksibel? | Prioritas |
|:------:|----------|----:|-----|:----------:|:---------:|
| 1A | Galon Bekas 19L | 1 | Unit | **Ya** | 1 (utama) |
| 1B | Galon Kosong 19L | 1 | Unit | **Ya** | 2 (cadangan) |
| 2 | Air Baku Gunung | 19 | Liter | Tidak | - |
| 3 | Tutup Mineral Biasa | 1 | Unit | Tidak | - |
| 4 | Segel Mineral Biasa | 1 | Unit | Tidak | - |

### Konsep "Komponen Fleksibel"

```
Sistem otomatis memilih:
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
  â”‚  Mau produksi 100 galon                      â”‚
  â”‚  Stok Galon Bekas: 70 â†’ pakai 70 (prioritas 1) â”‚
  â”‚  Kurang: 30 â†’ ambil Galon Kosong (prioritas 2)  â”‚
  â”‚  Total: 70 bekas + 30 baru = 100 galon      â”‚
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

> **Tips**: Setiap jenis air punya BOM sendiri karena tutup & segel berbeda warna/kode.

---

## 8. Fase 7: Zona Pengantaran

### Kapan Dibutuhkan?
Setelah karyawan kurir sudah ada. Zona menentukan **siapa antar ke mana**.

### Data yang Diperlukan

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama Zona** | âœ… | KESUGIHAN-1 | Nama area pengantaran |
| **Kurir** | âœ… | Budi (STF-PROL-001) | Karyawan yang bertanggung jawab |
| **Cabang** | âœ… | WAJIB PROLIMAN | Cabang pemilik zona |
| Provinsi | - | Jawa Tengah | Filter wilayah |
| Kabupaten | - | Cilacap | Filter wilayah |
| Kecamatan | - | Kesugihan | Filter wilayah |
| Desa | - | Kesugihan Lor | Filter wilayah |
| Area (GeoJSON) | - | (polygon peta) | Untuk mapping visual |
| **Status** | âœ… | Aktif | Aktif / Nonaktif |

### Hubungan Zona

```
Cabang PROLIMAN
  â””â”€â”€ Zona: KESUGIHAN-1
        â”œâ”€â”€ Kurir: Budi (STF-PROL-001)
        â”œâ”€â”€ Area: Kec. Kesugihan, Cilacap
        â””â”€â”€ Customer di zona ini:
              â”œâ”€â”€ Agus Prasetyo
              â”œâ”€â”€ Warung Bu Tini
              â””â”€â”€ Depot Sehat Jaya
```

> **1 zona = 1 kurir**. Jika 1 kurir handle 2 area, buat 2 zona yang menunjuk ke kurir yang sama.

---

## 9. Fase 8: Customer

### Kapan Dibutuhkan?
Setelah zona pengantaran sudah dibuat. Customer harus terhubung ke zona.

### Data yang Diperlukan

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Nama** | âœ… | Warung Makan Bu Tini | Nama customer |
| **Tipe** | âœ… | Bisnis / Perorangan | is_company = true/false |
| **Cabang** | âœ… | WAJIB PROLIMAN | Cabang pemilik |
| **Zona Pengantaran** | âœ… | KESUGIHAN-1 | Zona antar â†’ menentukan kurir |
| Telepon/HP | - | 081298765100 | Untuk konfirmasi order |
| Alamat | - | Jl. Pasar Kesugihan No.5 | Alamat lengkap |
| Kota | - | Cilacap | Kota |
| Koordinat GPS | - | -7.6400, 109.1200 | Untuk navigasi kurir |
| Email | - | tini@email.com | Opsional |

### Customer Khusus: "Pelanggan Umum"

Setiap cabang punya 1 customer dummy bernama **"Pelanggan Umum"** untuk:
- Penjualan walk-in / cash tanpa nama
- Order pickup langsung di depot
- Tidak memerlukan zona pengantaran

### Tips Input Customer

```
âœ… BENAR:
   Customer "Bu Tini" â†’ Zona "KESUGIHAN-1" â†’ Kurir "Budi"
   (Saat ada order Bu Tini, sistem otomatis assign ke Budi)

âŒ SALAH:
   Customer "Bu Tini" â†’ Tidak ada zona
   (Order tidak bisa di-assign ke kurir manapun)
```

---

## 10. Fase 9: Kendaraan (Fleet)

### Kapan Dibutuhkan?
Setelah karyawan kurir sudah ada. Untuk tracking kendaraan operasional.

### Data yang Diperlukan

| Field | Wajib | Contoh | Keterangan |
|-------|:-----:|--------|------------|
| **Cabang** | âœ… | WAJIB PROLIMAN | Pemilik kendaraan |
| **Penanggung Jawab** | âœ… | Budi (STF-PROL-001) | Karyawan yang pakai |
| **Nomor Polisi** | âœ… | R 1234 AB | Plat nomor |
| **Merek** | âœ… | Honda | Merek kendaraan |
| **Model** | âœ… | Scoopy | Tipe kendaraan |
| Tahun | - | 2024 | Tahun pembuatan |
| Warna | - | Putih | Warna body |
| No. Mesin | - | JFM2E1234567 | Identifikasi mesin |
| No. Rangka | - | MHJFM21EK12345 | Identifikasi rangka |
| KM Saat Ini | - | 5200 | Odometer terakhir |
| Tanggal Beli | - | 2024-06-15 | Tracking aset |
| Harga Beli | - | 22.000.000 | Nilai aset |
| **Status** | âœ… | Aktif | Aktif / Maintenance / Retired |
| Foto | - | (upload) | Foto kendaraan |

### Fitur Fleet yang Tersedia

- **Odometer Log**: Catat KM harian
- **Service Log**: Riwayat servis & perbaikan
- **Service Schedule**: Jadwal servis berkala
- **P2H (Pengecekan Harian)**: Checklist kondisi kendaraan sebelum jalan

---

## 11. Fase 10: Harga & Pricing

### Sistem Harga

Harga di sistem ini **ada di level produk**, bukan di price list terpisah:

| Field | Lokasi | Fungsi |
|-------|--------|--------|
| **Harga Jual** (`list_price`) | Produk | Harga default ke customer |
| **Harga Pokok** (`standard_price`) | Produk | HPP / cost dari produksi/pembelian |
| **Biaya Antar** (`delivery_surcharge_per_item`) | Perusahaan | Surcharge per galon per cabang |

### Contoh Harga Produk

| Produk | Harga Jual | HPP |
|--------|----------:|----:|
| Air Mineral Biasa 19L | 8.000 | 3.500 |
| Air Mineral Pegunungan 19L | 10.000 | 4.000 |
| Air RO 19L | 12.000 | 5.000 |
| Biaya Galon Baru | 35.000 | - |
| Denda Galon Rusak | 40.000 | - |
| Biaya Pinjam Galon | 5.000 | - |

### Harga Otomatis di Sales Order

Saat membuat order, harga otomatis terisi dari `list_price` produk. Admin bisa override harga per baris jika diperlukan.

---

## 12. Contoh Kasus Lengkap: Setup Cabang Baru

> **Skenario**: Buka cabang baru "WAJIB MAJENANG" di Kecamatan Majenang, Cilacap.

### Step-by-Step

```
STEP 1 â€” Buat Perusahaan
  Nama    : WAJIB MAJENANG
  Kode    : MAJENANG
  Alamat  : Jl. Raya Majenang No. 10, Cilacap
  Induk   : Wajib Indonesia
  Radius  : 200 meter

STEP 2 â€” Buat User & Karyawan
  a) Manager:
     User  : manager.majenang@wajib.co.id (Role: Manager)
     Karyawan: MGR-MAJ-001, Jabatan: Manager Cabang

  b) Admin Gudang:
     User  : admin.majenang@wajib.co.id (Role: Admin)
     Karyawan: ADM-MAJ-001, Jabatan: Admin Gudang

  c) Kurir 1:
     User  : kurir1.majenang@wajib.co.id (Role: Staff Pengantaran)
     Karyawan: STF-MAJ-001, Jabatan: Kurir

  d) Kurir 2:
     User  : kurir2.majenang@wajib.co.id (Role: Staff Pengantaran)
     Karyawan: STF-MAJ-002, Jabatan: Kurir

  e) Staff Produksi:
     User  : produksi.majenang@wajib.co.id (Role: Staff Produksi)
     Karyawan: PRD-MAJ-001, Jabatan: Operator Produksi

STEP 3 â€” Buat Gudang
  MAJENANG/STOCK    (Internal, Default âœ…)
  MAJENANG/VENDOR   (Vendor)
  MAJENANG/CUSTOMER (Customer)

STEP 4 â€” Produk & UOM
  â†’ Sudah ada (berlaku global untuk semua cabang)
  â†’ Tidak perlu buat ulang

STEP 5 â€” BOM
  â†’ Sudah ada (berlaku global)
  â†’ Tidak perlu buat ulang

STEP 6 â€” Buat Zona Pengantaran
  a) MAJENANG-KOTA
     Kurir     : Andi (STF-MAJ-001)
     Kecamatan : Majenang
     Desa      : Majenang (pusat kota)

  b) MAJENANG-SELATAN
     Kurir     : Eko (STF-MAJ-002)
     Kecamatan : Majenang
     Desa      : Sadabumi, Padangjaya

STEP 7 â€” Buat Customer
  a) Toko Pak Joko
     Zona    : MAJENANG-KOTA
     Telepon : 081234567890
     Alamat  : Jl. Pasar Majenang No. 3

  b) RM Sederhana
     Zona    : MAJENANG-KOTA
     Telepon : 081234567891
     Alamat  : Jl. Raya Majenang No. 15

  c) Pabrik Tahu Bu Wati
     Zona    : MAJENANG-SELATAN
     Telepon : 081234567892
     Alamat  : Desa Sadabumi RT 03/05

STEP 8 â€” Buat Kendaraan
  a) Motor Kurir 1
     Penanggung Jawab : Andi (STF-MAJ-001)
     Plat  : R 5678 CD
     Merek : Honda Scoopy
     Status: Aktif

  b) Motor Kurir 2
     Penanggung Jawab : Eko (STF-MAJ-002)
     Plat  : R 9012 EF
     Merek : Yamaha NMAX
     Status: Aktif

STEP 9 â€” Set Harga (jika beda dari default)
  â†’ Gunakan harga default global
  â†’ Ubah di produk jika cabang punya harga berbeda

  âœ… CABANG MAJENANG SIAP BEROPERASI!
```

---

## 13. Checklist Kesiapan Operasional

Gunakan checklist ini untuk memastikan cabang siap sebelum mulai operasional:

### Data Master

| # | Item | Status |
|:-:|------|:------:|
| 1 | Perusahaan/Cabang terdaftar | â˜ |
| 2 | Gudang STOCK, VENDOR, CUSTOMER sudah dibuat | â˜ |
| 3 | Manager sudah punya akun & login berhasil | â˜ |
| 4 | Admin gudang sudah punya akun | â˜ |
| 5 | Minimal 1 kurir sudah punya akun + data karyawan | â˜ |
| 6 | Staff produksi sudah punya akun | â˜ |

### Produk & BOM

| # | Item | Status |
|:-:|------|:------:|
| 7 | Produk galon isi sudah ada (min. 1 jenis air) | â˜ |
| 8 | Bahan baku (air baku) sudah ada | â˜ |
| 9 | Komponen (galon, tutup, segel) sudah ada | â˜ |
| 10 | BOM sudah dibuat untuk setiap jenis air | â˜ |

### Delivery

| # | Item | Status |
|:-:|------|:------:|
| 11 | Minimal 1 zona pengantaran sudah dibuat | â˜ |
| 12 | Zona sudah di-assign ke kurir | â˜ |
| 13 | Minimal 1 customer terdaftar di zona | â˜ |
| 14 | Customer "Pelanggan Umum" sudah ada | â˜ |

### Kendaraan

| # | Item | Status |
|:-:|------|:------:|
| 15 | Kendaraan kurir sudah terdaftar | â˜ |
| 16 | Kendaraan sudah di-assign ke karyawan | â˜ |

### Stok Awal

| # | Item | Status |
|:-:|------|:------:|
| 17 | Stok awal air baku sudah diinput (via PO atau Stock In) | â˜ |
| 18 | Stok awal galon kosong sudah diinput | â˜ |
| 19 | Stok awal tutup & segel sudah diinput | â˜ |
| 20 | Test produksi 1 galon berhasil | â˜ |

---

## Glosarium

| Istilah | Arti |
|---------|------|
| **UOM** | Unit of Measure â€” satuan ukur (Liter, Unit, Tangki, dll) |
| **PO** | Purchase Order â€” form pembelian barang ke supplier |
| **SO** | Sales Order â€” form penjualan barang ke customer |
| **BOM** | Bill of Materials â€” daftar bahan baku untuk memproduksi 1 produk |
| **MO** | Manufacturing Order â€” perintah produksi berdasarkan BOM |
| **Stock In** | Penerimaan stok masuk ke gudang |
| **Stock Move** | Perpindahan barang antar lokasi |
| **Delivery Zone** | Area pengantaran yang di-assign ke 1 kurir |
| **Fleet** | Kendaraan operasional (motor/mobil) |
| **P2H** | Pemeriksaan 2 Harian â€” checklist kondisi kendaraan |
| **HPP** | Harga Pokok Penjualan â€” biaya produksi per unit |
