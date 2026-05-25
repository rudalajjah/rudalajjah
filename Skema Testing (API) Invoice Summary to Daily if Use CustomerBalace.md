# Customer Summary : Invoice Customer Balance Menjadi Daily

## Tujuan

Memastikan customer yang setting invoice-nya summary tetap dibuat menjadi invoice daily jika booking memakai customer balance/deposit.

Rule baru:

| Kondisi booking | Hasil invoice |
| --- | --- |
| Customer summary = Y dan tidak ada deposit | Summary |
| Customer summary = Y dan ada deposit/customer balance | Daily |
| Customer summary = N | Daily |

## Data yang Perlu Disiapkan

Siapkan 4 customer untuk testing:

| Data customer | Setting yang dibutuhkan |
| --- | --- |
| Customer A | `summary = Y`, customer balance aktif, COA customer balance sudah disetting |
| Customer B | `summary = Y`, customer balance tidak aktif atau COA customer balance belum disetting |
| Customer C | `summary = N`, customer balance aktif, COA customer balance sudah disetting |
| Customer D | `summary = N`, customer balance tidak aktif|

Siapkan booking untuk product berikut:

- Airline
- Hotel
- Train
- Insurance

Minimal test wajib:

- 1 booking tanpa deposit/customer balance.
- 1 booking dengan deposit/customer balance.

## Cara Membedakan Booking Deposit

Di payload/API, lihat nilai:

```text
Booking.DepositDeducted
```

Patokan:

| Nilai `DepositDeducted` | Arti |
| --- | --- |
| `0` | Tidak pakai customer balance/deposit |
| Lebih dari `0` | Pakai customer balance/deposit |

## Skenario 1 - Customer Summary Tanpa Deposit

Tujuan: memastikan flow summary lama tidak berubah.

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer A |
| Customer summary | Y |
| `DepositDeducted` | 0 |

Langkah test:

1. Ambil sample dari production. Edit sesuai data yang diperlukan. Edit Amount di object `DepositDeducte`.
1. Post booking (Postman) Airline/Hotel/Train/Insurance dengan customer summary.
1. Pastikan nilai `DepositDeducted = 0`.
1. Cek invoice yang terbentuk.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Status posting | Sukses |
| Tipe invoice | Summary |
| Payment type | LG |
| COGS | `IsSummary = Y` |

## Skenario 2 - Customer Summary Dengan Deposit

Tujuan: memastikan booking customer balance tidak lagi masuk invoice summary.

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer A |
| Customer summary | Y |
| Customer balance | Aktif |
| COA customer balance | Sudah disetting |
| `DepositDeducted` | Lebih dari 0 |

Langkah test:

1. Ambil sample dari production. Edit sesuai data yang diperlukan. Edit Amount di object `DepositDeducte`.
1. Post booking Airline/Hotel/Train/Insurance dengan customer summary.
1. Pastikan nilai `DepositDeducted` lebih dari 0.
1. Cek invoice yang terbentuk.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Status posting | Sukses |
| Tipe invoice | Daily |
| Payment type | Deposit |
| Status paid invoice | Paid by deposit / `is_paid = 2` |
| Outstanding invoice | 0|
| COGS | `IsSummary = N` |
| Journal AR | Menggunakan COA customer balance |

Catatan penting:

- Walaupun customer disetting summary, invoice harus tetap daily karena booking memakai deposit/customer balance.
- Jika hasilnya masih summary, berarti enhancement belum berjalan.

## Skenario 3 - Customer Summary Dengan Deposit Tapi Setting Customer Balance Belum Lengkap

Tujuan: memastikan sistem tidak membuat invoice jika customer balance belum disetting.

Data:

| Field | Nilai |
| --- | --- |
| Customer | Customer B |
| Customer summary | Y |
| Customer balance / COA customer balance | Belum lengkap |
| `DepositDeducted` | Lebih dari 0 |

Langkah test:

1. Post booking dengan customer summary.
2. Pastikan nilai `DepositDeducted` lebih dari 0.
3. Cek response API.
4. Pastikan tidak ada invoice baru yang terbentuk.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Status posting | Gagal |
| Pesan error | Mengarah ke setting COA customer balance |
| Invoice | Tidak terbentuk |
| COGS | Tidak terbentuk |
| Journal | Tidak terbentuk |

## Skenario 4 - Customer Non Summary

Tujuan: memastikan customer non-summary tidak terdampak.

Data:

| Field | Nilai |
| --- | --- |
| Customer summary | N |
| `DepositDeducted` | 0 atau lebih dari 0 |
Langkah test:

1. Lakukan testing seperti skenario 1, 2, dan 3


Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Status posting | Sukses |
| Tipe invoice | Daily |
| COGS | `IsSummary = N` |

## Product yang Harus Dicoba

| Product | Minimal skenario |
| --- | --- | --- |
| Airline | Skenario 1, 2, 3 |
| Hotel | Skenario 2 |
| Hotel Per Room Per Night | Skenario 2 |
| Train | Skenario 2 |
| Insurance | Skenario 2 |

## Checklist Setelah Posting Sukses

Gunakan booking code atau invoice number dari hasil posting.

| Checklist | Summary tanpa deposit | Deposit/customer balance | Daily |
| --- | --- | --- | --- |
| Invoice terbentuk | Ya | Ya| Ya |
| Tipe invoice | Summary | Daily| Daily |
| Payment type | LG | Deposit | LG |
| Outstanding | 0 | 0 | Terisi |
| Suspend amount | Terisi | 0 | 0 |
| COGS `IsSummary` | Y | N | N |
| Journal | COA ditangguhkan | COA customer balance | COA AR |
| AccRcv | COA AR | COA customer balance | COA AR |

## Kesimpulan Expected

Enhancement dianggap benar jika:

- Booking customer summary tanpa deposit tetap menjadi invoice summary.
- Booking customer summary dengan deposit/customer balance menjadi invoice daily.
- Booking deposit gagal jika setting customer balance belum lengkap.
- Booking customer summary yang menjadi deposit, tidak bisa search invoice saat settle AR.
- Booking customer daily tetap seperti biasa. Deposit tetap jadi deposit, LG/ AR tetap jadi LG/ AR.
