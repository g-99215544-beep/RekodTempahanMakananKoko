# Sumber Perbelanjaan Wajib — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Guru wajib memilih sumber perbelanjaan (tajuk besar + pecahan kecil) semasa membuat tempahan; pilihan disimpan dan dipaparkan di kad tempahan (guru & kantin) serta resit digital.

**Architecture:** Semua dalam satu fail `index.html`. Satu pemalar JS `SUMBER_PERBELANJAAN` menyimpan hierarki. Dua `<select>` dalam borang guru — dropdown kedua diisi secara dinamik ikut pilihan pertama. Medan `sumberBesar` & `sumberKecil` ditambah pada objek `order`. Fungsi pembantu `sumberLabel(o)` menjana teks paparan yang dikongsi semua tempat.

**Tech Stack:** HTML/CSS/JS vanilla, Firebase RTDB (compat SDK). Tiada rangka ujian — pengesahan adalah manual dalam pelayar.

## Global Constraints

- Semua perubahan dalam `index.html` sahaja (aplikasi satu-fail).
- Semua teks UI dalam Bahasa Melayu, konsisten dengan gaya sedia ada.
- Guna `escapeHtml(...)` untuk sebarang nilai teks yang dimasukkan ke dalam `innerHTML`.
- Tempahan lama tanpa medan sumber MESTI papar tanpa ralat (tiada baris "Sumber").
- Nama tajuk besar & kecil disalin VERBATIM daripada spec `docs/superpowers/specs/2026-07-06-sumber-perbelanjaan-design.md`.
- Tiada rangka ujian automatik — setiap task disahkan dengan membuka `index.html` dalam pelayar dan memerhati kelakuan.

---

### Task 1: Pemalar data `SUMBER_PERBELANJAAN` + pembantu `sumberLabel`

**Files:**
- Modify: `index.html` — tambah pemalar & fungsi dalam blok `<script>`, letak berhampiran pemalar konfigurasi lain (selepas `APP_CONFIG` / sebelum fungsi tempahan; cari lokasi selepas `const firebaseConfig` blok atau berhampiran pembolehubah global `let orders`).

**Interfaces:**
- Produces:
  - `const SUMBER_PERBELANJAAN` — array objek. Setiap objek: `{ besar: string, pecahan: string[] }` ATAU `{ besar: string, kumpulan: [{ label: string, pecahan: string[] }] }`.
  - `function sumberEntryByBesar(besar)` → pulang objek entri atau `undefined`.
  - `function entryHasPecahan(entry)` → `boolean` (true jika ada `kumpulan` atau `pecahan.length > 0`).
  - `function sumberLabel(o)` → `string`. Jika `o.sumberBesar` kosong → `""`. Jika `o.sumberKecil` ada → `` `${o.sumberBesar} — ${o.sumberKecil}` ``. Jika tidak → `o.sumberBesar`.

- [ ] **Step 1: Tambah pemalar data**

Letak selepas pembolehubah global (cth. selepas baris `let orders = {};` atau berhampiran pemalar konfigurasi). Salin senarai verbatim:

```js
/* ============================================================
   SUMBER PERBELANJAAN (rujukan Excel PCG 2026 + 3 tambahan)
   Senarai tetap dalam kod. Kemas kini di sini bila peruntukan berubah.
   ============================================================ */
const SUMBER_PERBELANJAAN = [
  {
    besar: "PCG MATA PELAJARAN",
    kumpulan: [
      { label: "Teras 1", pecahan: [
        "SAINS", "MATEMATIK", "BAHASA INGGERIS", "BAHASA CINA KOMUNIKASI",
        "BAHASA ARAB", "PENDIDIKAN ISLAM", "PENDIDIKAN MORAL", "BAHASA MELAYU"
      ]},
      { label: "Teras 2", pecahan: [
        "REKA BENTUK & TEKNOLOGI", "PENDIDIKAN SENI VISUAL", "PENDIDIKAN MUZIK",
        "PENDIDIKAN JASMANI DAN KESIHATAN", "SEJARAH - SPK1 / 2021",
        "KERTAS UJIAN DALAMAN - SPK1 / 2021",
        "KEGIATAN PENDIDIKAN ISLAM / PENDIDIKAN MORAL - SPK1 / 2021"
      ]}
    ]
  },
  { besar: "PCG BUKAN MATA PELAJARAN", pecahan: [
    "PUSAT SUMBER SEKOLAH", "BIMBINGAN DAN KAUNSELING", "LPBT",
    "YURAN KHAS SEKOLAH - SPK1 / 2021"
  ]},
  { besar: "PRASEKOLAH", pecahan: [
    "PCG PRASEKOLAH", "BANTUAN MAKANAN PRASEKOLAH (BMP)",
    "BANTUAN KOKURIKULUM PRASEKOLAH (BKKP)"
  ]},
  { besar: "BANTUAN SUKAN", pecahan: [
    "BANTUAN SUKAN SEKOLAH - SPK1 / 2021 (BSS)",
    "SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)"
  ]},
  { besar: "BANTUAN KOKURIKULUM", pecahan: [
    "BANTUAN KOKURIKULUM SEKOLAH - SPK1 / 2021 (BKKS)",
    "KOKURIKULUM TAMBAHAN - SPK1 / 2021 (BKKS)"
  ]},
  { besar: "RANCANGAN MAKANAN TAMBAHAN - RMT", pecahan: [] },
  { besar: "KEM BESTARI SOLAT - KBS", pecahan: [] },
  { besar: "PENGANJURAN KEJOHANAN", pecahan: [] }
];

function sumberEntryByBesar(besar){
  return SUMBER_PERBELANJAAN.find(e => e.besar === besar);
}
function entryHasPecahan(entry){
  if(!entry) return false;
  if(entry.kumpulan) return entry.kumpulan.some(k => k.pecahan.length > 0);
  return (entry.pecahan || []).length > 0;
}
function sumberLabel(o){
  if(!o || !o.sumberBesar) return "";
  return o.sumberKecil ? `${o.sumberBesar} — ${o.sumberKecil}` : o.sumberBesar;
}
```

