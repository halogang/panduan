# 🏪 Analisis & Perencanaan: Owner Cabang (Model Franchise Depot Air Minum)

> **Dokumen**: Planning & Requirement - Owner Cabang / Franchise Model  
> **Tanggal**: 27 Maret 2026  
> **Status**: DRAFT - Menunggu Review  
> **Konteks**: Wajib adalah depot air minum yang akan memiliki banyak cabang (seperti model Alfamart). Setiap cabang memiliki Owner Cabang yang mengelola operasional cabangnya sendiri.

---

## 1. Analogi Bisnis: Alfamart Franchise Model

### Bagaimana Alfamart Bekerja

```
PT Sumber Alfaria Trijaya (Pusat)
├── Alfamart Jl. Gatsu No.12     → Owner: Pak Ahmad (Franchise)
├── Alfamart Jl. Panjaitan No.5  → Owner: Bu Siti (Franchise)
├── Alfamart Jl. Cipto No.8      → Owner: Pak Budi (Franchise)
└── Alfamart Jl. Proliman No.3   → Owner: Bu Dewi (Franchise)

Setiap toko punya owner sendiri yang:
✅ Investasi modal untuk buka toko
✅ Melihat laporan keuangan toko-nya
✅ Menerima setoran harian dari kasir (SPV)
✅ Mengatur karyawan di toko-nya
✅ TAPI operasional harian dijalankan oleh Manager/SPV
```

### Penerapan ke Wajib Air Minum

```
Wajib Indonesia (Pusat / Franchisor)
│
│   👑 Super Admin → Mengelola sistem, semua cabang
│
├── Wajib Gatsu        → 🏪 Owner Cabang: Pak Ahmad
│   ├── Manager Gatsu
│   ├── SPV Gatsu
│   ├── Admin Gatsu
│   ├── Staff Pengantaran Gatsu
│   └── Staff Produksi Gatsu
│
├── Wajib Panjaitan    → 🏪 Owner Cabang: Bu Siti
│   ├── Manager Panjaitan
│   ├── SPV Panjaitan
│   └── ... staff
│
├── Wajib Cipto        → 🏪 Owner Cabang: Pak Budi
│   └── ... staff
│
└── Wajib Proliman     → 🏪 Owner Cabang: Bu Dewi
    └── ... staff
```

---

## 2. Kondisi Saat Ini (As-Is)

### Hirarki Role Saat Ini

| Role | Scope | Keterangan |
|------|-------|------------|
| Super Admin | Semua cabang | Full access, kelola sistem |
| **Owner** | **1 company (MAIN)** | **Helicopter view TAPI hanya lihat data MAIN** |
| Manager | 1 cabang | Operasional penuh cabang |
| SPV | 1 cabang | Monitoring & verifikasi |
| Admin | 1 cabang | Produksi & inventory |
| Staff Pengantaran | 1 cabang | Delivery |
| Staff Produksi | 1 cabang | Manufacturing |

### Masalah dengan Model Sekarang

| # | Masalah | Detail |
|---|---------|--------|
| 1 | **Hanya ada 1 Owner** | Owner saat ini `company_id = MAIN`, tidak per cabang |
| 2 | **Owner tidak bisa lihat semua cabang** | Query di-filter `company_id`, Owner hanya lihat data MAIN |
| 3 | **Tidak ada konsep "kepemilikan cabang"** | Tidak ada relasi antara Owner dan cabang yang dimilikinya |
| 4 | **Setoran ke siapa?** | OwnerDeposit ada `confirmed_by` tapi tidak ada konsep cabang → owner mapping |
| 5 | **Tidak ada pemisahan Franchisor vs Franchisee** | Pusat dan cabang diperlakukan sama |
| 6 | **1 orang bisa buka banyak cabang tapi tidak difasilitasi** | Pak Ahmad bisa punya 3 cabang, tapi sistem belum support |

---

## 3. Model Baru yang Diusulkan

### 3.1. Hirarki Baru

