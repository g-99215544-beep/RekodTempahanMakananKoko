# Sumber Perbelanjaan Wajib pada Tempahan — Design

**Tarikh:** 2026-07-06
**Fail terlibat:** `index.html` (aplikasi satu-fail)

## Masalah

Ketika guru membuat tempahan makanan, tiada rekod sumber perbelanjaan (peruntukan
kewangan) yang digunakan. Sekolah perlu tahu setiap tempahan dikira ke akaun/peruntukan
yang mana, berdasarkan pecahan PCG dan bantuan lain.

Guru mesti **wajib** memilih:
1. **Tajuk besar** sumber perbelanjaan (cth. "PCG MATA PELAJARAN").
2. **Pecahan kecil** di bawah tajuk besar itu (cth. "SAINS") — jika tajuk besar
   tersebut mempunyai pecahan.

## Sumber Data

Rujukan: `PECAHAN AWAL TAHUN PCG 2026.xlsx`, sheet "PECAHAN  PCG 2026", ditambah
3 tajuk besar luar Excel (RMT, KBS, Penganjuran Kejohanan).

Senarai **tetap dalam kod** (pemalar JS `SUMBER_PERBELANJAAN`). Bila senarai berubah
tahun hadapan, fail HTML diedit. Guru tidak boleh ubah.

### Hierarki penuh

1. **PCG MATA PELAJARAN** (ada sub-kumpulan Teras)
   - *Teras 1:* SAINS · MATEMATIK · BAHASA INGGERIS · BAHASA CINA KOMUNIKASI ·
     BAHASA ARAB · PENDIDIKAN ISLAM · PENDIDIKAN MORAL · BAHASA MELAYU
   - *Teras 2:* REKA BENTUK & TEKNOLOGI · PENDIDIKAN SENI VISUAL · PENDIDIKAN MUZIK ·
     PENDIDIKAN JASMANI DAN KESIHATAN · SEJARAH - SPK1 / 2021 ·
     KERTAS UJIAN DALAMAN - SPK1 / 2021 ·
     KEGIATAN PENDIDIKAN ISLAM / PENDIDIKAN MORAL - SPK1 / 2021
2. **PCG BUKAN MATA PELAJARAN**
   - PUSAT SUMBER SEKOLAH · BIMBINGAN DAN KAUNSELING · LPBT ·
     YURAN KHAS SEKOLAH - SPK1 / 2021
3. **PRASEKOLAH**
   - PCG PRASEKOLAH · BANTUAN MAKANAN PRASEKOLAH (BMP) ·
     BANTUAN KOKURIKULUM PRASEKOLAH (BKKP)
4. **BANTUAN SUKAN**
   - BANTUAN SUKAN SEKOLAH - SPK1 / 2021 (BSS) ·
     SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)
5. **BANTUAN KOKURIKULUM**
   - BANTUAN KOKURIKULUM SEKOLAH - SPK1 / 2021 (BKKS) ·
     KOKURIKULUM TAMBAHAN - SPK1 / 2021 (BKKS)
6. **RANCANGAN MAKANAN TAMBAHAN - RMT** — tiada pecahan
7. **KEM BESTARI SOLAT - KBS** — tiada pecahan
8. **PENGANJURAN KEJOHANAN** — tiada pecahan

## Struktur Data (pemalar JS)

```js
const SUMBER_PERBELANJAAN = [
  {
    besar: "PCG MATA PELAJARAN",
    kumpulan: [
      { label: "Teras 1", pecahan: ["SAINS", "MATEMATIK", ...] },
      { label: "Teras 2", pecahan: ["REKA BENTUK & TEKNOLOGI", ...] }
    ]
  },
  { besar: "PCG BUKAN MATA PELAJARAN", pecahan: ["PUSAT SUMBER SEKOLAH", ...] },
  ...
  { besar: "RANCANGAN MAKANAN TAMBAHAN - RMT", pecahan: [] },
  ...
];
```

Setiap entri ada sama ada `pecahan` (senarai rata) **atau** `kumpulan` (senarai
kumpulan ber-`<optgroup>`). `pecahan: []` bermaksud tiada pecahan kecil.

## Perubahan UI (borang Guru)

Diletak antara medan **Tarikh Diperlukan** dan **Pilih dari Menu**:

- Dropdown **"Sumber Perbelanjaan"** (`<select id="inSumberBesar">`) — pilihan pertama
  placeholder "— Pilih sumber —". Wajib.
- Dropdown kedua **"Pecahan"** (`<select id="inSumberKecil">`), pada mulanya tersembunyi.
  Bila tajuk besar dipilih:
  - Jika tajuk besar ada pecahan → isi dropdown kedua (guna `<optgroup>` untuk PCG
    Mata Pelajaran) dan tunjuk. Wajib.
  - Jika tiada pecahan (RMT/KBS/Penganjuran) → sembunyi dropdown kedua dan kosongkan
    nilainya.
- Nota kecil di bawah medan:
  *"Jika anda tidak pasti sumber perbelanjaan mana untuk dipilih, sila rujuk kerani
  sekolah."*

## Pengesahan (`submitOrder`)

Tambah sebelum bina objek `order`:
- Jika `sumberBesar` kosong → toast ralat "Sila pilih sumber perbelanjaan." dan henti.
- Jika tajuk besar terpilih mempunyai pecahan tetapi `sumberKecil` kosong → toast ralat
  "Sila pilih pecahan sumber perbelanjaan." dan henti.

## Simpanan

Dua medan baru dalam objek `order` yang di-`push` ke `tempahan`:

```js
sumberBesar: "BANTUAN SUKAN",
sumberKecil: "SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)"  // "" jika tiada pecahan
```

Selepas hantar berjaya, reset kedua-dua dropdown (`inSumberBesar` = "", sembunyi
`inSumberKecil`).

## Paparan

Fungsi pembantu `sumberLabel(o)` yang pulangkan teks:
- Ada pecahan → `"PCG MATA PELAJARAN — SAINS"`
- Tiada pecahan → `"RANCANGAN MAKANAN TAMBAHAN - RMT"`
- Tempahan lama tiada medan → pulang kosong; baris "Sumber" tidak dipaparkan.

Tunjuk baris "Sumber: <label>" di:
- Kad tempahan Guru (senarai "Tempahan Saya")
- Kad tempahan Kantin — senarai pending & senarai bayaran
- Resit digital

## Skop & YAGNI

- Tempahan sedia ada (tiada medan sumber) tetap papar tanpa ralat — hanya tanpa baris
  "Sumber".
- Tiada kaitan dengan pengiraan baki peruntukan / RM dalam Excel — hanya rekod pilihan.
- Kantin **tidak** boleh edit senarai (ditolak; simpan dalam kod sahaja).