- [ ] **Step 2: Sahkan dalam konsol pelayar**

Buka `index.html` dalam pelayar, buka DevTools Console, taip:

```js
SUMBER_PERBELANJAAN.length
entryHasPecahan(sumberEntryByBesar("BANTUAN SUKAN"))
entryHasPecahan(sumberEntryByBesar("KEM BESTARI SOLAT - KBS"))
sumberLabel({sumberBesar:"BANTUAN SUKAN", sumberKecil:"SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)"})
sumberLabel({sumberBesar:"KEM BESTARI SOLAT - KBS", sumberKecil:""})
sumberLabel({})
```

Expected: `8`, `true`, `false`, `"BANTUAN SUKAN — SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)"`, `"KEM BESTARI SOLAT - KBS"`, `""`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: pemalar sumber perbelanjaan + pembantu sumberLabel"
```

---

### Task 2: Dua dropdown dalam borang guru + pengisian dinamik

**Files:**
- Modify: `index.html:400-408` — tambah medan selepas medan Tarikh (`inTarikh`), sebelum medan "Pilih dari Menu".
- Modify: `index.html:131` — tambah `.field select` pada peraturan gaya input.
- Modify: blok `<script>` — tambah fungsi `populateSumberBesar()`, `onSumberBesarChange()`, dan panggil `populateSumberBesar()` pada permulaan (berhampiran panggilan render awal seperti `renderMenuPicker()`).

**Interfaces:**
- Consumes: `SUMBER_PERBELANJAAN`, `sumberEntryByBesar`, `entryHasPecahan` (Task 1).
- Produces:
  - Elemen `<select id="inSumberBesar">` dan `<select id="inSumberKecil">`, pembungkus `<div id="sumberKecilField">`.
  - `function populateSumberBesar()` — isi dropdown pertama.
  - `function onSumberBesarChange()` — isi/sembunyi dropdown kedua ikut pilihan pertama.

- [ ] **Step 1: Tambah gaya select**

Ubah baris 131 supaya `<select>` dalam borang bergaya sama seperti input. Tukar:

```css
  .field input[type=text], .field input[type=date], .field input[type=time], .field textarea{
```

kepada:

```css
  .field input[type=text], .field input[type=date], .field input[type=time], .field textarea, .field select{
```

- [ ] **Step 2: Tambah HTML medan borang**

Selepas blok medan Tarikh (baris 400-403, berakhir dengan `</div>` medan `inTarikh`) dan sebelum `<div class="field"><label>Pilih dari Menu</label>`, sisipkan:

```html
        <div class="field">
          <label for="inSumberBesar">Sumber Perbelanjaan</label>
          <select id="inSumberBesar" onchange="onSumberBesarChange()">
            <option value="">— Pilih sumber —</option>
          </select>
          <p class="field-hint">Jika anda tidak pasti sumber perbelanjaan mana untuk dipilih, sila rujuk kerani sekolah.</p>
        </div>
        <div class="field hidden" id="sumberKecilField">
          <label for="inSumberKecil">Pecahan</label>
          <select id="inSumberKecil">
            <option value="">— Pilih pecahan —</option>
          </select>
        </div>
```

- [ ] **Step 3: Tambah gaya `.field-hint`**

Selepas baris 130 (`.field label{...}`) tambah:

```css
  .field-hint{margin:6px 0 0; font-size:11.5px; color:var(--ink-soft); line-height:1.4;}
```

- [ ] **Step 4: Tambah fungsi pengisian dropdown**

Letak berhampiran `renderMenuPicker` / fungsi borang guru lain:

```js
function populateSumberBesar(){
  const sel = document.getElementById('inSumberBesar');
  if(!sel) return;
  sel.innerHTML = '<option value="">— Pilih sumber —</option>' +
    SUMBER_PERBELANJAAN.map(e => `<option value="${escapeHtml(e.besar)}">${escapeHtml(e.besar)}</option>`).join('');
}

function onSumberBesarChange(){
  const besar = document.getElementById('inSumberBesar').value;
  const field = document.getElementById('sumberKecilField');
  const sel = document.getElementById('inSumberKecil');
  const entry = sumberEntryByBesar(besar);
  if(!entry || !entryHasPecahan(entry)){
    sel.innerHTML = '<option value="">— Pilih pecahan —</option>';
    sel.value = '';
    field.classList.add('hidden');
    return;
  }
  let opts = '<option value="">— Pilih pecahan —</option>';
  if(entry.kumpulan){
    opts += entry.kumpulan.map(k =>
      `<optgroup label="${escapeHtml(k.label)}">` +
      k.pecahan.map(p => `<option value="${escapeHtml(p)}">${escapeHtml(p)}</option>`).join('') +
      `</optgroup>`
    ).join('');
  } else {
    opts += entry.pecahan.map(p => `<option value="${escapeHtml(p)}">${escapeHtml(p)}</option>`).join('');
  }
  sel.innerHTML = opts;
  sel.value = '';
  field.classList.remove('hidden');
}
```

- [ ] **Step 5: Panggil `populateSumberBesar()` pada permulaan**

Cari lokasi render awal borang guru (cth. berhampiran `renderMenuPicker();` semasa init). Tambah selepasnya:

```js
populateSumberBesar();
```

- [ ] **Step 6: Sahkan dalam pelayar**

Buka `index.html`. Pada borang guru:
- Dropdown "Sumber Perbelanjaan" ada 8 pilihan + placeholder.
- Pilih "PCG MATA PELAJARAN" → dropdown "Pecahan" muncul, tunjuk kumpulan "Teras 1" & "Teras 2".
- Pilih "BANTUAN SUKAN" → "Pecahan" tunjuk 2 pilihan tanpa kumpulan.
- Pilih "KEM BESTARI SOLAT - KBS" → "Pecahan" HILANG (tersembunyi).
- Nota "sila rujuk kerani sekolah" kelihatan di bawah dropdown pertama.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: dropdown sumber perbelanjaan + pecahan pada borang guru"
```

---

### Task 3: Pengesahan & simpanan dalam `submitOrder` + reset

**Files:**
- Modify: `index.html:771-815` — fungsi `submitOrder`.

**Interfaces:**
- Consumes: `sumberEntryByBesar`, `entryHasPecahan` (Task 1); `onSumberBesarChange` (Task 2).
- Produces: objek `order` kini mengandungi `sumberBesar` & `sumberKecil`.

- [ ] **Step 1: Baca nilai & sahkan**

Dalam `submitOrder`, selepas baris `const catatan = ...` (baris 775), tambah:

```js
  const sumberBesar = document.getElementById('inSumberBesar').value;
  const sumberKecilEl = document.getElementById('inSumberKecil');
  const sumberKecil = sumberKecilEl.value;
```

Selepas blok pengesahan tarikh (baris 779, `if(!tarikh){...}`), tambah:

```js
  if(!sumberBesar){ showToast('Sila pilih sumber perbelanjaan.', true); return; }
  const sumberEntry = sumberEntryByBesar(sumberBesar);
  if(entryHasPecahan(sumberEntry) && !sumberKecil){
    showToast('Sila pilih pecahan sumber perbelanjaan.', true); return;
  }
```

- [ ] **Step 2: Tambah medan pada objek `order`**

Dalam objek `const order = {...}` (baris 791-800), selepas `aktiviti: aktiviti,` tambah:

```js
    sumberBesar: sumberBesar,
    sumberKecil: sumberKecil,
```

- [ ] **Step 3: Reset dropdown selepas hantar**

Dalam blok `.then(...)` selepas berjaya (berhampiran baris 806-812 yang reset medan lain), tambah:

```js
      document.getElementById('inSumberBesar').value = '';
      onSumberBesarChange();
```

- [ ] **Step 4: Sahkan dalam pelayar**

Buka `index.html`, isi borang tanpa pilih sumber → tekan Hantar → toast "Sila pilih sumber perbelanjaan." dan tak hantar. Pilih "PCG MATA PELAJARAN" tanpa pecahan → toast "Sila pilih pecahan sumber perbelanjaan." Pilih penuh + item → hantar berjaya; dalam Firebase Console (atau DevTools Network) tempahan baru ada `sumberBesar` & `sumberKecil`. Selepas hantar, kedua-dua dropdown reset ke placeholder & "Pecahan" tersembunyi.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: sahkan & simpan sumber perbelanjaan pada tempahan"
```

---

### Task 4: Papar sumber di kad tempahan (guru & kantin) + resit

**Files:**
- Modify: `index.html:881-886` — detail `renderGuruOrders`.
- Modify: `index.html:907-914` — detail `renderKantinPending`.
- Modify: `index.html:1005-1018` — detail `renderKantinBayaran`.
- Modify: `index.html:519-521` — HTML resit modal (tambah baris Sumber).
- Modify: `index.html:1145-1147` — `openResit` (isi baris Sumber).

**Interfaces:**
- Consumes: `sumberLabel(o)` (Task 1).

- [ ] **Step 1: Tambah baris sumber dalam kad guru**

Dalam `renderGuruOrders`, di dalam templat `detail` (baris 881-886), selepas baris `<div class="order-meta">${escapeHtml(o.guruNama)}</div>` tambah:

```js
      ${sumberLabel(o) ? `<div class="order-sumber">Sumber: ${escapeHtml(sumberLabel(o))}</div>` : ''}
```

- [ ] **Step 2: Tambah baris sumber dalam kad kantin (pending)**

Dalam `renderKantinPending`, selepas baris `<div class="order-meta">${escapeHtml(o.guruNama)} · Dihantar ...</div>` (baris 908) tambah baris yang sama:

```js
      ${sumberLabel(o) ? `<div class="order-sumber">Sumber: ${escapeHtml(sumberLabel(o))}</div>` : ''}
```

- [ ] **Step 3: Tambah baris sumber dalam kad kantin (bayaran)**

Dalam `renderKantinBayaran`, selepas baris `<div class="order-meta">${escapeHtml(o.guruNama)} · ${escapeHtml(o.resitNo||'')}</div>` (baris 1006) tambah baris yang sama:

```js
      ${sumberLabel(o) ? `<div class="order-sumber">Sumber: ${escapeHtml(sumberLabel(o))}</div>` : ''}
```

- [ ] **Step 4: Tambah gaya `.order-sumber`**

Berhampiran gaya `.order-meta` / `.order-note` (cari `.order-note{` dalam CSS), tambah:

```css
  .order-sumber{font-size:12px; color:var(--ink-soft); margin:2px 0 8px; font-weight:600;}
```

- [ ] **Step 5: Tambah baris Sumber dalam resit modal (HTML)**

Dalam resit modal, selepas baris Aktiviti (baris 520) tambah:

```html
      <div class="resit-row"><span>Sumber Perbelanjaan</span><span id="resitSumber">-</span></div>
```

- [ ] **Step 6: Isi baris Sumber dalam `openResit`**

Dalam `openResit`, selepas baris `document.getElementById('resitAktiviti').textContent = o.aktiviti || '-';` (baris 1146) tambah:

```js
  document.getElementById('resitSumber').textContent = sumberLabel(o) || '-';
```

- [ ] **Step 7: Sahkan dalam pelayar**

Buka `index.html`. Hantar tempahan baru dengan sumber "BANTUAN SUKAN" → "SUKAN TAHUNAN...". Semak:
- Kad "Tempahan Saya" (guru) tunjuk baris `Sumber: BANTUAN SUKAN — SUKAN TAHUNAN SEKOLAH - SPK1 / 2021 (BSS)`.
- Sebagai kantin, kad "Menunggu Pengesahan" tunjuk baris sumber sama.
- Sahkan tempahan; kad "Bayaran" tunjuk baris sumber.
- Buka "Lihat Resit" → baris "Sumber Perbelanjaan" tunjuk label.
- Untuk tempahan lama (tiada medan sumber): tiada baris "Sumber" muncul, tiada ralat konsol; resit tunjuk "-".

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: papar sumber perbelanjaan di kad tempahan & resit"
```

---

## Self-Review Notes

- **Spec coverage:** Data hierarki (Task 1), dropdown + optgroup Teras + nota kerani (Task 2), pengesahan wajib + simpanan (Task 3), paparan guru/kantin/resit + serasi tempahan lama (Task 4). Semua bahagian spec ada task.
- **Placeholder scan:** Tiada TBD/TODO; semua langkah ada kod sebenar.
- **Type consistency:** `sumberBesar`/`sumberKecil` konsisten merentas Task 3 & 4; `sumberLabel(o)`, `sumberEntryByBesar`, `entryHasPecahan` ditakrif Task 1 dan diguna dengan tandatangan sama di Task 2-4.