```
┌─────────────────────────────────────────────────────────┐
│                    LEVEL PUSAT                          │
│                                                         │
│  👑 Super Admin     → Full system access                │
│  🏛️ Owner Pusat     → Lihat SEMUA cabang (franchisor)   │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                   LEVEL CABANG                          │
│                                                         │
│  🏪 Owner Cabang    → Lihat & kelola cabang sendiri     │
│  📋 Manager         → Operasional harian cabang         │
│  👁️ SPV             → Monitoring & verifikasi           │
│  ⚙️ Admin           → Produksi & inventory              │
│  🚚 Staff Antarkan  → Delivery                          │
│  🏭 Staff Produksi  → Manufacturing                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 3.2. Role Baru: Owner Cabang

**Definisi**: Pemilik investasi dari satu atau lebih cabang depot air minum Wajib. Bukan karyawan operasional, melainkan pemilik modal yang memantau performa bisnisnya.

| Aspek | Owner Pusat | Owner Cabang |
|-------|-------------|--------------|
| **Scope** | Semua cabang | Hanya cabang yang dimiliki |
| **Dashboard** | Agregasi semua cabang + perbandingan | Dashboard per cabang miliknya |
| **Keuangan** | Lihat total revenue semua cabang | Lihat revenue cabang sendiri |
| **Setoran** | Terima setoran dari semua cabang | Terima setoran dari cabang sendiri |
| **Karyawan** | Lihat semua karyawan | Lihat karyawan di cabangnya |
| **Keputusan** | Buka/tutup cabang, kebijakan pusat | Atur diskon, jadwal, promo lokal |
| **Laporan** | Perbandingan antar cabang | Detail performa cabang sendiri |

---

## 4. Requirement Detail

### 4.1. Database Changes

#### A. Opsi 1: Tambah Role Baru `Owner Cabang` (Rekomendasi ⭐)

```
Roles Enum:
  SUPER_ADMIN     = 'Super Admin'
  OWNER           = 'Owner'           ← Rename jadi "Owner Pusat" di display
  OWNER_CABANG    = 'Owner Cabang'    ← BARU
  MANAGER         = 'Manager'
  SPV             = 'Spv'
  ADMIN           = 'Admin'
  STAFF_ANTAR     = 'Staff Pengantaran'
  STAFF_PRODUKSI  = 'Staff Produksi'
