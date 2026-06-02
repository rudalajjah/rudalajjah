# Airline Discount

## Tujuan Akhir

Memastikan field airline discount dari API dan UI Add PNR tersimpan dan terhitung dengan benar pada flow Opsigo airline.

Sebelumnya:

- Belum ada field `AirlineDiscount` pada `opsigo_airline_payment`.
- Belum ada field `airline_discount` pada `trx_ticket_invoice_detail`.
- Perhitungan NTA belum mengurangi airline discount.

Sekarang:

- API `api_repost_new` harus menerima nilai discount dari payload dan menyimpan ke `opsigo_airline_payment.AirlineDiscount`.
- API `api_repost_new` harus menyimpan nilai yang sama ke `trx_ticket_invoice_detail.airline_discount`.
- UI Add PNR harus menerima input `Airline Discount`.
- Perhitungan NTA harus mengurangi airline discount.
- Tampilan invoice detail harus menghitung `other_price` dengan mengurangi `airline_discount`.

## Area yang Berpengaruh

Field/data yang berpengaruh:

| Setting/Data | Pengaruh |
| --- | --- |
| API `AirDetails[].Discount` | Source airline discount dari Opsigo |
| `AirDetails[].AirlineDiscount` | Field internal hasil mapping dari API |
| UI `Airline Discount` | Input manual Add PNR |
| `opsigo_airline_payment.AirlineDiscount` | Penyimpanan discount di payment Opsigo |
| `trx_ticket_invoice_detail.airline_discount` | Penyimpanan discount di detail invoice |
| `other_price` | Harus dikurangi `airline_discount` |

## Rumus yang Divalidasi (wrong calculate NTA)

Rumus NTA dari API dan list Opsigo:

```text
NettToAgent = BaseFare + AddCharge + Tax + IW + Psc - AgentCommission + BaggageAmount - AirlineDiscount
```

Rumus NTA dari UI Add PNR:

```text
NTA = TotalAirfares - AgentCommission - AirlineDiscount
```

## Persiapan Data

Siapkan data Opsigo airline dengan kondisi berikut:

| Data | Nilai yang dibutuhkan |
| --- | --- |
| Product | Airline / Ticket |
| Customer | Customer aktif |
| Supplier | Supplier airline aktif |
| Currency | IDR |
| Flight type | Sesuai product ticket |
| Base fare | Lebih dari 0 |
| Tax / AirTax | Boleh 0 atau lebih dari 0 |
| IW / Psc / Add charge | Boleh 0 atau lebih dari 0 |
| Agent commission | Boleh 0 atau lebih dari 0 |
| Airline discount | Lebih dari 0 untuk positive case |

Catatan:

- Gunakan ticket number unik supaya tidak bentrok dengan validasi duplicate.
- Untuk test multi pax, gunakan airline discount berbeda per pax.
- Untuk regression, siapkan juga data tanpa airline discount.

## Skenario 1 - API Repost Dengan Airline Discount

Tujuan: memastikan discount dari API tersimpan di payment dan detail invoice.

Data:

| Field | Nilai contoh |
| --- | ---: |
| `BaseFare` | 1,000,000 |
| `Tax` / `AirTax` | 100,000 |
| `Iw` | 5,000 |
| `Psc` | 20,000 |
| `AddCharge` | 0 |
| `AgentCommission` | 50,000 |
| `Discount` | 25,000 |
| `NettToAgent` | 1,050,000 |

Langkah test:

1. Post payload ke API `api_repost_new` dengan `AirDetails[].Discount = 25000`.
2. Pastikan response API success.
3. Ambil `TicketNo`, `PnrCode`, dan invoice number hasil posting.
4. Cek data di `opsigo_airline_payment`.
5. Cek data di `trx_ticket_invoice_detail`.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| API response | Success |
| `opsigo_airline_payment.AirlineDiscount` | 25,000 |
| `trx_ticket_invoice_detail.airline_discount` | 25,000 |
| Validasi NTA | Tidak terkena `Wrong Calculate NTA` |
| Data invoice | Berhasil terbentuk |

