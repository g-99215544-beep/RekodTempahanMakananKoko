# Reka Bentuk: Bayaran & Attach Resit (buang penjejakan "Diambil")

Tarikh: 2026-07-01
App: Tempahan Makanan Kokurikulum (`index.html`, satu fail, Firebase RTDB)

## Ringkasan / Matlamat

Buang konsep penjejakan pengambilan ("sudah/belum diambil") sepenuhnya. Selepas
pengusaha kantin **mengesahkan** tempahan, tempahan terus masuk ke penjejakan
**bayaran**. Untuk setiap tempahan disahkan, pengusaha boleh:

1. Masukkan **nilai (RM)** tempahan (inline dalam kad, taip nombor sahaja cth `23.40`).
2. **Tanda "Sudah Dibayar"** (hanya selepas nilai RM diisi).
3. **Attach resit** (muat naik gambar ke Firebase Storage) — pilihan.

Tab **Bayaran** memaparkan **jumlah belum dibayar** di bahagian atas. Sisi guru
juga memaparkan status bayaran & resit yang di-attach.

## Keputusan Reka Bentuk (disahkan pengguna)

- Attach resit: **muat naik gambar ke Firebase Storage** (bukan URL/base64).
- Struktur tab: **satu tab "Bayaran"** sahaja untuk tempahan disahkan
  (tab "Rekod Tempahan" dibuang).
- Nilai RM: **inline dalam kad tab Bayaran** (boleh ubah bila-bila masa).
- Label "RM" tetap di UI; operator taip nilai nombor sahaja (cth `23.40`).
- Syarat bayar: **RM wajib** sebelum "Sudah Dibayar"; **resit pilihan**
  (boleh attach bila-bila masa).
- Sisi guru: **papar** status bayaran + resit bayaran yang di-attach.

## Perubahan UI

### Tab kantin
Sebelum: `Menunggu Pengesahan | Rekod Tempahan | Bayaran | Urus Menu | Tetapan`
Selepas: `Menunggu Pengesahan | Bayaran | Urus Menu | Tetapan`

- Buang butang tab `tabBtn-rekod`, bahagian `kantinTab-rekod`, dan senarai
  `kantinRekodList`.
- Kemas kini `setKantinTab` array daripada
  `['menunggu','rekod','bayaran','menu','tetapan']` kepada
  `['menunggu','bayaran','menu','tetapan']`.

### Aliran pengesahan
- `confirmOrder(id)`: kekal set `status:'disahkan'`, `resitNo`, `disahkanPada`.
  Buang `claimStatus:'belum'`. Tempahan disahkan kini terus muncul di tab Bayaran.

### Tab Bayaran — kepala (total)
- Tambah kad ringkasan di atas senarai:
  **"Jumlah Belum Dibayar: RM 72.00"** = jumlah `hargaRM` bagi semua tempahan
  `status==='disahkan'` yang `bayaranStatus!=='sudah'` dan `hargaRM` telah diisi.
- Jika ada tempahan disahkan yang `hargaRM` belum diisi, papar nota kecil:
  *"N tempahan belum ditetapkan nilai"* (tidak dikira dalam total).

### Tab Bayaran — setiap kad (tempahan disahkan)
Senarai: semua `status==='disahkan'` (tidak lagi ditapis ikut "diambil").
Kekalkan carian (`bayaranSearch`) & tapisan status (`bayaranFilter`: semua/belum/sudah).

Setiap kad ada:
- **Medan Nilai (RM)** — input inline dengan awalan "RM" tetap; operator taip nombor
  (cth `23.40`). Simpan sebagai nombor ke `hargaRM`. Simpan bila blur/tekan
  butang simpan kecil. Papar nilai sedia ada jika ada.
- **Butang "✓ Tanda Sudah Dibayar"** — jika `hargaRM` kosong/0 → toast amaran
  "Sila isi nilai RM dahulu." dan jangan tukar status. Jika sudah dibayar →
  papar "↺ Tanda Belum Dibayar".
- **Attach Resit** — butang muat naik (`<input type=file accept="image/*">`).
  Muat naik ke Firebase Storage di `tempahanMakanan/resit/{orderId}/{timestamp}-{namafail}`.
  Simpan `resitBayaran:{url,path,dimuatNaikPada}` ke order. Jika sudah ada:
  papar thumbnail/pautan **"Lihat Resit Bayaran"** + butang ganti & buang
  (buang = padam dari Storage & set `resitBayaran:null`).