```

**Kelebihan**:
- Pemisahan permission yang jelas
- Tidak perlu ubah logic Owner yang sudah ada
- Mudah di-extend untuk fitur franchise di masa depan

**Kekurangan**:
- Perlu update RoleSeeder, PermissionSeeder
- Middleware perlu handle role baru

#### B. Opsi 2: Tetap Pakai Role `Owner` + Multi-Company Assignment

```
Tabel baru: owner_company_assignments
├── id
├── user_id        → FK users (yang role-nya Owner)
├── company_id     → FK res_company (cabang yang dimiliki)
├── ownership_type → 'pusat' | 'cabang'
├── is_primary     → boolean (cabang utama owner ini)
├── created_at
└── updated_at
```

**Kelebihan**:
- Tidak perlu role baru
- Satu owner bisa punya banyak cabang
- Fleksibel untuk kasus Pak Ahmad punya 3 cabang

**Kekurangan**:
- Permission sama untuk Owner Pusat dan Owner Cabang (harus pakai logic tambahan)
- Filtering data lebih kompleks

#### C. Opsi 3 (Hybrid - DIREKOMENDASIKAN) ⭐⭐

Kombinasi kedua opsi: **Role baru `Owner Cabang`** + **Tabel assignment multi-company**

```sql
-- Tabel baru: company_ownership
CREATE TABLE company_ownerships (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,           -- FK → users
    company_id BIGINT NOT NULL,        -- FK → res_company
    ownership_type ENUM('franchisor', 'franchisee') DEFAULT 'franchisee',
    -- 'franchisor' = Owner Pusat (lihat semua)
    -- 'franchisee' = Owner Cabang (lihat yang dimiliki)
    
    share_percentage DECIMAL(5,2) DEFAULT 100.00,  -- % kepemilikan
    start_date DATE NOT NULL,
    end_date DATE NULL,                -- NULL = masih aktif
    status ENUM('active', 'suspended', 'terminated') DEFAULT 'active',
    
    -- Informasi kontrak
    contract_number VARCHAR(50) NULL,
    franchise_fee DECIMAL(15,2) NULL,          -- Biaya franchise
    royalty_percentage DECIMAL(5,2) NULL,       -- % royalty ke pusat
    
    notes TEXT NULL,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    
    UNIQUE KEY (user_id, company_id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (company_id) REFERENCES res_company(id)
);
```

**Alur Logika**:
```
User login sebagai "Owner Cabang"
  → Cek company_ownerships WHERE user_id = X AND status = 'active'
  → Dapat list company_id [2, 5] (Gatsu & Cipto)
  → Dashboard menampilkan data dari company_id IN (2, 5)
  → Setoran masuk dari company_id 2 dan 5

User login sebagai "Owner" (Pusat)
  → ownership_type = 'franchisor'
  → Lihat SEMUA company (tidak di-filter)
  → Atau cek company_ownerships jika ingin scope tertentu
```

### 4.2. Permission Owner Cabang

```
┌─────────────────────────────────────────────────────────────┐
│                 PERMISSION OWNER CABANG                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ✅ READ (Cabang sendiri saja):                             │
│    Dashboard, Sales, Orders, Purchasing, Inventory,         │
│    Stock, Manufacturing, Distribution, Delivery, HR,        │
│    Employees, Attendance, Finance, CRM, Reports,            │
│    Courier Performance, Fleet, Asset, Daily Closing         │
│                                                             │
│ ✅ WRITE (Terbatas):                                        │
│    - Preferences cabang (delivery surcharge, dll)           │
│    - Diskon & promo lokal                                   │
│    - Jadwal karyawan                                        │
│    - Payment method                                         │
│    - Konfirmasi setoran (OwnerDeposit)                      │
│                                                             │
│ ✅ SPECIAL:                                                 │
│    - Verifikasi canvassing cabang sendiri                   │
│    - Lihat laporan SPV cabang sendiri                       │
│    - Set target KPI kurir cabang sendiri                    │
│    - Approve/reject pengeluaran besar                       │
│                                                             │
│ ❌ TIDAK BOLEH:                                             │
│    - Lihat data cabang lain                                 │
│    - Ubah pengaturan sistem pusat                           │
│    - Kelola user/role (kecuali request ke pusat)            │
│    - Hapus data apapun                                      │
│    - Operasional harian (itu tugas Manager/SPV)             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.3. Update Model ResCompany

```php
// Tambahan field di res_company
Schema::table('res_company', function (Blueprint $table) {
    // Franchise info
    $table->string('branch_type')->default('owned');  
    // 'owned' = milik pusat, 'franchise' = milik franchisee
    
    $table->date('opening_date')->nullable();
    $table->string('franchise_agreement_number')->nullable();
    $table->decimal('monthly_royalty_amount', 15, 2)->nullable();
});
```

### 4.4. Update Model User / CompanyService

```php
// CompanyService - Method baru
class CompanyService
{
    /**
     * Dapatkan list company_id yang bisa diakses user
     * - Super Admin: semua
     * - Owner (Pusat): semua 
     * - Owner Cabang: hanya yang dimiliki (dari company_ownerships)
     * - Manager/SPV/Staff: hanya company_id sendiri
     */
    public function getAccessibleCompanyIds(User $user): array
    {
        if ($user->hasRole('Super Admin') || $user->hasRole('Owner')) {
            return ResCompany::where('active', true)->pluck('id')->toArray();
        }
        
        if ($user->hasRole('Owner Cabang')) {
            return CompanyOwnership::where('user_id', $user->id)
                ->where('status', 'active')
                ->pluck('company_id')
                ->toArray();
        }
        
        // Default: company sendiri
        return [$user->getCompanyId()];
    }
}
```

---

## 5. Fitur-Fitur Owner Cabang

### 5.1. Dashboard Owner Cabang

```
┌──────────────────────────────────────────────────────┐
│  🏪 Dashboard - Wajib Gatsu                    [▼]  │  ← Pilih cabang (jika punya > 1)
├──────────────────────────────────────────────────────┤
│                                                      │
│  📊 Ringkasan Hari Ini                              │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐       │
│  │Revenue │ │Expense │ │Orders  │ │Delivery│       │
│  │Rp 2.5M │ │Rp 800K │ │47      │ │42      │       │
│  └────────┘ └────────┘ └────────┘ └────────┘       │
│                                                      │
│  📈 Grafik Pendapatan (7 hari terakhir)             │
│  ████████████████████████████████                    │
│  ██████████████████                                  │
│  ████████████████████████                            │
│                                                      │
│  👥 Kehadiran Hari Ini                              │
│  ✅ Hadir: 8/10  │  ❌ Tidak Hadir: 2              │
│                                                      │
│  📦 Stok Rendah                                     │
│  ⚠️ Galon Kosong: 5 (min: 20)                      │
│  ⚠️ Aqua 600ml: 12 (min: 50)                       │
│                                                      │
│  💰 Setoran Terakhir                                │
│  [27 Mar] SPV mengirim Rp 1.8M - ⏳ Menunggu       │
│  [26 Mar] SPV mengirim Rp 2.1M - ✅ Dikonfirmasi   │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 5.2. Konfirmasi Setoran (OwnerDeposit)

```
Alur Setoran:

SPV Cabang                      Owner Cabang
    │                                │
    │  1. Tutup kasir harian         │
    │  2. Hitung uang                │
    │  3. Buat setoran               │
    │     (amount, bukti transfer)   │
    │                                │
    │  ─── Submit Setoran ──────►    │
    │                                │  4. Terima notifikasi
    │                                │  5. Cek bukti transfer/cash
    │                                │  6. Cocokkan dengan laporan
    │                                │
    │    ◄── Confirm / Reject ───    │
    │                                │
    │  7. Terima status              │
    │  (jika reject, kirim ulang)    │
    │                                │
```

### 5.3. Laporan Keuangan Cabang

```
┌──────────────────────────────────────────────────────┐
│  💰 Laporan Keuangan - Wajib Gatsu                  │
│  Periode: Maret 2026                                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  PENDAPATAN                                          │
│  ├── Penjualan Galon Refill    Rp 45.000.000        │
│  ├── Penjualan Galon Baru      Rp 12.000.000        │
│  ├── Biaya Antar               Rp  3.200.000        │
│  └── Lainnya                   Rp  1.500.000        │
│  Total Pendapatan              Rp 61.700.000        │
│                                                      │
│  PENGELUARAN                                         │
│  ├── Bahan Baku / Purchase     Rp 22.000.000        │
│  ├── Gaji Karyawan             Rp 15.000.000        │
│  ├── Operasional               Rp  5.000.000        │
│  ├── Royalti ke Pusat (5%)     Rp  3.085.000        │
│  └── Lainnya                   Rp  2.000.000        │
│  Total Pengeluaran             Rp 47.085.000        │
│                                                      │
│  ════════════════════════════════════════            │
│  LABA BERSIH                   Rp 14.615.000        │
│  ════════════════════════════════════════            │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 5.4. Monitoring Karyawan Cabang

```
Fitur:
├── Lihat daftar karyawan di cabang sendiri
├── Lihat kehadiran harian & rekap bulanan
├── Lihat performa kurir (KPI delivery)
├── Set target KPI kurir
├── Lihat jadwal shift
├── Approve/request pengajuan cuti (opsional masa depan)
└── TIDAK bisa hire/fire (request ke pusat/manager)
```

### 5.5. Multi-Cabang Owner (1 Owner > 1 Cabang)

```
Pak Ahmad memiliki 3 cabang:

┌──────────────────────────────────────────────────────┐
│  🏪 Cabang Saya                                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────────┐  ┌─────────────────┐           │
│  │ Wajib Gatsu     │  │ Wajib Cipto     │           │
│  │ Revenue: 61.7M  │  │ Revenue: 45.2M  │           │
│  │ Orders: 1,247   │  │ Orders: 892     │           │
│  │ Staff: 10       │  │ Staff: 8        │           │
│  │ ✅ Sehat        │  │ ⚠️ Stok rendah  │           │
│  │ [Buka Dashboard]│  │ [Buka Dashboard]│           │
│  └─────────────────┘  └─────────────────┘           │
│                                                      │
│  ┌─────────────────┐                                │
│  │ Wajib Proliman  │  📊 Perbandingan Cabang        │
│  │ Revenue: 38.1M  │  Gatsu    ████████████ 42.5%   │
│  │ Orders: 701     │  Cipto    █████████ 31.1%      │
│  │ Staff: 7        │  Proliman ███████ 26.3%        │
│  │ ✅ Sehat        │                                │
│  │ [Buka Dashboard]│                                │
│  └─────────────────┘                                │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 6. Perbedaan Tiap Role (Matrix Lengkap)

### 6.1. Matrix Akses

| Fitur | Super Admin | Owner Pusat | Owner Cabang | Manager | SPV |
|-------|:-----------:|:-----------:|:------------:|:-------:|:---:|
| Lihat SEMUA cabang | ✅ | ✅ | ❌ | ❌ | ❌ |
| Lihat cabang sendiri | ✅ | ✅ | ✅ | ✅ | ✅ |
| Dashboard agregasi | ✅ | ✅ | ✅ (miliknya) | ❌ | ❌ |
| Buka/tutup cabang | ✅ | ✅ | ❌ | ❌ | ❌ |
| Konfirmasi setoran | ✅ | ✅ | ✅ (miliknya) | ❌ | ❌ |
| Submit setoran | ❌ | ❌ | ❌ | ✅ | ✅ |
| Kelola karyawan | ✅ | ✅ | 👁️ Read | ✅ | ❌ |
| Atur diskon cabang | ✅ | ✅ | ✅ (miliknya) | ✅ | ❌ |
| Operasi harian | ✅ | ❌ | ❌ | ✅ | ✅ |
| Hapus data | ✅ | ❌ | ❌ | Terbatas | ❌ |
| Kelola role/user | ✅ | ❌ | ❌ | ❌ | ❌ |
| Setting sistem | ✅ | Terbatas | ❌ | ❌ | ❌ |
| Verifikasi canvassing | ✅ | ✅ | ✅ (miliknya) | ❌ | ✅ |
| Laporan keuangan | ✅ | ✅ (semua) | ✅ (miliknya) | ✅ | 👁️ |
| Set royalti/fee | ✅ | ✅ | ❌ | ❌ | ❌ |

### 6.2. Data Scope per Role

```
Super Admin  → WHERE active = true (semua company aktif)
Owner Pusat  → WHERE active = true (semua company aktif)
Owner Cabang → WHERE company_id IN (company_ownerships.company_id WHERE user_id = X)
Manager      → WHERE company_id = user.company_id (1 cabang)
SPV          → WHERE company_id = user.company_id (1 cabang)
Admin        → WHERE company_id = user.company_id (1 cabang)
Staff        → WHERE company_id = user.company_id (1 cabang)
```

---

## 7. Alur Bisnis Franchise

### 7.1. Onboarding Cabang Baru

```
1. CALON FRANCHISEE
   │
   │  Hubungi Wajib Pusat
   │  Ajukan lokasi cabang
   │  Bayar franchise fee
   │
   ▼
2. PUSAT (Super Admin / Owner Pusat)
   │
   │  Verifikasi lokasi & kelayakan
   │  Buat kontrak franchise
   │  Buat record res_company baru
   │  Buat user Owner Cabang
   │  Assign company_ownership
   │
   ▼
3. SETUP CABANG
   │
   │  Setup stock location (gudang)
   │  Setup delivery zone
   │  Setup product & harga
   │  Recruit & assign karyawan
   │  Setup fleet (kendaraan)
   │
   ▼
4. OPERASIONAL
   │
   │  Owner Cabang login → lihat dashboard
   │  Manager jalankan operasional
   │  SPV monitoring & submit setoran
   │  Owner Cabang konfirmasi setoran
   │
   ▼
5. ONGOING
   │
   │  Royalti bulanan otomatis terhitung
   │  Performa di-monitor pusat
   │  Owner Cabang terima laporan rutin
```

### 7.2. Alur Harian Owner Cabang

```
🌅 PAGI (5 menit)
│
├── Login ke aplikasi
├── Cek notifikasi
│   ├── 🔔 "Setoran Rp 1.8M dari SPV Gatsu menunggu konfirmasi"
│   ├── ⚠️ "Stok galon kosong rendah di Cipto"
│   └── 📊 "Laporan harian kemarin sudah siap"
├── Quick glance dashboard: revenue today, orders, attendance
└── Selesai

🌤️ SIANG (opsional, 2-3 menit)
│
├── Cek setoran masuk
├── Konfirmasi/reject setoran
└── Cek stok jika ada alert

🌙 MALAM (10-15 menit)
│
├── Buka dashboard lengkap
├── Review pendapatan hari ini vs target
├── Cek performa kurir
├── Review laporan harian dari SPV
├── Bandingkan performa antar cabang (jika punya > 1)
└── Konfirmasi setoran yang belum di-approve

📆 MINGGUAN (15-20 menit)
│
├── Review tren 7 hari
├── Top/bottom produk
├── Performa karyawan
├── Rencana stok minggu depan
└── Evaluasi delivery zone

📊 BULANAN (30 menit)
│
├── Laporan P&L bulanan
├── Evaluasi biaya operasional
├── Review royalti ke pusat
├── Rencana ekspansi / perbaikan
└── Meeting dengan Manager (opsional)
```

### 7.3. Alur Setoran Detail

```
                    Manager/SPV                 Owner Cabang              Owner Pusat
                        │                           │                        │
  Tutup kasir ──────────┤                           │                        │
  Hitung uang           │                           │                        │
  Buat OwnerDeposit     │                           │                        │
  Upload bukti          │                           │                        │
                        │                           │                        │
  Submit ──────────────►│ Terima notifikasi ────────┤                        │
                        │ Cek amount vs daily close │                        │
                        │                           │                        │
                        │ ┌── Match? ──┐            │                        │
                        │ │            │            │                        │
                        │ YES          NO           │                        │
                        │ │            │            │                        │
                        │ Confirm      Reject       │                        │
                        │ │            │            │                        │
  Terima status ◄───────┤ │            │Kirim       │                        │
  "Dikonfirmasi"        │ │            │alasan      │                        │
                        │ │            │            │                        │
                        │ │       Revisi & ◄────────┤                        │
                        │ │       Submit ulang      │                        │
                        │ │                         │                        │
                        │ └──► Tercatat di ─────────┼──► Visible di          │
                        │      laporan cabang       │    laporan pusat       │
                        │                           │                        │
```

---

## 8. Implementasi Teknis (Tahapan)

### Phase 1: Database & Infrastruktur (Foundation)

**Estimasi Scope**: Core foundation

| # | Task | Detail |
|---|------|--------|
| 1.1 | Migration: `company_ownerships` | Tabel relasi owner ↔ company |
| 1.2 | Model: `CompanyOwnership` | Eloquent model + relationships |
| 1.3 | Role: Tambah `Owner Cabang` | Update RoleSeeder & PermissionSeeder |
| 1.4 | Update `CompanyService` | Method `getAccessibleCompanyIds()` |
| 1.5 | Middleware update | Support multi-company scope untuk Owner Cabang |
| 1.6 | Seeder: Owner Cabang sample | Buat sample data 2-3 owner cabang |

**Detail Migration**:
```php
// database/migrations/xxxx_create_company_ownerships_table.php

Schema::create('company_ownerships', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained('users');
    $table->foreignId('company_id')->constrained('res_company');
    $table->enum('ownership_type', ['franchisor', 'franchisee'])->default('franchisee');
    $table->decimal('share_percentage', 5, 2)->default(100.00);
    $table->date('start_date');
    $table->date('end_date')->nullable();
    $table->enum('status', ['active', 'suspended', 'terminated'])->default('active');
    $table->string('contract_number', 50)->nullable();
    $table->decimal('franchise_fee', 15, 2)->nullable();
    $table->decimal('royalty_percentage', 5, 2)->nullable();
    $table->text('notes')->nullable();
    $table->timestamps();

    $table->unique(['user_id', 'company_id']);
});
```

### Phase 2: Permission & Role Setup

| # | Task | Detail |
|---|------|--------|
| 2.1 | Define permission set Owner Cabang | Copy dari Owner, tambah/kurangi sesuai matrix |
| 2.2 | Update PermissionSeeder | Assign permission ke role Owner Cabang |
| 2.3 | Update gate/policy checks | Cek role Owner Cabang di authorization logic |
| 2.4 | Update sidebar/menu | Tampilkan menu sesuai role Owner Cabang |

### Phase 3: Dashboard Owner Cabang

| # | Task | Detail |
|---|------|--------|
| 3.1 | Company selector component | Dropdown untuk pilih cabang (jika punya > 1) |
| 3.2 | Dashboard page Owner Cabang | KPI cards, grafik, ringkasan |
| 3.3 | Multi-company aggregation | Query yang bisa handle array company_id |
| 3.4 | Perbandingan cabang | Side-by-side comparison (untuk multi-cabang owner) |

### Phase 4: Setoran & Keuangan

| # | Task | Detail |
|---|------|--------|
| 4.1 | Update OwnerDeposit flow | Filter berdasarkan company_ownerships |
| 4.2 | Notifikasi setoran | Push notification ke Owner Cabang yang relevan |
| 4.3 | Laporan keuangan per cabang | P&L, cash flow, filtered by company_id |
| 4.4 | Royalti tracking | Hitung & tampilkan royalti ke pusat |

### Phase 5: Management & Monitoring

| # | Task | Detail |
|---|------|--------|
| 5.1 | Employee viewer | Lihat karyawan di cabang sendiri |
| 5.2 | Attendance monitoring | Rekap kehadiran per cabang |
| 5.3 | Courier KPI | Performa kurir per cabang |
| 5.4 | Stock alert | Notifikasi stok rendah per cabang |
| 5.5 | Daily closing viewer | Lihat laporan tutup kasir harian |

### Phase 6: Admin Pusat - Franchise Management

| # | Task | Detail |
|---|------|--------|
| 6.1 | CRUD Owner Cabang | Super Admin bisa add/edit/deactivate owner cabang |
| 6.2 | Assign cabang ke owner | UI untuk mapping owner ↔ company |
| 6.3 | Franchise dashboard pusat | Overview semua cabang + owner |
| 6.4 | Royalti report pusat | Rekap royalti dari semua cabang |

---

## 9. Contoh Data Seeder

```php
// database/seeders/Company/GatsuSeeder.php (updated)

