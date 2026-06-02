# Temporary Invoice Manual from Other , Rentcar dan Package Tour

## Tujuan

Memastikan perubahan flow temporary invoice di menu Other Manual berjalan benar: draft aktif terdeteksi, user bisa memilih draft existing atau membuat draft baru, lock draft mencegah bentrok antar tab/user, detail tersimpan ke invoice temp yang benar, dan invoice final berhasil dibuat.

## Scope

- Menu: Other > Manual Invoice.
- Prefix temporary invoice: `tempOT-`.
- Table utama: `trx_other_invoice_detail`, `trx_invoice_draft_lock`, `trx_other_invoice`, `trx_other_invoice_total`.

## Precondition

- Siapkan 1 customer aktif, supplier aktif, product Other aktif, currency/rate valid.
- Browser bisa dibuka minimal 2 tab. Jika memungkinkan, siapkan 2 user berbeda dalam branch yang sama.
- Pastikan tidak ada data testing lama yang mengganggu, khususnya temp invoice milik user yang sama.

## Data Uji

| Data | Nilai |
|---|---|
| Product | Other |
| Invoice mode | Daily dan Summary jika tersedia |
| Detail count | Minimal 2 detail |
| Amount | NTA, Service Fee, Sell Price valid |
| Prefix temp yang diharapkan | `tempOT-` |

## Skenario 1 - Buka Halaman dan Buat Draft Baru

Steps:
1. Login sebagai user support/QA.
2. Buka Other > Manual Invoice.
3. Jika muncul modal `Active Draft Other`, klik `Start New Draft`.
4. Tambahkan 1 detail invoice, lalu klik save detail.

Expected result:
- Halaman load ke UI V2 tanpa error.
- Draft baru memakai nomor dengan prefix `tempOT-`.
- Detail tersimpan di `trx_other_invoice_detail.invoice_no` sesuai temp invoice yang sedang aktif.
- Data detail tampil kembali di halaman setelah save.

## Skenario 2 - Lanjutkan Draft Aktif

Steps:
1. Dari Skenario 1, tutup tab atau buka ulang menu Other > Manual Invoice.
2. Saat modal `Active Draft Other` muncul, cek draft yang tersedia.
3. Klik `Continue` pada draft sebelumnya.

Expected result:
- Modal menampilkan temp invoice sebelumnya beserta jumlah detail.
- Setelah `Continue`, halaman memuat detail draft yang sama.
- Tidak terbentuk temp invoice baru jika user memilih draft existing.

## Skenario 3 - Start New Draft Saat Ada Draft Lama

Steps:
1. Pastikan user punya minimal 1 active draft.
2. Buka Other > Manual Invoice.
3. Pada modal `Active Draft Other`, klik `Start New Draft`.
4. Tambahkan 1 detail baru.

Expected result:
- Sistem membuat/ memakai draft baru dari nomor temp yang digenerate saat page load.
- Detail baru masuk ke temp invoice baru, bukan draft lama.
- Draft lama tetap muncul sebagai draft aktif saat halaman dibuka ulang.

## Skenario 4 - Lock Draft Antar Tab

Steps:
1. Buka draft yang sama di Tab A dengan klik `Continue`.
2. Buka menu Other > Manual Invoice di Tab B dengan user yang sama.
3. Cek status draft pada modal.
4. Coba pilih draft yang sedang dibuka di Tab A.

Expected result:
- Draft yang sedang dipakai tampil status `In Use`.
- Tombol `Continue` untuk draft tersebut disabled, atau muncul pesan bahwa draft sedang dibuka di tab lain.
- Tab B tetap bisa memilih draft lain atau klik `Start New Draft`.

## Skenario 5 - Renew dan Release Lock

Steps:
1. Buka draft di satu tab.
2. Tunggu minimal 5 menit atau monitor request renew lock jika memungkinkan.
3. Tutup tab, lalu buka ulang menu.
4. Pilih draft yang sama.

Expected result:
- Saat tab masih aktif, lock tetap diperpanjang.
- Setelah tab ditutup, lock dilepas atau expired sesuai TTL.
- Draft bisa dipilih ulang setelah lock dilepas/expired.

## Skenario 6 - Submit Invoice Final

Steps:
1. Lanjutkan draft yang punya minimal 2 detail.
2. Lengkapi header invoice dan perhitungan sampai valid.
3. Klik create/submit invoice.
4. Cek hasil di list invoice dan database.

Expected result:
- Invoice final berhasil terbentuk di `trx_other_invoice` dan `trx_other_invoice_total`.
- Semua detail temp berubah memakai invoice final, bukan `tempOT-`.
- Lock untuk draft tersebut hilang dari `trx_invoice_draft_lock`.
- `sessionStorage` draft dibersihkan, sehingga buka ulang halaman tidak langsung memakai draft final tadi.

## Skenario 8 - Validasi Duplicate Submit

Steps:
1. Setelah invoice final berhasil dibuat, coba submit ulang dari tab lama atau data yang sama jika masih terbuka.
2. Amati response dan tampilan.

Expected result:
- Sistem menolak duplicate invoice.
- Muncul pesan `Already create invoice !`.
- Tidak ada invoice final ganda untuk detail yang sama.

## Checklist Database

| Check | Expected |
|---|---|
| `trx_other_invoice_detail.invoice_no LIKE 'tempOT-%'` setelah save draft | Ada selama belum submit final |
| `trx_invoice_draft_lock` setelah draft dipilih | Ada row sesuai `product_code = OT` dan invoice temp |
| `trx_invoice_draft_lock.expired_at` | Lebih besar dari waktu sekarang |
| Setelah submit final | Lock draft terhapus |
| Setelah submit final | Detail memakai invoice final, bukan `tempOT-` |

## Regression Wajib Lulus

| Area | Expected |
|---|---|
| Add/Edit/Delete detail | Tetap berjalan normal pada draft aktif |
| Upload attachment detail | Tetap memakai temp invoice aktif sebelum submit |
| Preview invoice/journal | Bisa dibuka tanpa error |
| Daily invoice | Invoice final tersimpan benar |
| Summary invoice | Invoice final tersimpan benar jika mode tersedia |
| Browser console | Tidak ada `ReferenceError` atau `TypeError` |
| Network | Tidak ada 4xx/5xx pada endpoint draft, save detail, dan submit invoice |

## Kesimpulan Expected

Flow dianggap lulus jika user support/QA bisa membuat beberapa temporary invoice Other, memilih draft yang benar, tidak terjadi bentrok antar tab/user, dan submit final mengubah draft menjadi invoice asli tanpa duplicate atau data tertukar.
