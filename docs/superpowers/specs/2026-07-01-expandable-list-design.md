# Reka Bentuk: Senarai Boleh Kembang (Accordion)

Tarikh: 2026-07-01
App: Tempahan Makanan Kokurikulum (`index.html`, satu fail)

## Matlamat

Tukar tiga senarai rekod daripada kad penuh kepada **baris list ringkas** yang
**kembang (dropdown) apabila diklik** untuk memaparkan butiran penuh. Perilaku
**accordion**: buka satu baris menutup baris lain dalam senarai yang sama.

Senarai terlibat:
- **Tempahan Saya** (sisi guru) — `renderGuruOrders` / `#guruOrderList`
- **Menunggu Pengesahan** (kantin) — `renderKantinPending` / `#kantinPendingList`
- **Bayaran** (kantin) — `renderKantinBayaran` / `#kantinBayaranList`

## Keputusan (disahkan pengguna)

- Baris ringkas papar: **Aktiviti + tarikh diperlukan + badge status**.
- **Accordion**: satu baris terbuka pada satu masa setiap senarai.
- Skop: **ketiga-tiga** senarai di atas.

## Pendekatan

State accordion dijejak dalam JS (bukan `<details>` native), supaya baris yang
terbuka kekal terbuka merentas kemas kini langsung Firebase (tab Bayaran kerap
render semula). Butiran hilang keadaan buka jika guna `<details>` sebab DOM
dibina semula setiap render.

## Komponen

### State
```js
let openRows = { guru: null, pending: null, bayaran: null }; // id terbuka / null
```

### Fungsi toggle
```js
function toggleRow(listKey, id){
  openRows[listKey] = (openRows[listKey] === id) ? null : id;
  if(listKey === 'guru') renderGuruOrders();
  else if(listKey === 'pending') renderKantinPending();
  else if(listKey === 'bayaran') renderKantinBayaran();
}
```
Render semula senarai berkenaan menjamin accordion (hanya `openRows[listKey]`
mendapat kelas `open`). Fungsi render sedia ada dipanggil oleh listener Firebase,
jadi keadaan buka diguna semula setiap kali.

### Struktur HTML setiap item
```html
<div class="order-item ${isOpen?'open':''}">
  <div class="order-item-head" onclick="toggleRow('<listKey>','<id>')">
    <div class="oi-main">
      <h3>${aktiviti}</h3>
      <div class="oi-sub">Diperlukan ${tarikh}</div>
    </div>
    <div class="oi-right">${badges}<span class="oi-chevron">▾</span></div>
  </div>
  <div class="order-item-detail">
    <!-- kandungan sama seperti kad sekarang: guru/resitNo, item, catatan,
         medan RM + aksi (ikut senarai) -->
  </div>
</div>
```
- `.order-item-head` guna `<div ... onclick>` (konsisten gaya sedia ada; bukan
  `<button>` untuk elak isu heading dalam button). Butang aksi berada dalam
  `.order-item-detail` (sibling), jadi kliknya tidak mencetus toggle.

### CSS baharu
- `.order-item` — kotak (border/paper/radius/shadow seperti `.order-card`).
- `.order-item-head` — flex antara `.oi-main` dan `.oi-right`, `cursor:pointer`.
- `.oi-sub` — meta kecil (warna `--ink-soft`).
- `.oi-chevron` — putar 180° bila `.order-item.open`.
- `.order-item-detail` — `display:none`; `.order-item.open .order-item-detail{display:block}`;
  garis pemisah atas + padding.
- Guna token warna sedia ada; padanan visual dengan kad.

## Kandungan butiran mengikut senarai

- **guru**: badge-row = `statusBadge` (+`bayaranBadge` jika disahkan) di
  `.oi-right`; detail = item tags, catatan, sebab tolak (jika ada), butang
  "Lihat Resit" + "Lihat Resit Bayaran" (jika ada) untuk status disahkan.
- **pending**: badge = `statusBadge` (MENUNGGU); detail = item tags, catatan,
  butang "Sahkan" / "Tolak". (Buang paparan "Dihantar ..." atau pindah ke detail —
  simpan dalam detail sebagai meta.)
- **bayaran**: badge = `bayaranBadge`; detail = resitNo (dalam meta detail),
  item tags, catatan, `pay-line` (RM + Simpan Nilai), butang Tanda Dibayar,
  Attach/Lihat/Buang resit, Lihat Resit.

## Tak berubah

Empty state; kepala tab & "Jumlah Belum Dibayar"; carian/tapisan; borang buat
tempahan; modal resit; Urus Menu; Tetapan; semua logik data (RM, bayaran,
Storage).

## Kesan / risiko

- Nilai RM yang ditaip tetapi belum "Simpan Nilai" hilang bila baris lain dibuka
  (accordion render semula). Munasabah — simpan dahulu.
- Tiada perubahan model data. Perubahan UI/render sahaja.
```
