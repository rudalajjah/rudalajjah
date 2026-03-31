# Checklist Testing - Modal Add Manual COGS Other And Rentcar
**form :** add manuall untuk product other dan rentcar.  
**Tujuan:** Form untuk entry manual COGS (Cost of Goods Sold) invoice  
**Waktu Testing:** ~2-3 jam (1 person)

---

## ✅ QUICK START - APA YANG PERLU DITEST?

### Step 1: Buka Modal
- [ ] Klik tombol untuk buka modal "Add Manual COGS"
- [ ] Modal terbuka dengan benar
- [ ] Tidak bisa ditutup dengan klik di luar modal

### Step 2: Isi Customer & Passenger Info
- [ ] Field "Customer" sudah terisi otomatis (readonly)
- [ ] Isi "Issued Date" dengan format DD-MM-YYYY (contoh: 15-03-2024)
- [ ] Pilih "Title" untuk Passenger (Mr/Mrs/Ms/Mstr)
- [ ] Isi "First Name" Passenger (WAJIB)
- [ ] Isi "Last Name" Passenger (boleh kosong)
- [ ] Isi "Email" Passenger dengan format benar (contoh: john@email.com)
- [ ] Isi "Mobile Phone" dengan nomor (contoh: 0812-3456-789)
- [ ] Isi "Home Phone" dengan angka saja, max 13 karakter

### Step 3: Remarks (Opsional)
- [ ] Klik "Show / Hide" untuk buka/tutup remark fields
- [ ] Setiap remark field bisa diisi max 70 karakter

### Step 4: Booker Contact (Opsional)
- [ ] Isi data Booker: Title, First Name, Last Name, Email, Phone

### Step 5: Pilih Produk & Supplier
- [ ] Pilih "Product" dari dropdown
- [ ] Setelah pilih, supplier list harus ter-update
- [ ] Pilih "Supplier" dari dropdown yang sudah updated
- [ ] Setelah pilih Supplier, "Supplier Product" list harus ter-update
- [ ] Pilih "Supplier Product"
- [ ] Isi "Booking Code" (wajib)

### Step 6: Info Transaksi
- [ ] Isi "Reference Number / Voucher" sesuai produk yang dipilih
- [ ] Jika produk = "Rental" (RT), field ini jadi readonly
- [ ] Isi "Issued By" dari dropdown
- [ ] Isi "Transaction Description"

### Step 7: Rental Dates (Hanya jika Product = Rental/RT)
- [ ] Isi "Rental Start Date" (format DD-MM-YYYY)
- [ ] Isi "Rental End Date" (format DD-MM-YYYY)
- [ ] Field "Duration" harus auto-update dengan jumlah hari

### Step 8: Financial Details
- [ ] Pilih "Currency" dari dropdown
- [ ] Isi "Base Fare" dengan angka desimal 2 (contoh: 1000.50)
- [ ] Isi "Service Fee" dengan angka desimal 2
- [ ] Isi "Quantity" dengan min 1
- [ ] Cek "Net To Agent (NTA)" auto-hitung: `Base Fare × Qty × Duration`
- [ ] Cek "Net Sales" auto-hitung: `(Base Fare × Qty × Duration) + (Service Fee × Qty × Duration)`

### Step 9: Tombol & Aksi Final
- [ ] Klik tombol "Save Manual COGS" → form submit & modal tutup
- [ ] Klik tombol "Cancel" → modal tutup tanpa simpan

---

## 🔍 VALIDASI YANG HARUS DICEK

### Field Wajib Diisi
- [ ] Issued Date (format DD-MM-YYYY)
- [ ] Passenger Title
- [ ] Passenger First Name
- [ ] Product
- [ ] Supplier
- [ ] Supplier Product
- [ ] Booking Code
- [ ] Reference Number / Voucher
- [ ] Issued By
- [ ] Description
- [ ] Currency
- [ ] Jika Rental (RT): Rental Start Date, Rental End Date

### Error Handling - Coba Test
- [ ] Tekan "Save" tanpa isi Issued Date → Error message muncul
- [ ] Tekan "Save" tanpa pilih Product → Error message muncul
- [ ] Isi Start Date > End Date untuk rental → Tidak bisa save
- [ ] Remark lebih dari 70 karakter → Otomatis potong/truncate

---

## 🧮 PERHITUNGAN YANG HARUS BENAR

### Test Calculation Set 1:
```
Base Fare: 1000
Service Fee: 100
Qty: 1
Duration: 1
Expected NTA: 1000.00
Expected Net Sales: 1100.00
```
**Verify:** Hitung otomatis benar?

### Test Calculation Set 2:
```
Base Fare: 500
Service Fee: 50
Qty: 2
Duration: 3
Expected NTA: 3000.00 (500 × 2 × 3)
Expected Net Sales: 3300.00 (3000 + 50×2×3)
```
**Verify:** Hitung otomatis benar?

### Test Calculation Set 3 (Null/Empty):
```
Base Fare: kosong
Service Fee: 100
Qty: 1
Duration: 1
Expected NTA: 0
Expected Net Sales: 100.00
```
**Verify:** Nilai kosong tidak error?

---

## 🎯 SPECIAL CASES

### Case 1: Product Type Conditional (Hanya jika OT)
- [ ] Pilih Product = Rental → field "Product Type" TIDAK muncul
- [ ] Pilih Product = Other (OT) → field "Product Type" MUNCUL
- [ ] Ganti Product dari OT ke Rental → "Product Type" otomatis hilang

### Case 2: Rental Conditional (Hanya jika RT)
- [ ] Pilih Product = Rental → field "Rental Dates" TIDAK muncul
- [ ] Pilih Product = Rental (RT) → field "Rental Start/End" MUNCUL
- [ ] Field "Duration" auto-calculate saat entry Start & End date

### Case 3: Dropdown Dynamic
- [ ] Ganti Product → Supplier list BERUBAH (filtered)
- [ ] Ganti Supplier → Supplier Product list BERUBAH (filtered)
- [ ] Summary Preview di bawah UPDATE dengan supplier_product_name

### Case 4: Readonly Field
- [ ] Produk Rental (RT) → field "Voucher Number" = READONLY (tidak bisa diubah)
- [ ] Produk Lain → "Voucher Number" bisa di-edit

---

## POINT YANG DIHARAPKAN
1. Proses add, edit, ataupun delete berjalan.
2. Setelah proses add, data yang tampil pada list sesuai dengan data yang diinput pada form add.
3. Bandingkan hasil add pada manual other dan rent car; nilai amount seperti NTA, sell price, VAT, dan komponen terkait lainnya harus sama/sesuai.
3. Bandingkan journal summary dari add manual other dan rentcar. COA di journal summary harus sama.

---

## 📝 NOTES TESTER
(Silakan isi dengan temuan, bug, atau catatan penting)

```
Issue Found:
[Isikan di sini]

Catatan:
[Isikan di sini]

Todo:
[Isikan di sini]
```

