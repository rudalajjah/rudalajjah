# Skema Testing Support: Journal Suspend Service Fee di Summary Invoice

## Tujuan

Memastikan Summary Invoice yang memiliki Service Fee dan/atau Agent Comm tetap membentuk journal dengan COA ditangguhkan.

Rule yang dicek:

| Kondisi invoice | Hasil yang benar |
| --- | --- |
| Summary Invoice + Service Fee | Journal Service Fee terbentuk memakai COA Service Fee Ditangguhkan |
| Summary Invoice + Agent Comm | Journal Agent Comm terbentuk memakai COA Agent Comm Ditangguhkan |
| Daily Invoice + Service Fee / Agent Comm | Tetap memakai COA normal, bukan COA ditangguhkan |

## Module yang Berpengaruh

Testing dilakukan pada module invoice yang bisa membentuk Summary Invoice dan memiliki Service Fee / Agent Comm:

| Module | Fokus testing |
| --- | --- |
| Airline | Summary Invoice dengan Service Fee dan Agent Comm |
| Document | Summary Invoice dengan Service Fee dan Agent Comm |
| Hotel | Summary Invoice dengan Service Fee dan Agent Comm |
| Insurance | Summary Invoice dengan Service Fee dan Agent Comm |
| Kereta | Summary Invoice dengan Service Fee dan Agent Comm |
| Other | Summary Invoice dengan Service Fee dan Agent Comm |
| Rentcar | Summary Invoice dengan Service Fee dan Agent Comm |

Catatan: jika salah satu module tidak memiliki field Service Fee atau Agent Comm di environment test, catat sebagai "Not Available" dan lanjut ke module berikutnya.

## Task yang Dikerjakan Support

| No | Task | Output |
| --- | --- | --- |
| 1 | Siapkan customer Summary | Customer dengan Summary = Yes |
| 2 | Siapkan transaksi/detail dengan Service Fee dan/atau Agent Comm | Nilai Service Fee / Agent Comm lebih dari 0 |
| 3 | Create Summary Invoice | Invoice berhasil terbentuk sebagai Summary |
| 4 | Cek invoice hasil create | Nomor invoice, tipe invoice, nilai Service Fee / Agent Comm |
| 5 | Cek journal | Journal Service Fee / Agent Comm memakai COA ditangguhkan |
| 6 | Cek balance journal | Total debit = total credit |
| 7 | Regression Daily Invoice | Daily tetap memakai COA normal |

## Data yang Perlu Disiapkan

Siapkan data berikut sebelum testing:

| Data | Setting yang dibutuhkan |
| --- | --- |
| Customer A | Summary = Yes |
| Customer B | Summary = No |
| Product/setting module | Service Fee aktif jika ada settingnya |
| Product/setting module | Agent Comm aktif jika ada settingnya |
| COA Service Fee Ditangguhkan | Sudah terisi |
| COA Agent Comm Ditangguhkan | Sudah terisi |
| VAT display journal | Aktif jika setting tersedia |

Pastikan juga invoice summary tidak memakai Customer Balance/deposit. Jika memakai Customer Balance, invoice bisa menjadi Daily dan tidak masuk scope utama testing ini.

## Flow Utama: Summary Invoice dengan Service Fee dan Agent Comm

Gunakan flow ini untuk setiap module yang dites.

### 1. Pilih Customer Summary

1. Buka menu invoice module yang akan dites.
2. Pilih Customer A.
3. Pastikan Customer A adalah customer Summary.
4. Jangan aktifkan Customer Balance/deposit.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Customer | Customer A |
| Customer Summary | Yes |
| Customer Balance/deposit | Tidak dipakai |
| Tipe invoice expected | Summary |

### 2. Add Detail / Pilih Transaksi

Jika module input manual:

1. Klik Add Detail.
2. Isi detail product/transaksi seperti biasa.
3. Isi nilai Service Fee lebih dari 0.
4. Isi nilai Agent Comm lebih dari 0 jika field tersedia.
5. Simpan detail.

Jika module memakai booking/COGS:

1. Search transaksi/booking.
2. Pilih transaksi yang memiliki Service Fee lebih dari 0.
3. Pilih transaksi yang memiliki Agent Comm lebih dari 0 jika tersedia.
4. Pastikan detail masuk ke invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Detail transaksi | Masuk ke invoice |
| Service Fee | Lebih dari 0 |
| Agent Comm | Lebih dari 0 jika tersedia |
| Error validasi | Tidak ada |

### 3. Calculate / Generate Invoice

1. Klik Calculate, Generate, atau tombol sejenis sesuai module.
2. Pastikan invoice tetap Summary.
3. Cek total Service Fee dan Agent Comm pada tampilan invoice jika tersedia.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Tipe invoice | Summary |
| Payment type | LG / bukan deposit |
| Service Fee | Ada nilai |
| Agent Comm | Ada nilai jika tersedia |

### 4. Create Invoice

1. Klik Create Invoice / Save.
2. Pastikan proses berhasil.
3. Catat nomor invoice yang terbentuk.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Status create | Sukses |
| Nomor invoice | Terbentuk |
| Tipe invoice | Summary Invoice |

## Cek Setelah Create Invoice

Gunakan nomor invoice dari hasil create.

### 1. Cek List Invoice