// Owner Cabang Gatsu
$ownerGatsu = User::firstOrCreate(['username' => 'owner_gatsu'], [
    'name' => 'Pak Ahmad (Owner Gatsu)',
    'email' => 'ahmad@wajibairminum.com',
    'password' => Hash::make('password'),
    'company_id' => $companyGatsu->id,
]);
$ownerGatsu->assignRole('Owner Cabang');

// Assign ownership
CompanyOwnership::firstOrCreate([
    'user_id' => $ownerGatsu->id,
    'company_id' => $companyGatsu->id,
], [
    'ownership_type' => 'franchisee',
    'share_percentage' => 100.00,
    'start_date' => '2026-01-01',
    'status' => 'active',
    'franchise_fee' => 50000000,    // 50 juta
    'royalty_percentage' => 5.00,   // 5%
]);

// Owner multi-cabang (contoh: Ahmad juga punya Cipto)
CompanyOwnership::firstOrCreate([
    'user_id' => $ownerGatsu->id,
    'company_id' => $companyCipto->id,
], [
    'ownership_type' => 'franchisee',
    'share_percentage' => 100.00,
    'start_date' => '2026-03-01',
    'status' => 'active',
    'franchise_fee' => 50000000,
    'royalty_percentage' => 5.00,
]);
```

---

## 10. Pertimbangan Teknis

### 10.1. Query Performance

```php
// SEBELUM (1 company)
SalesOrder::where('company_id', $companyId)->sum('amount_total');

