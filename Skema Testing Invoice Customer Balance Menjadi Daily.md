# Skema Testing : Customer Balance Menjadi Daily

## Tujuan

Memastikan invoice yang memakai Customer Balance selalu menjadi Daily Invoice, walaupun customer disetting Summary.

Rule yang dicek:

| Kondisi | Hasil yang benar |
| --- | --- |
| Customer Summary, tidak pakai Customer Balance | Summary Invoice |
| Customer Summary, pakai Customer Balance | Daily Invoice |
| Customer Non Summary | Daily Invoice |

## Module yang Perlu Dicoba

Lakukan minimal 1 kali testing Customer Balance pada masing-masing module:

| Module | Menu yang dites |
| --- | --- |
| Airline | `fend_airline`,  `fend_inv_opsigo`|
| Document | `fend_document_new` |
| Hotel | `fend_hotel` |
| Insurance | `fend_insurance` |
| Kereta | `fend_kereta` |
| Other | `fend_other` |
| Rentcar | `fend_rentcar` |

## Data Customer yang Dibutuhkan

Siapkan 3 customer:

| Customer | Setting | Dipakai untuk |
| --- | --- | --- |
| Customer A | Summary = Yes, Customer Balance = Yes, COA Customer Balance sudah ada | Test utama |
| Customer B | Summary = Yes, Customer Balance belum lengkap | Test error |
| Customer C | Summary = No | Test pembanding daily biasa |

Cara cek customer:

1. Buka menu Master Customer.
2. Cari customer yang akan dipakai.
3. Pastikan setting Summary sesuai kebutuhan.
4. Pastikan Customer Balance aktif jika ingin test Customer Balance.
5. Pastikan COA Customer Balance sudah terisi untuk Customer A.

## Flow Utama: Customer Summary Pakai Customer Balance

Gunakan flow ini untuk setiap module.

### 1. Pilih Customer

1. Buka menu invoice module yang akan dites.
2. Pilih Customer A.
3. Pastikan customer tersebut adalah customer Summary.
4. Jika ada pilihan Customer Balance, pilih Yes.
5. Jika invoice dibuat dari transaksi/booking, pilih transaksi yang payment type-nya deposit atau memakai Customer Balance.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Customer | Customer A |
| Customer summary | Yes |
| Customer balance | Yes / aktif |

### 2. Add Detail

Jika module perlu input detail manual:

1. Klik Add Detail.
2. Isi data product/transaksi seperti biasa.
3. Isi amount yang cukup untuk membentuk invoice.
4. Simpan detail.

Jika module memakai booking/COGS yang sudah ada:

1. Search transaksi/booking.
2. Pilih minimal 1 detail.
3. Pastikan detail masuk ke list invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Detail transaksi | Masuk ke invoice |
| Amount | Terisi |
| Error validasi | Tidak ada |

### 3. Create Invoice

1. Klik Create Invoice / Save.
2. Pastikan proses berhasil.
3. Catat nomor invoice yang terbentuk.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Status create | Sukses |
| Nomor invoice | Terbentuk |
| Tipe invoice | Daily Invoice |
| Payment type | Deposit |
| Outstanding | 0 |

## Cek Setelah Create Invoice

Lakukan pengecekan berikut memakai nomor invoice yang baru dibuat.

### 1. Cek List Invoice

1. Buka list invoice module terkait.
2. Search nomor invoice.
3. Buka detail invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice ditemukan | Ya |
| Invoice masuk Daily | Ya |
| Invoice masuk Summary | Tidak |
| Outstanding | 0 |
| Status paid | Paid by deposit / Customer Balance |

### 2. Cek Summary Invoice

1. Buka menu/list Summary Invoice jika tersedia.
2. Search customer atau periode invoice.
3. Pastikan invoice Customer Balance tadi tidak masuk summary.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice Customer Balance muncul di Summary | Tidak |
| Invoice tetap sebagai Daily | Ya |