- **Butang "📄 Lihat Resit"** — resit tempahan digital sedia ada (kekal).

### Modal Resit digital (`resitModal`)
- Buang badge claim (`resitClaimBadge` "BELUM/SUDAH DIAMBIL").
- Kekal `resitBayaranBadge` (BELUM/SUDAH DIBAYAR).
- (Pilihan) papar nilai RM & pautan resit bayaran jika ada — dimasukkan supaya
  resit lengkap. Papar baris "Jumlah (RM)" jika `hargaRM` diisi.

### Sisi Guru (`renderGuruOrders`)
- Buang `claimBadge`; jangan papar badge "diambil".
- Untuk tempahan `disahkan`: papar `bayaranBadge` (BELUM/SUDAH DIBAYAR) dan,
  jika ada `resitBayaran`, pautan **"Lihat Resit Bayaran"** (buka imej dalam
  tab baharu). Kekal butang "Lihat Resit" digital.

## Model Data (RTDB: `tempahanMakanan/tempahan/{id}`)

Tambah:
- `hargaRM` — nombor (cth `23.40`). Tiada = belum ditetapkan.
- `resitBayaran` — objek `{ url:string, path:string, dimuatNaikPada:number }`
  atau tiada/null.

Guna sedia ada (kekal):
- `bayaranStatus` — `'belum'` | `'sudah'`.
- `dibayarPada` — timestamp bila ditanda sudah dibayar.

Berhenti guna (data lama diabai, tidak dipadam):
- `claimStatus`, `diclaimPada`.

## Firebase Storage (baharu)

- Tambah skrip: `firebase-storage-compat.js` (versi 9.23.0, sepadan dgn app/database).
- Inisialisasi `const storage = firebase.storage();` selepas `firebase.initializeApp`.
- Muat naik: `storage.ref(path).put(file)` → dapatkan `getDownloadURL()`.
- Buang: `storage.ref(path).delete()`.
- Path fail: `tempahanMakanan/resit/{orderId}/{timestamp}-{namafail-bersih}`.
- Semasa muat naik: papar keadaan "Memuat naik..." pada butang; had saiz
  (cth tolak > ~5 MB) & sahkan jenis imej; toast ralat jika gagal.

### Nota keselamatan (perlu tindakan pengguna)
App tiada auth (PIN sahaja, disimpan dalam DB) — model terbuka sedia ada.
Pengguna perlu:
1. Aktifkan **Storage** di Firebase Console.
2. Tetapkan **Storage Rules** benarkan baca/tulis di bawah `tempahanMakanan/resit/**`
   (petikan rule akan disediakan semasa perlaksanaan), selaras dengan RTDB rules
   per-node sedia ada (rujuk memori `shared-firebase-rtdb`).

## Format & utiliti

- Tambah `formatRM(n)` → `"RM " + Number(n).toFixed(2)` (cth `RM 23.40`);
  pulangkan `"-"` jika tiada.
- Hurai input RM: buang aksara bukan nombor/titik, `parseFloat`, tolak jika NaN/≤0.

## Skop & YAGNI

- Tiada sejarah/log bayaran, tiada eksport, tiada berbilang resit setiap tempahan
  (satu resit bayaran sahaja setiap tempahan; ganti untuk tukar).
- Tiada perubahan pada aliran Menunggu Pengesahan selain buang set `claimStatus`.
- Data lama dengan `claimStatus` dibiar (tiada migrasi).

## Kesan pada fungsi sedia ada (senarai sentuhan)

- HTML: buang tab rekod + kandungannya; buang `resitClaimBadge`.
- JS buang/ubah: `claimBadge`, `renderKantinRekod`, `toggleClaim`,
  array tab dalam `setKantinTab`, badge claim dalam `renderGuruOrders`,
  `claimStatus` dalam `submitOrder`/`confirmOrder`, logik claim dalam `openResit`.
- JS tambah: muat naik/ buang Storage, medan RM inline + simpan, kiraan total
  belum dibayar, papar resit bayaran (kantin + guru).
- Listener `db.ref('.../tempahan')` buang panggilan `renderKantinRekod()`.
