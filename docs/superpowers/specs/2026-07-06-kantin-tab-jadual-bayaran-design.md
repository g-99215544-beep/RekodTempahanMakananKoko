# Tab Kantin: Sembunyi Tab Kosong + Jadual Bayaran — Design

**Tarikh:** 2026-07-06
**Fail terlibat:** `index.html` (aplikasi satu-fail)

## Masalah

Pada tab Pengusaha Kantin:
1. Tab **"Menunggu Pengesahan"** sentiasa dipapar walaupun tiada tempahan menunggu
   (menunjukkan "(0)"), memenuhkan ruang tanpa guna.
2. Tab **Bayaran** memaparkan senarai kad accordion yang memakan ruang menegak dan
   tidak menyerupai borang rekod fizikal yang biasa digunakan sekolah.

Objektif: sembunyikan tab menunggu bila kosong, dan jadikan senarai Bayaran satu
**jadual ringkas** (Bil, Tarikh, Nama, Program, Kuantiti, Catatan) yang **muat lebar
skrin telefon tanpa scroll mendatar**.

## Bahagian A — Sembunyi tab "Menunggu Pengesahan" bila 0

**Kelakuan:**
- Bila bilangan tempahan berstatus `menunggu` = 0 → butang `#tabBtn-menunggu`
  disembunyikan (`hidden`).
- Bila ≥ 1 → butang dipapar dengan kiraan, cth "Menunggu Pengesahan (2)"
  (teks kiraan sedia ada dikekalkan).
- Jika `#tabBtn-menunggu` sedang aktif lalu kiraan jatuh ke 0 → auto-tukar ke tab
  `bayaran` melalui `setKantinTab('bayaran')`.
- Bila kantin log masuk / render pertama dan tiada tempahan menunggu → mendarat di
  tab `bayaran` (terhasil daripada peraturan auto-tukar di atas).

**Pelaksanaan:**
- Dalam `renderKantinPending()`, selepas mengira `list` (tempahan menunggu):
  - `document.getElementById('tabBtn-menunggu').classList.toggle('hidden', list.length === 0)`
  - Jika `list.length === 0 && currentKantinTab === 'menunggu'` → `setKantinTab('bayaran')`.
- `renderKantinPending()` sudah dipanggil pada setiap kemas kini data Firebase, jadi
  keterlihatan tab sentiasa segerak dengan data.

**Nota:** butang tab lain (Bayaran, Urus Menu, Tetapan) tidak terjejas. Tab
*Menunggu Pengesahan* kekal sebagai senarai kad (`card-grid`) — cuma disembunyikan
bila kosong.

## Bahagian B — Tab Bayaran sebagai jadual ringkas

### Struktur

Ganti `<div id="kantinBayaranList" class="card-grid">` (senarai kad accordion) dengan
satu `<table class="bayaran-table">`. Kekalkan di atasnya: ringkasan
`#bayaranSummary`, kotak carian `#bayaranSearch`, penapis `#bayaranFilter`.

**Lajur (6):**
| Lajur | Sumber data | Catatan |
|-------|-------------|---------|
| Bil | nombor turutan | 1, 2, 3… ikut senarai terpapar (selepas tapis/susun) |
| Tarikh | `o.tarikhPerlu` | format pendek `dd/mm` |
| Nama | `o.guruNama` | dipotong dengan ellipsis |
| Program | `o.aktiviti` | dipotong dengan ellipsis |
| Kuantiti | jumlah `o.items[].kuantiti` | integer |
| Catatan | `o.catatan` | dipotong dengan ellipsis; "–" jika kosong |

**Satu baris = satu tempahan** (bukan satu baris per item).

### Muat skrin telefon (tiada scroll mendatar)
- `table-layout: fixed; width: 100%;` dengan lebar lajur ditetapkan (peratus) supaya
  jumlah = 100% dan tidak melimpah.
- Saiz teks kecil pada telefon (cth `font-size: 11–12px`), padding ketat.
- Lajur teks (Nama, Program, Catatan): `overflow:hidden; text-overflow:ellipsis;
  white-space:nowrap;`.
- Cadangan lebar: Bil ~8%, Tarikh ~14%, Nama ~24%, Program ~26%, Kuantiti ~12%,
  Catatan ~16% (jumlah 100%).

### Tap baris → butiran (accordion)
- Tap sebarang baris tempahan → baris butiran (`<tr class="bayaran-detail">` dengan
  satu `<td colspan="6">`) terbuka tepat di bawahnya.
- Guna corak `openRows.bayaran` / `toggleRow('bayaran', id)` sedia ada — satu baris
  terbuka pada satu masa.
- Kandungan butiran = kandungan `detail` sedia ada dalam `renderKantinBayaran`:
  - Senarai item penuh (guna `itemTagsHtml(o.items)`).
  - Catatan penuh (jika ada).
  - Baris RM: input `#harga-<id>` + butang **Simpan Nilai** (`saveHarga`).
  - Butang **Tanda Sudah/Belum Dibayar** (`toggleBayaran`).
  - Kawalan resit: attach / lihat / ganti / buang (`uploadResit`, `openResitBayaran`,
    `removeResit`, `openResit`).

### Penanda status bayaran
- Baris tempahan yang **sudah dibayar** (`o.bayaranStatus === 'sudah'`): seluruh baris
  (dan baris butirannya bila terbuka) berlatar **hijau** (guna warna sedia ada, cth
  `--confirmed-bg` / hijau lembut yang konsisten dengan tema).
- Baris **belum dibayar**: kekal warna default baris.
- Tiada lajur status berasingan; tiada penanda lain (cth nombor hijau) — warna baris
  sahaja.

### Carian, penapis, ringkasan
- `#bayaranSearch` (cari nama guru / aktiviti) dan `#bayaranFilter` (semua/belum/sudah)
  kekal fungsi sedia ada — cuma menapis baris jadual.
- `#bayaranSummary` (Jumlah Belum Dibayar + kiraan tempahan tanpa nilai) kekal seperti
  sedia ada.
- Bila senarai kosong selepas tapis → papar mesej kosong (boleh guna satu baris
  `<tr>` merentang atau `<div>` di bawah jadual).

## Skop & YAGNI
- Tab *Menunggu Pengesahan*, resit digital, tab *Urus Menu* & *Tetapan* tidak berubah.
- Tiada perubahan skema data Firebase — hanya paparan.
- Tiada fungsi baru untuk bayaran (RM, tanda dibayar, resit) — guna semula yang sedia
  ada, cuma dialih ke dalam butiran baris.
- Tempahan yang belum ditetapkan nilai RM kekal dikira/dipapar seperti sekarang
  (ringkasan tidak berubah).