## Skenario 2 - API Repost Tanpa Airline Discount

Tujuan: memastikan flow lama tetap aman jika payload tidak mengirim discount.

Data:

| Field | Nilai |
| --- | --- |
| `AirDetails[].AirlineDiscount` | Tidak dikirim |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| API response | Success |
| `opsigo_airline_payment.AirlineDiscount` | 0 |
| `trx_ticket_invoice_detail.airline_discount` | 0 |
| Error undefined index | Tidak ada |
| Perhitungan NTA | Sama seperti flow existing tanpa discount |

## Skenario 3 - Validasi Discrepancy NTA

Tujuan: memastikan airline discount ikut mempengaruhi validasi selisih NTA.

Data:

| Field | Nilai |
| --- | --- |
| `UseValidationDiscrepancyOpsigo` | Y |
| `LimitAmountDiscrepancyOpsigo` | Sesuai config environment |
| Airline discount | Lebih dari 0 |

Langkah test:

1. Post payload dengan `NettToAgent` yang sudah mengurangi airline discount.
2. Pastikan payload pertama success.
3. Post payload lain dengan `NettToAgent` yang tidak mengurangi airline discount.
4. Pastikan payload kedua gagal validasi jika selisih melewati limit.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Payload NTA benar | Success |
| Payload NTA salah | Gagal validasi |
| Message invalid | `SemiAuto : Wrong Calculate NTA` |

## Skenario 5 - Add PNR Manual Dengan Airline Discount

Tujuan: memastikan input `Airline Discount` di UI Add PNR tersimpan dan mempengaruhi kalkulasi.

Data:

| Field | Nilai contoh |
| --- | ---: |
| Basic | 1,000,000 |
| IW | 5,000 |
| AirTax | 100,000 |
| Psc | 20,000 |
| Other | 0 |
| Insurance | 0 |
| AgentComm | 50,000 |
| Airline Discount | 25,000 |

Langkah test:

1. Buka menu Opsigo invoice.
2. Klik Add PNR.
3. Isi field passenger dan segment mandatory.
4. Isi field `Airline Discount = 25000`.
5. Pastikan kalkulasi NTA berubah.
6. Save Add PNR.
7. Cek data di `opsigo_airline_payment`.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Field `Airline Discount` | Bisa diisi angka 2 desimal |
| Kalkulasi NTA | `TotalAirfares - AgentCommission - AirlineDiscount` |
| Save Add PNR | Success |
| `opsigo_airline_payment.AirlineDiscount` | Sesuai input UI |

## Skenario 6 - Add PNR Manual Tanpa Airline Discount

Tujuan: memastikan Add PNR tetap berjalan jika airline discount kosong atau 0.

Data:

| Field | Nilai |
| --- | --- |
| `Airline Discount` | Kosong atau 0 |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Save Add PNR | Success |
| `opsigo_airline_payment.AirlineDiscount` | 0 |
| Kalkulasi NTA | Sama seperti flow existing tanpa discount |

## Skenario 7 - View/Edit PNR Dengan Airline Discount

Tujuan: memastikan nilai airline discount tidak hilang saat data dibuka ulang.

Langkah test:

1. Buat PNR dari API atau UI dengan airline discount.
2. Buka ulang PNR tersebut dari list Opsigo.
3. Pastikan field `Airline Discount` terisi.
4. Save ulang tanpa mengubah nilai discount.
5. Cek kembali data payment.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Field `Airline Discount` saat view/edit | Terisi sesuai database |
| Save ulang | Success |
| Nilai discount | Tidak berubah atau hilang |

## Skenario 8 - Create Invoice Dari PNR Opsigo

Tujuan: memastikan airline discount dari payment masuk ke detail invoice.

Langkah test:

1. Pilih PNR Opsigo yang memiliki `AirlineDiscount`.
2. Generate invoice.
3. Ambil invoice number.
4. Cek detail invoice.
5. Cek tampilan invoice detail.

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Generate invoice | Success |
| `trx_ticket_invoice_detail.airline_discount` | Sama dengan `opsigo_airline_payment.AirlineDiscount` |
| `other_price` | Sudah dikurangi `airline_discount` |
| Tampilan invoice detail | Amount other/taxes sesuai rumus |