1. Buka list invoice module terkait.
2. Search nomor invoice.
3. Buka detail invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice ditemukan | Ya |
| Tipe invoice | Summary |
| Service Fee | Sesuai input/transaksi |
| Agent Comm | Sesuai input/transaksi jika tersedia |

### 2. Cek Summary Invoice

1. Buka menu/list Summary Invoice jika tersedia.
2. Search customer, periode, atau nomor summary.
3. Pastikan invoice masuk ke Summary Invoice.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice masuk Summary | Ya |
| Nomor summary | Terbentuk / terhubung |

### 3. Cek Journal

Jika support punya akses journal:

1. Buka journal dari invoice tersebut.
2. Search nomor invoice.
3. Cek baris journal Service Fee dan Agent Comm.

Jika support tidak punya akses journal:

1. Catat nomor invoice.
2. Minta tim accounting/dev cek journal berdasarkan nomor invoice.
3. Isi hasilnya di checklist.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Journal invoice | Terbentuk |
| Journal Service Fee | Ada jika Service Fee > 0 |
| COA Service Fee | COA Service Fee Ditangguhkan |
| Journal Agent Comm | Ada jika Agent Comm > 0 |
| COA Agent Comm | COA Agent Comm Ditangguhkan |
| Journal Sales | Nilainya sudah dikurangi Service Fee dan Agent Comm |
| Balance journal | Total debit = total credit |

Catatan untuk support: yang paling penting dicek adalah nama/nomor COA journal Service Fee dan Agent Comm harus memakai COA "ditangguhkan", bukan COA normal.

## Skenario Minimal yang Wajib Dites

### Skenario 1 - Summary dengan Service Fee dan Agent Comm

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer A |
| Customer Summary | Yes |
| Customer Balance/deposit | Tidak |
| Service Fee | Lebih dari 0 |
| Agent Comm | Lebih dari 0 |

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice | Summary |
| Journal Service Fee | Ada, COA ditangguhkan |
| Journal Agent Comm | Ada, COA ditangguhkan |
| Journal balance | Debit = Credit |

### Skenario 2 - Summary hanya Service Fee

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer A |
| Customer Summary | Yes |
| Service Fee | Lebih dari 0 |
| Agent Comm | 0 / tidak ada |

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice | Summary |
| Journal Service Fee | Ada, COA Service Fee Ditangguhkan |
| Journal Agent Comm | Tidak wajib ada |
| Journal balance | Debit = Credit |

### Skenario 3 - Summary hanya Agent Comm

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer A |
| Customer Summary | Yes |
| Service Fee | 0 / tidak ada |
| Agent Comm | Lebih dari 0 |

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice | Summary |
| Journal Service Fee | Tidak wajib ada |
| Journal Agent Comm | Ada, COA Agent Comm Ditangguhkan |
| Journal balance | Debit = Credit |

### Skenario 4 - Regression Daily Invoice

Tujuan: memastikan Daily Invoice tidak ikut memakai COA ditangguhkan.

Langkah:

1. Pilih Customer B atau customer yang Summary = No.
2. Buat invoice dengan Service Fee dan/atau Agent Comm.
3. Create invoice sampai sukses.
4. Cek journal.

Expected:

| Yang dicek | Hasil yang benar |
| --- | --- |
| Invoice | Daily |
| COA Service Fee | COA normal, bukan COA Service Fee Ditangguhkan |
| COA Agent Comm | COA normal, bukan COA Agent Comm Ditangguhkan |
| Journal balance | Debit = Credit |

## Checklist Evidence Support

Isi checklist ini untuk setiap module yang dites.

| Module | Skenario | Invoice no | Summary/Daily | Service Fee > 0 | Agent Comm > 0 | COA Journal Benar | Balance | Result | Catatan |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Airline | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Document | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Hotel | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Insurance | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Kereta | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Other | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |
| Rentcar | Summary SF + Agent Comm |  | Summary | Ya/Tidak | Ya/Tidak | Ya/Tidak | Ya/Tidak | Pass/Fail |  |

## Kriteria Pass

Testing dianggap Pass jika:

- Summary Invoice berhasil dibuat.
- Invoice Summary dengan Service Fee membentuk journal Service Fee.
- Invoice Summary dengan Agent Comm membentuk journal Agent Comm.
- Journal Service Fee memakai COA Service Fee Ditangguhkan.
- Journal Agent Comm memakai COA Agent Comm Ditangguhkan.
- Journal Sales sudah dikurangi nilai Service Fee dan Agent Comm.
- Total debit dan credit journal balance.
- Daily Invoice tetap memakai COA normal, bukan COA ditangguhkan.

## Catatan Jika Fail

Catat sebagai fail jika menemukan salah satu kondisi berikut:

| Kondisi fail | Catatan yang perlu ditulis |
| --- | --- |
| Invoice tidak menjadi Summary | Tulis customer, module, dan nomor transaksi |
| Service Fee ada tapi journal Service Fee tidak ada | Tulis invoice no dan nilai Service Fee |
| Agent Comm ada tapi journal Agent Comm tidak ada | Tulis invoice no dan nilai Agent Comm |
| COA memakai COA normal di Summary Invoice | Tulis COA yang muncul di journal |
| Journal tidak balance | Tulis total debit dan total credit |
| Daily Invoice memakai COA ditangguhkan | Tulis invoice no dan module |