// SESUDAH (multi company - Owner Cabang)
SalesOrder::whereIn('company_id', $accessibleCompanyIds)->sum('amount_total');

// Index yang perlu ditambah
$table->index(['company_id', 'state']);           // Di semua model ber-company_id
$table->index(['user_id', 'status']);             // Di company_ownerships
```

### 10.2. Caching Strategy

```
Cache per company per owner:
- dashboard_stats_{user_id}_{company_id} → 5 min TTL
- company_list_{user_id} → 30 min TTL (jarang berubah)
- deposit_pending_{user_id} → 1 min TTL (realtime-ish)
```

### 10.3. Security Considerations

```
1. AUTHORIZATION: Setiap request WAJIB cek company_ownerships
   → Owner Cabang tidak bisa akses data cabang yang bukan miliknya
   → Middleware: VerifyCompanyAccess

2. DATA ISOLATION: Owner Cabang A tidak bisa lihat data Owner Cabang B
   → Query selalu filter by accessible company_ids
   → API endpoint validate company_id against ownership

3. ROLE ESCALATION: Owner Cabang tidak bisa jadikan diri sendiri Owner Pusat
   → Hanya Super Admin yang bisa assign/change ownership_type

4. DEPOSIT FRAUD: Owner Cabang hanya bisa konfirmasi setoran ke cabangnya
   → OwnerDeposit.company_id IN accessible_company_ids