### 3. Cek Settle AR

1. Buka menu settle AR / receipt voucher / pembayaran AR yang biasa dipakai untuk invoice LG.
2. Search customer dan nomor invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice Customer Balance muncul sebagai AR biasa | Tidak |
| Outstanding | 0 |
| Perlu settle AR manual | Tidak |

Catatan: invoice Customer Balance sudah dianggap paid by deposit, jadi tidak boleh perlu disettle seperti invoice LG biasa.

### 4. Cek Journal

1. Buka detail journal dari invoice, atau minta tim accounting/dev cek journal jika support tidak punya akses.
2. Pastikan journal AR memakai COA Customer Balance.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Journal terbentuk | Ya |
| COA AR | COA Customer Balance |
| COA suspend summary | Tidak dipakai untuk invoice Customer Balance |

## Flow Pembanding: Customer Summary Tanpa Customer Balance

Tujuan: memastikan flow Summary lama tidak berubah.

Langkah:

1. Buka module yang dites.
2. Pilih Customer A.
3. Jangan aktifkan Customer Balance.
4. Add detail atau pilih transaksi seperti biasa. ( lakukan langkah seperti Flow Utama, hanya berbeda customer saja)
5. Create invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Status create | Sukses |
| Tipe invoice | Summary Invoice |
| Payment type | LG |
| Invoice masuk summary | Ya |

## Flow Error: Customer Balance Belum Lengkap

Tujuan: memastikan sistem menolak invoice jika Customer Balance belum disetting lengkap.

Langkah:

1. Buka module yang dites.
2. Pilih Customer B.
3. Aktifkan Customer Balance atau pilih transaksi deposit.
4. Add detail.
5. Klik Create Invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Status create | Gagal |
| Pesan error | Mengarah ke setting Customer Balance / COA Customer Balance |
| Invoice terbentuk | Tidak |
| Journal terbentuk | Tidak |

## Flow Pembanding: Customer Non Summary

Tujuan: memastikan customer daily biasa tidak terdampak.

Langkah:

1. Buka module yang dites.
2. Pilih Customer C.
4. Add detail atau pilih transaksi seperti biasa. ( lakukan langkah seperti Flow Utama, hanya berbeda customer saja)
3. Buat invoice tanpa Customer Balance.
4. Jika memungkinkan, buat invoice dengan Customer Balance.

Expected:

| Kondisi | Hasil yang benar |
| --- | --- |
| Tanpa Customer Balance | Daily Invoice |
| Dengan Customer Balance | Daily Invoice, payment type Deposit, outstanding 0 |

## Checklist Singkat Support

Isi checklist ini untuk setiap module.

| Module | Customer | Pakai Customer Balance | Invoice no | Hasil benar? | Catatan |
| --- | --- | --- | --- | --- | --- |
| Airline | Customer A | Ya |  | Pass/Fail |  |
| Document | Customer A | Ya |  | Pass/Fail |  |
| Hotel | Customer A | Ya |  | Pass/Fail |  |
| Insurance | Customer A | Ya |  | Pass/Fail |  |
| Kereta | Customer A | Ya |  | Pass/Fail |  |
| Other | Customer A | Ya |  | Pass/Fail |  |
| Rentcar | Customer A | Ya |  | Pass/Fail |  |

## Kriteria Pass

Testing dianggap Pass jika:

- Customer Summary tanpa Customer Balance tetap menjadi Summary Invoice.
- Customer Summary dengan Customer Balance menjadi Daily Invoice.
- Invoice Customer Balance payment type-nya Deposit.
- Outstanding invoice Customer Balance adalah 0.
- Invoice Customer Balance tidak masuk Summary Invoice.
- Invoice Customer Balance tidak perlu settle AR manual.
- Journal memakai COA Customer Balance.
- Jika setting Customer Balance belum lengkap, invoice gagal dibuat.
