# Add Cn for Invoice summary

## Test Plan

### Scope
Module yang diuji:

- Invoice
- Summary
- Credit Note (CN)
- Journal
- Settle CN
- Void Process

---

# 1. Pre-Condition

Sebelum testing dimulai:

1. Pilih **Customer**
2. Pastikan customer memiliki setting:
   - Summary aktif
   - KeepComm In aktif
   - KeepComm Out aktif
3. Pastikan user memiliki akses:
   - Create Invoice
   - Create Summary
   - Create CN
   - Journal
   - Void & Settle

### Expected Result

- Customer tersedia dan dapat dipilih
- Setting summary dan keep commission sudah aktif
- Semua module dapat diakses

---

# 2. Scenario: Invoice & Summary

## TC-01 Create Invoice

### Steps

1. Masuk ke module **Invoice**
2. Klik **Create**
3. Pilih customer (yang sudah disetting)
4. Input data invoice
5. Klik Save

### Expected Result

- Invoice berhasil dibuat
- Status invoice = Created

---

## TC-02 Create Summary

### Steps

1. Masuk ke module **Summary**
2. Klik **Create**
3. Pilih invoice dari TC-01
4. Save

### Expected Result

- Summary berhasil dibuat
- Invoice masuk ke summary

---

## TC-03 Publish Summary

### Steps

1. Buka summary
2. Klik **Publish**

### Expected Result

- Summary berhasil dipublish
- Tanggal publish tersimpan

---

# 3. Scenario: Credit Note (CN)

## TC-04 Add CN Before Summary Publish

### Steps

1. Gunakan invoice dari TC-01 (sebelum publish)
2. Klik **Add CN**

### Expected Result

- Tidak bisa add CN
- Muncul pesan error/alasan

---

## TC-05 Add CN After Summary Publish

### Steps

1. Gunakan invoice dari summary yang sudah dipublish
2. Klik **Add CN**
3. Input data CN
4. Save

### Expected Result

- CN berhasil dibuat
- CN Date mengikuti tanggal publish summary

---

## TC-06 Preview Journal CN

### Steps

1. Setelah create CN
2. Klik **Preview Journal**

### Expected Result

- Journal tampil tanpa error
- Data journal sesuai

---

## TC-07 Save CN

### Steps

1. Klik Save CN

### Expected Result

- CN berhasil disimpan
- Status CN = Created

---

# 4. Scenario: Journal Validation

## TC-08 Validate Journal Summary

### Steps

1. Masuk ke module **Journal**
2. Cari journal summary

### Expected Result

- Journal summary tersedia
- Journal CN sudah terbentuk

---

## TC-09 Validate Journal Reference

### Field yang dicek

- RefNo
- RefNo6
- RefNo7

### Expected Result

- RefNo = nomor summary
- RefNo6:
  - Debit = nomor invoice
  - Credit = nomor CN
- RefNo7 = nomor summary

---

# 5. Scenario: Settle CN

## TC-10 Settle CN

### Steps

1. Masuk ke module **CN**
2. Pilih CN
3. Klik **Settle**

### Expected Result

- CN berhasil disettle
- Status CN = Settled

---

# 6. Scenario: Void CN

## TC-11 Void CN

### Steps

1. Create ulang flow:
   - Invoice
   - Summary
   - Publish
   - CN
2. Pilih CN
3. Klik **Void**

### Expected Result

- CN berhasil di-void
- Status CN = Void

---

## TC-12 Validate Invoice After Void CN

### Steps

1. Klik **Add CN**
2. Search invoice yang CN-nya sudah di-void

### Expected Result

- Invoice muncul kembali
- Invoice bisa digunakan untuk create CN lagi

---

# 7. Scenario: Void Summary

## TC-13 Void Summary

### Steps

1. Pilih summary yang sudah memiliki CN
2. Klik **Void**

### Expected Result

- Summary berhasil di-void

---

## TC-14 Validate Journal After Void Summary

### Steps

1. Cek journal summary

### Expected Result

- Semua journal memiliki pembalik
- Termasuk journal CN

---

## TC-15 Validate CN After Void Summary

### Steps

1. Masuk ke Quick Tools / CN List

### Expected Result

- CN ikut ter-void
- Status CN = Void

---

# 8. Data Integrity Testing

## TC-16 Validate End-to-End Flow

### Expected Result

Flow berjalan dengan benar:

- Invoice → Summary → Publish → CN → Journal → Settle → Void

Semua kondisi:
- Validasi berjalan
- Journal sesuai
- Status berubah sesuai proses
