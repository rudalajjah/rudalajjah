# Invoice Multi Product - Document Testing

## Test Plan

### Scope
Module yang diuji:

- Invoice New
- Keep Commission
- Invoice Collection
- Source Receipt Status

---

# 1. Pre-Condition

Sebelum testing dimulai:

1. Pilih **Customer** yang akan digunakan untuk testing
2. Pastikan **KeepComm sudah disetting** untuk product:
   - Visa
   - Passport

Expected Result:

- Setting KeepComm untuk customer tersedia
- Customer dapat dipilih pada saat create invoice

---

# 2. Scenario: Create Invoice (Document)

## TC-01 Create Document Receipt

### Steps

1. Masuk ke module **Document Receipt**
2. Klik **Add**
3. Pilih customer
4. Save

### Expected Result

- Receipt berhasil dibuat
- Receipt muncul di list receipt

---

## TC-02 Create Invoice

### Steps

1. Masuk ke module **Invoice New**
2. Klik **Create**
3. Pilih customer
4. Klik **Add**
5. Pilih minimal 2 receipt
6. Klik Save

### Expected Result

- Receipt muncul di **Invoice Detail List**
- Data receipt sesuai dengan yang dipilih

---

## TC-03 Validate Selected Receipt

### Steps

1. Cek receipt yang dicentang

### Expected Result

- Semua receipt muncul di list detail invoice

---

## TC-04 Add Keep Commission

### Steps

1. Klik **Add KeepComm**
2. Pilih keep commission

### Expected Result

- List KeepComm sesuai dengan setting customer

---

## TC-05 Validate Keep Commission Amount

### Steps

1. Save KeepComm
2. Cek field **Keep Comm**

### Expected Result
Sum Amount KeepComm = Amount di field Keep Comm


---

## TC-06 Validate Receipt Cannot Be Selected Again

### Steps

1. Klik **Add**
2. Cari receipt yang sudah dipakai

### Expected Result

- Receipt yang sudah ada di detail **tidak muncul di search result**

---

## TC-07 Validate Footer Calculation

### Field yang dicek

- Sub Total
- VAT
- Rounding
- Grand Total

### Expected Result

Semua perhitungan harus benar.

---

## TC-08 Save Invoice

### Steps

1. Klik **Save Invoice**

### Expected Result

- Invoice berhasil disimpan
- Status invoice = **Created**

---

## TC-09 Validate Invoice Preview

### Steps

1. Masuk ke **Invoice Collection List**
2. Preview invoice

### Expected Result

Data harus sama dengan:

- Detail Receipt
- KeepComm
- Amount
- History

---

## TC-10 Validate Receipt Status

### Steps

1. Masuk ke module **Document Receipt**

### Expected Result

Receipt yang dipakai di invoice memiliki status:
Invoice Status = Created


---

# 3. Scenario: Edit Invoice

## TC-11 Edit Invoice

### Steps

1. Buka invoice yang telah dibuat
2. Klik Edit

### Expected Result

- Invoice berhasil dibuka untuk edit

---

## TC-12 Validate Receipt Search After Edit

### Steps

1. Klik **Add**

### Expected Result

Receipt yang sudah ada di detail tidak muncul lagi.

---

## TC-13 Delete COGS

### Steps

1. Pilih salah satu COGS
2. Delete

### Expected Result

- COGS berhasil dihapus
- Total invoice berubah sesuai

---

## TC-14 Add KeepComm After Edit

### Steps

1. Tambahkan keepcomm pada COGS

### Expected Result

- KeepComm berhasil ditambahkan
- Total invoice berubah

---

## TC-15 Delete KeepComm

### Steps

1. Hapus salah satu keepcomm

### Expected Result

- KeepComm berhasil dihapus
- Total invoice berubah

---

## TC-16 Validate Calculation After Edit

### Expected Result

Perhitungan tetap benar:

- Sub Total
- VAT
- Rounding
- Grand Total

---

## TC-17 Save Edited Invoice

### Steps

1. Klik Save

### Expected Result

- Invoice berhasil disimpan
- Data berubah sesuai perubahan

---

## TC-18 Validate Invoice Preview After Edit

### Steps

1. Masuk ke **Invoice Collection List**
2. Preview invoice

### Expected Result

Data harus sama dengan hasil edit.

---

# 4. Scenario: Duplicate Validation

## TC-19 Duplicate Receipt Prevention

### Steps

1. Tambahkan receipt ke invoice
2. Klik Add lagi

### Expected Result

Receipt yang sama tidak muncul lagi di list.

---

## TC-20 Receipt Used In Another Invoice

### Steps

1. Buat invoice A
2. Gunakan receipt X
3. Buat invoice B
4. Pilih receipt X

### Expected Result

System menolak.

Error:
Receipt already used in another invoice


---

# 5. Scenario: Concurrency Testing

## TC-21 Double Tab Invoice Creation

### Steps

1. Buka 2 tab create invoice
2. Pilih receipt yang sama
3. Save kedua invoice

### Expected Result

Hanya 1 invoice yang berhasil save.

Invoice kedua harus gagal validasi.

---


# 6. Edge Case Testing

## TC-25 Multiple Receipts

### Steps

1. Tambahkan 10–20 receipts

### Expected Result

- Sistem tetap stabil
- Perhitungan tetap benar

---

## TC-26 Multiple KeepComm

### Steps

1. Tambahkan banyak keepcomm

### Expected Result

Total tetap akurat.

---

# 7. Data Integrity Testing

## TC-28 Invoice History

### Expected Result

History mencatat:

- Create Invoice
- Edit Invoice
- Add KeepComm
- Delete KeepComm
- Delete COGS

---