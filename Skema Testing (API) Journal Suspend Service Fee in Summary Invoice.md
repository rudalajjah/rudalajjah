# QA - Journal Service Fee dan Agent Comm untuk Invoice Summary

## Tujuan Akhir

Memastikan invoice tipe summary tetap membentuk journal service fee dan agent comm jika nilainya ada.

Sebelumnya:

- Invoice summary tidak membentuk journal service fee.
- Invoice summary tidak membentuk journal agent comm.

Sekarang:

- Invoice summary harus membentuk journal service fee dengan COA `CoaServiceFeeDitangguhkan`.
- Invoice summary harus membentuk journal agent comm dengan COA `CoaAgentCommDitangguhkan`.
- Nilai journal sales harus dikurangi service fee dan agent comm supaya total journal tetap balance.

## Area yang Berpengaruh

Field/setting yang berpengaruh:

| Setting/Data | Pengaruh |
| --- | --- |
| Invoice summary | Menentukan penggunaan COA ditangguhkan |
| `service_fee > 0` | Membentuk journal service fee |
| `agent_comm > 0` | Membentuk journal agent comm |
| `ConfigUseTravelServices = Y` | Syarat journal service fee terbentuk |
| `UseSalesComm = Y` | Syarat journal agent comm terbentuk |
| `vat_display_journal = Y` | Syarat service fee dan agent comm ditampilkan di journal |
| `CoaServiceFeeDitangguhkan` | COA journal service fee untuk summary |
| `CoaAgentCommDitangguhkan` | COA journal agent comm untuk summary |

## Persiapan Data

Siapkan customer dan product dengan kondisi berikut:

| Data | Nilai yang dibutuhkan |
| --- | --- |
| Customer | Customer summary |
| Customer `summary` | Y |
| `DepositDeducted` | 0 |
| Invoice type expected | Summary |
| Service fee | Lebih dari 0 |
| Agent comm | Lebih dari 0 |
| Config travel service | Aktif / Y |
| Config sales comm | Aktif / Y |
| VAT display journal | Aktif / Y |
| COA service fee ditangguhkan | Terisi |
| COA agent comm ditangguhkan | Terisi |

Catatan:

- `DepositDeducted` harus 0 supaya invoice tetap summary.
- Jika `DepositDeducted > 0`, invoice menjadi daily dan tidak masuk scope test ini.

## Skenario 1 - Invoice Summary Dengan Service Fee dan Agent Comm

Tujuan: memastikan dua journal baru terbentuk pada invoice summary.

Data:

| Field | Nilai |
| --- | --- |
| Customer summary | Y |
| `DepositDeducted` | 0 |
| Service fee | > 0 |
| Agent comm | > 0 |
| `ConfigUseTravelServices` | Y |
| `UseSalesComm` | Y |
| `vat_display_journal` | Y |

Langkah test:

1. Post booking sampai invoice summary terbentuk.
2. Ambil invoice number hasil posting.
3. Cek journal detail berdasarkan invoice number tersebut.
4. Pastikan journal service fee dan agent comm terbentuk.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Tipe invoice | Summary |
| Journal service fee | Ada |
| COA service fee | `CoaServiceFeeDitangguhkan` |
| Debit/Credit service fee | Credit sebesar nilai service fee |
| Journal agent comm | Ada |
| COA agent comm | `CoaAgentCommDitangguhkan` |
| Debit/Credit agent comm | Credit sebesar nilai agent comm |
| Journal sales | Credit sudah dikurangi service fee dan agent comm |
| Balance journal | Debit = Credit |

## Skenario 2 - Invoice Summary Hanya Service Fee

Tujuan: memastikan journal service fee tetap terbentuk walaupun agent comm tidak ada.

Data:

| Field | Nilai |
| --- | --- |
| Customer summary | Y |
| `DepositDeducted` | 0 |
| Service fee | > 0 |
| Agent comm | 0 |
| `ConfigUseTravelServices` | Y |
| `vat_display_journal` | Y |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Journal service fee | Ada |
| COA service fee | `CoaServiceFeeDitangguhkan` |
| Journal agent comm | Tidak ada |
| Balance journal | Debit = Credit |

## Skenario 3 - Invoice Summary Hanya Agent Comm

Tujuan: memastikan journal agent comm tetap terbentuk walaupun service fee tidak ada.

Data:

| Field | Nilai |
| --- | --- |
| Customer summary | Y |
| `DepositDeducted` | 0 |
| Service fee | 0 |
| Agent comm | > 0 |
| `UseSalesComm` | Y |
| `vat_display_journal` | Y |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Journal service fee | Tidak ada |
| Journal agent comm | Ada |
| COA agent comm | `CoaAgentCommDitangguhkan` |
| Balance journal | Debit = Credit |


## Skenario 4 - Regression Invoice Daily

Tujuan: memastikan invoice daily tetap memakai COA normal, bukan COA ditangguhkan.

Data:

| Field | Nilai |
| --- | --- |
| Customer summary | N, atau summary Y dengan `DepositDeducted > 0` |
| Invoice type expected | Daily |
| Service fee | > 0 |
| Agent comm | > 0 |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Tipe invoice | Daily |
| COA service fee | `CoaServiceFee` |
| COA agent comm | `CoaAgentComm` |
| Tidak memakai | `CoaServiceFeeDitangguhkan`, `CoaAgentCommDitangguhkan` |
| Balance journal | Debit = Credit |

## Checklist Validasi Journal

Gunakan invoice number hasil posting.

| Journal | DBCR | Amount | COA untuk summary |
| --- | --- | ---: | --- |
| AR/piutang | DB | Sell price | COA piutang ditangguhkan |
| Sales | CR | Sell price - MDR - stamp duty - VAT - service fee - agent comm | COA pendapatan ditangguhkan |
| Service fee | CR | Service fee | `CoaServiceFeeDitangguhkan` |
| Agent comm | CR | Agent comm | `CoaAgentCommDitangguhkan` |
| VAT | CR | VAT | COA pajak ditangguhkan |
| MDR, stamp duty, COGS/AP | Sesuai existing flow | Sesuai existing flow | Sesuai existing flow summary |

## Kriteria Lulus

Task dianggap valid jika:

- Invoice summary dengan service fee membentuk journal service fee.
- Invoice summary dengan agent comm membentuk journal agent comm.
- COA yang dipakai adalah COA ditangguhkan, bukan COA normal.
- Journal sales sudah dikurangi service fee dan agent comm.
- Total journal tetap balance.
- Invoice daily tetap memakai COA normal.