## Skenario 9 - Multi Pax Dengan Airline Discount Berbeda

Tujuan: memastikan airline discount tersimpan per ticket/pax, tidak tertukar antar pax.

Data:

| Pax | Ticket | Airline discount |
| --- | --- | ---: |
| Pax 1 | Ticket 1 | 25,000 |
| Pax 2 | Ticket 2 | 10,000 |

Expected result:

| Yang dicek | Hasil yang diharapkan |
| --- | --- |
| Payment Ticket 1 | `AirlineDiscount = 25,000` |
| Payment Ticket 2 | `AirlineDiscount = 10,000` |
| Detail invoice Ticket 1 | `airline_discount = 25,000` |
| Detail invoice Ticket 2 | `airline_discount = 10,000` |
| Invoice total | Tetap sesuai kalkulasi per pax |

## Query Validasi

Gunakan ticket number atau invoice number hasil posting.

```sql
SELECT
    OAId,
    PnrId,
    TicketNo,
    BaseFare,
    AddCharge,
    AirTax,
    IW,
    Psc,
    Other,
    Insurance,
    AgentCommission,
    AirlineDiscount,
    NettToAgent,
    (
        NettToAgent - (
            (IFNULL(BaseFare, 0) + IFNULL(AddCharge, 0) + IFNULL(AirTax, 0) + IFNULL(IW, 0) + IFNULL(Psc, 0) + IFNULL(Other, 0) + IFNULL(Insurance, 0))
            - IFNULL(AgentCommission, 0)
            - IFNULL(AirlineDiscount, 0)
        )
    ) AS SelisihNta
FROM opsigo_airline_payment
WHERE TicketNo IN ('TICKET_NO_1', 'TICKET_NO_2');
```

```sql
SELECT
    invoice_no,
    pnr_code,
    pnr_id,
    ticket_no,
    base_fare,
    fare_tax,
    iw,
    pass_service_charge,
    other,
    inflight_insurance,
    airline_discount,
    (
        IFNULL(iw, 0)
        + IFNULL(pass_service_charge, 0)
        + IFNULL(other, 0)
        + IFNULL(inflight_insurance, 0)
        - IFNULL(airline_discount, 0)
    ) AS expected_other_price
FROM trx_ticket_invoice_detail
WHERE invoice_no = 'INV_NO';
```

## Checklist Validasi

| Area | Yang dicek | Expected |
| --- | --- | --- |
| API mapping | `Discount` menjadi `AirlineDiscount` | Sesuai payload, nilai negatif menjadi positif |
| Payment Opsigo | `opsigo_airline_payment.AirlineDiscount` | Terisi sesuai source |
| Detail invoice | `trx_ticket_invoice_detail.airline_discount` | Terisi sesuai payment |
| UI Add PNR | Field `Airline Discount` | Bisa input dan memicu kalkulasi |
| NTA | Selisih NTA | 0 atau dalam limit discrepancy |
| Invoice detail | `other_price` | Dikurangi airline discount |
| Regression | Payload/UI tanpa discount | Tetap success dan nilai 0 |
| Multi pax | Nilai per ticket | Tidak tertukar |

## Kriteria Lulus

Task dianggap valid jika:

- API `api_repost_new` berhasil menyimpan airline discount ke `opsigo_airline_payment`.
- API `api_repost_new` berhasil menyimpan airline discount ke `trx_ticket_invoice_detail`.
- UI Add PNR berhasil menyimpan airline discount dari field `Airline Discount`.
- Perhitungan NTA sudah mengurangi airline discount.
- Validasi discrepancy NTA memakai rumus yang sudah mengurangi airline discount.
- Invoice detail menampilkan `other_price` yang sudah dikurangi airline discount.
- Flow tanpa airline discount tetap berjalan normal dengan nilai 0.
- Flow multi pax menyimpan airline discount sesuai masing-masing ticket.