5. AUDIT TRAIL: Semua konfirmasi setoran tercatat dengan user_id dan timestamp
```

---

## 11. Migration Path dari Sistem Sekarang

### Langkah Migrasi

```
STEP 1: Tambah tabel company_ownerships (backward compatible)
         → Sistem lama tetap jalan, tabel baru belum dipakai

STEP 2: Tambah role "Owner Cabang" ke roles table
         → Role lama tidak terpengaruh

STEP 3: Update CompanyService.getAccessibleCompanyIds()
         → Jika user bukan Owner Cabang, behavior sama seperti sebelumnya

STEP 4: Buat user Owner Cabang di seeder
         → Untuk testing & development

STEP 5: Update dashboard queries untuk support whereIn
         → Fallback ke where jika single company_id

STEP 6: Buat UI Dashboard Owner Cabang
         → Route & page baru, tidak mengubah yang ada

STEP 7: Update OwnerDeposit flow
         → Tambah filter company_ownerships, flow lama tetap jalan

STEP 8: Gradual rollout
         → Mulai dari 1 cabang, verifikasi, lalu expand
```

### Backward Compatibility

| Komponen | Impact | Mitigasi |
|----------|--------|----------|
| Owner lama | Tidak terpengaruh | Tetap role `Owner`, tetap lihat data MAIN |
| Manager/SPV | Tidak terpengaruh | company_id scope tidak berubah |
| API endpoint | Perlu update | Tambah company_ownerships check untuk role baru |
| Dashboard | Perlu extend | Tambah mode multi-company, single-company tetap jalan |
| Setoran | Perlu extend | Tambah routing ke owner yang tepat |

---

## 12. Risiko & Mitigasi

| Risiko | Impact | Probabilitas | Mitigasi |
|--------|--------|--------------|----------|
| Data bocor antar cabang | 🔴 High | Medium | Middleware VerifyCompanyAccess di semua endpoint |
| Owner lihat data bukan miliknya | 🔴 High | Low | company_ownerships validation + unit test |
| Setoran dikonfirmasi owner salah | 🟡 Medium | Low | Filter deposit by company_ownership |
| Performance query multi-company | 🟡 Medium | Medium | Index + caching |
| Kompleksitas role bertambah | 🟡 Medium | High | Dokumentasi clear + permission matrix |
| Franchise fee/royalti salah hitung | 🔴 High | Medium | Unit test + monthly reconciliation |

---

## 13. Kesimpulan & Rekomendasi

### Rekomendasi Utama

1. **Gunakan Opsi 3 (Hybrid)**: Role baru `Owner Cabang` + tabel `company_ownerships`
2. **Mulai dari Phase 1-2**: Foundation dulu (database + permission), baru UI
3. **Migrasi gradually**: Backward compatible, tidak break fitur yang ada
4. **1 Owner bisa punya banyak cabang**: Design from day 1 untuk multi-cabang
5. **Security first**: Middleware VerifyCompanyAccess wajib ada sebelum UI

### Prioritas Implementasi

```
🔴 MUST HAVE (Phase 1-3):
├── Role Owner Cabang
├── Tabel company_ownerships
├── CompanyService multi-company support
├── Dashboard Owner Cabang (basic)
└── Company selector (jika multi-cabang)

🟡 SHOULD HAVE (Phase 4):
├── Setoran flow ke Owner Cabang
├── Laporan keuangan per cabang
├── Notifikasi
└── Royalti tracking

🟢 NICE TO HAVE (Phase 5-6):
├── Franchise management (CRUD dari pusat)
├── Advanced analytics & comparison
├── Alert system
└── Kontrak/agreement management
```

---

*Dokumen ini akan di-update seiring development berlangsung.*
