# Tab Kantin: Sembunyi Tab Kosong + Jadual Bayaran — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Sembunyikan tab "Menunggu Pengesahan" bila tiada tempahan menunggu, dan tukar senarai tab Bayaran kepada jadual ringkas (Bil/Tarikh/Nama/Program/Kuantiti/Catatan) yang muat lebar skrin telefon, dengan tap-baris untuk buka tindakan bayaran dan baris hijau bila sudah dibayar.

**Architecture:** Semua dalam `index.html`. Task 1 mengubah `renderKantinPending` untuk toggle keterlihatan butang tab. Task 2 menulis semula `renderKantinBayaran` supaya menjana `<table class="bayaran-table">` dengan baris boleh-kembang (guna corak `openRows.bayaran`/`toggleRow` sedia ada), menambah CSS jadual, dan mewarnakan baris yang sudah dibayar.

**Tech Stack:** HTML/CSS/JS vanilla, Firebase RTDB (compat). Tiada rangka ujian — pengesahan manual dalam pelayar + `node --check` pada skrip inline + semakan Grep.

## Global Constraints

- Semua perubahan dalam `index.html` sahaja.
- Semua teks UI dalam Bahasa Melayu, konsisten dengan gaya sedia ada.
- Guna `escapeHtml(...)` untuk sebarang nilai teks yang dimasukkan ke `innerHTML`.
- Jadual Bayaran MESTI muat lebar skrin telefon — TIADA scroll mendatar (`table-layout:fixed`, lebar lajur = 100%, teks panjang dipotong ellipsis).
- Guna semula fungsi bayaran sedia ada tanpa mengubah kelakuannya: `saveHarga`, `toggleBayaran`, `uploadResit`, `removeResit`, `openResit`, `openResitBayaran`, dan pembantu `formatRM`, `formatTarikh`, `itemTagsHtml`, `sumberLabel`, `escapeHtml`.
- Warna baris sudah dibayar guna pemboleh ubah sedia ada `--confirmed-bg` (#E1F2E8). Baris belum dibayar kekal warna default.
- Corak kembang: satu baris terbuka pada satu masa, guna `openRows.bayaran` + `toggleRow('bayaran', id)` sedia ada (fungsi itu memanggil semula `renderKantinBayaran`).
- Tiada rangka ujian automatik — sahkan dengan `node --check` skrip inline + Grep + pelayar.

---

### Task 1: Sembunyikan butang tab "Menunggu Pengesahan" bila 0

**Files:**
- Modify: `index.html` — fungsi `renderKantinPending()` (cari dengan `grep -n "function renderKantinPending"`).

**Interfaces:**
- Consumes: `currentKantinTab` (global sedia ada), `setKantinTab(tab)` (sedia ada), elemen `#tabBtn-menunggu`.
- Produces: tiada fungsi baru.

- [ ] **Step 1: Baca fungsi `renderKantinPending` sepenuhnya**

Guna Grep `function renderKantinPending` dan baca badannya. Ia bermula dengan mengira `list` (tempahan berstatus `menunggu`) dan menetapkan teks `#tabBtn-menunggu` kepada `Menunggu Pengesahan (${list.length})`.

- [ ] **Step 2: Tambah toggle keterlihatan selepas `list` dikira**

Cari baris yang menetapkan teks butang, iaitu:

```js
  document.getElementById('tabBtn-menunggu').textContent = `Menunggu Pengesahan (${list.length})`;
```

Tepat SELEPAS baris itu, tambah:

```js
  const menungguBtn = document.getElementById('tabBtn-menunggu');
  menungguBtn.classList.toggle('hidden', list.length === 0);
  if(list.length === 0 && currentKantinTab === 'menunggu'){
    setKantinTab('bayaran');
  }
```

(Jika teks butang ditetapkan dengan sintaks sedikit berbeza, cari baris `tabBtn-menunggu` yang menetapkan `.textContent` dan letak blok di atas selepasnya.)

- [ ] **Step 3: Sahkan struktur (Grep)**

Run: `grep -n "toggle('hidden', list.length === 0)" index.html`
Expected: satu padanan di dalam `renderKantinPending`.
Run: `grep -n "currentKantinTab === 'menunggu'" index.html`
Expected: satu padanan.

- [ ] **Step 4: Sahkan sintaks skrip inline**

Guna skrip Python ini untuk keluarkan blok `<script>` inline terbesar dan semak sintaks:

```bash
python - <<'PY'
import re
html=open('index.html',encoding='utf-8').read()
s=max(re.findall(r'<script(?![^>]*\bsrc=)[^>]*>(.*?)</script>',html,re.S),key=len)
open('.superpowers/sdd/inline.js','w',encoding='utf-8').write(s)
print(len(s))
PY
node --check .superpowers/sdd/inline.js && echo "SYNTAX OK"
```
Expected: cetak saiz + `SYNTAX OK`.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: sembunyi tab Menunggu Pengesahan bila tiada tempahan menunggu"
```

- [ ] **Step 6: Nota untuk pengesahan pelayar (controller)**

Pengesahan sebenar (butang hilang bila 0, muncul bila ada, auto-tukar ke Bayaran) dibuat oleh controller dalam pelayar — subagent tidak boleh jalankan pelayar. Nyatakan ini dalam laporan.

---

### Task 2: Tukar senarai Bayaran kepada jadual ringkas boleh-kembang

**Files:**
- Modify: `index.html` — (a) CSS: tambah blok gaya `.bayaran-table` berhampiran gaya senarai sedia ada (selepas peraturan `.pay-summary` atau berdekatan `.order-item`); (b) fungsi `renderKantinBayaran()` (cari `grep -n "function renderKantinBayaran"`) — tulis semula bahagian penjanaan senarai supaya menghasilkan jadual; (c) TIADA perubahan pada markup `#kantinBayaranList` diperlukan (kekal `<div>` bekas; jadual dijana ke dalamnya).

**Interfaces:**
- Consumes: `formatRM`, `formatTarikh`, `itemTagsHtml`, `sumberLabel`, `escapeHtml`, `bayaranBadge`, `openRows`, `toggleRow`, `saveHarga`, `toggleBayaran`, `uploadResit`, `removeResit`, `openResit`, `openResitBayaran` (semua sedia ada).
- Produces: tiada fungsi baru (hanya menulis semula badan `renderKantinBayaran`).

- [ ] **Step 1: Baca fungsi `renderKantinBayaran` sepenuhnya**

Guna Grep `function renderKantinBayaran` dan baca keseluruhannya. Perhatikan: pengiraan `allConfirmed`/`unpaid`/`totalBelum`/`tanpaNilai`, penetapan teks `#tabBtn-bayaran`, `#bayaranSummary`, penapisan `search`/`filter`, susunan `sort`, semakan senarai kosong, dan blok akhir `wrap.innerHTML = list.map(...)` yang membina `detail` dan memanggil `expandableItem('bayaran', ...)`.

- [ ] **Step 2: Tambah CSS jadual Bayaran**

Cari peraturan `.pay-summary` dalam CSS (Grep `.pay-summary`). Selepas blok gaya pay-summary (atau mana-mana lokasi munasabah dalam `<style>`), tambah:

```css
  /* ---------- jadual bayaran ---------- */
  .bayaran-table{
    width:100%; table-layout:fixed; border-collapse:collapse;
    background:var(--paper); border:1px solid var(--border);
    border-radius:13px; overflow:hidden; box-shadow:var(--shadow);
    font-size:12px;
  }
  .bayaran-table th, .bayaran-table td{
    padding:9px 8px; text-align:left; vertical-align:top;
    border-bottom:1px solid var(--border);
  }
  .bayaran-table thead th{
    font-size:10.5px; font-weight:700; text-transform:uppercase;
    letter-spacing:.03em; color:var(--ink-soft); background:var(--bg-alt);
  }
  /* lebar lajur — jumlah 100% supaya muat tanpa scroll mendatar */
  .bayaran-table col.c-bil{width:8%;}
  .bayaran-table col.c-tarikh{width:14%;}
  .bayaran-table col.c-nama{width:24%;}
  .bayaran-table col.c-program{width:26%;}
  .bayaran-table col.c-kuan{width:12%;}
  .bayaran-table col.c-catatan{width:16%;}
  .bayaran-table td.ell{
    overflow:hidden; text-overflow:ellipsis; white-space:nowrap;
  }
  .bayaran-table tr.btr{cursor:pointer;}
  .bayaran-table tr.btr:hover{background:var(--bg-alt);}
  .bayaran-table tr.btr.paid, .bayaran-table tr.btr-detail.paid{background:var(--confirmed-bg);}
  .bayaran-table tr.btr.paid:hover{background:var(--confirmed-bg);}
  .bayaran-table tr.btr-detail > td{padding:12px 10px;}
  .bayaran-empty{
    padding:22px; text-align:center; color:var(--ink-soft); font-size:13px;
  }
```

- [ ] **Step 3: Tulis semula blok penjanaan senarai dalam `renderKantinBayaran`**

BIARKAN tidak berubah: pengiraan `allConfirmed`/`unpaid`/`totalBelum`/`tanpaNilai`, penetapan `#tabBtn-bayaran`, penetapan `#bayaranSummary`, penapisan `search`/`filter`, dan baris `list.sort(...)`.

GANTIKAN blok semakan-kosong + `wrap.innerHTML = list.map(...)` (dari `if(list.length === 0){ ... }` hingga penghujung `.join('');`) dengan yang berikut:

```js
  if(list.length === 0){
    wrap.innerHTML = `<div class="bayaran-empty"><strong>Tiada rekod</strong><br>Tempahan yang disahkan akan muncul di sini untuk jejak bayaran.</div>`;
    return;
  }

  const rows = list.map(([id,o], idx) => {
    const paid = (o.bayaranStatus||'belum') === 'sudah';
    const open = openRows.bayaran === id;
    const bil = idx + 1;
    const tp = (o.tarikhPerlu||'').split('-');
    const tarikhPendek = tp.length === 3 ? `${tp[2]}/${tp[1]}` : '-';
    const qty = (o.items||[]).reduce((s,it) => s + (Number(it.kuantiti)||0), 0);
    const catatanSel = o.catatan ? escapeHtml(o.catatan) : '–';

    const headRow = `<tr class="btr ${paid?'paid':''} ${open?'open':''}" onclick="toggleRow('bayaran','${id}')">
        <td>${bil}</td>
        <td>${tarikhPendek}</td>
        <td class="ell">${escapeHtml(o.guruNama||'')}</td>
        <td class="ell">${escapeHtml(o.aktiviti||'')}</td>
        <td>${qty}</td>
        <td class="ell">${catatanSel}</td>
      </tr>`;

    if(!open) return headRow;

    const hargaVal = (o.hargaRM > 0) ? Number(o.hargaRM).toFixed(2) : '';
    const payBtn = paid
      ? `<button class="btn btn-unclaim btn-sm" onclick="toggleBayaran('${id}','belum')">↺ Tanda Belum Dibayar</button>`
      : `<button class="btn btn-claim btn-sm" onclick="toggleBayaran('${id}','sudah')">✓ Tanda Sudah Dibayar</button>`;
    const resitCtrl = o.resitBayaran && o.resitBayaran.url
      ? `<button class="btn btn-ghost btn-sm" onclick="openResitBayaran('${id}')">🧾 Lihat Resit Bayaran</button>
         <label class="btn btn-ghost btn-sm">Ganti<input type="file" accept="image/*" style="display:none" onchange="uploadResit('${id}', this)"></label>
         <button class="btn btn-ghost btn-sm" onclick="removeResit('${id}')">Buang Resit</button>`
      : `<label class="btn btn-ghost btn-sm">📎 Attach Resit<input type="file" accept="image/*" style="display:none" onchange="uploadResit('${id}', this)"></label>`;
    const detail = `
      <div class="order-meta">${escapeHtml(o.guruNama||'')} · ${escapeHtml(o.resitNo||'')} · ${bayaranBadge(o.bayaranStatus)}</div>
      ${sumberLabel(o) ? `<div class="order-sumber">Sumber: ${escapeHtml(sumberLabel(o))}</div>` : ''}
      ${itemTagsHtml(o.items)}
      ${o.catatan ? `<div class="order-note">"${escapeHtml(o.catatan)}"</div>` : ''}
      <div class="pay-line">
        <span class="rm-prefix">RM</span>
        <input type="text" inputmode="decimal" id="harga-${id}" class="rm-input" value="${hargaVal}" placeholder="0.00">
        <button class="btn btn-ghost btn-sm" onclick="saveHarga('${id}')">Simpan Nilai</button>
      </div>
      <div class="order-card-actions">
        ${payBtn}
        <button class="btn btn-ghost btn-sm" onclick="openResit('${id}')">📄 Lihat Resit</button>
        ${resitCtrl}
      </div>`;
    const detailRow = `<tr class="btr-detail ${paid?'paid':''}"><td colspan="6">${detail}</td></tr>`;
    return headRow + detailRow;
  }).join('');

  wrap.innerHTML = `<table class="bayaran-table">
      <colgroup>
        <col class="c-bil"><col class="c-tarikh"><col class="c-nama">
        <col class="c-program"><col class="c-kuan"><col class="c-catatan">
      </colgroup>
      <thead><tr>
        <th>Bil</th><th>Tarikh</th><th>Nama</th><th>Program</th><th>Kuan.</th><th>Catatan</th>
      </tr></thead>
      <tbody>${rows}</tbody>
    </table>`;
```

Nota penting: baris butiran (`btr-detail`) ialah `<tr>` BERASINGAN tanpa `onclick`, jadi klik pada input RM / butang di dalamnya TIDAK akan mengecilkan baris. Hanya baris kepala (`btr`) yang boleh diklik.

- [ ] **Step 4: Sahkan struktur (Grep)**

Run: `grep -n "class=\"bayaran-table\"" index.html`
Expected: satu padanan (dalam `renderKantinBayaran`).
Run: `grep -n "toggleRow('bayaran','" index.html`
Expected: sekurang-kurangnya satu padanan (baris kepala).
Run: `grep -n "btr-detail" index.html`
Expected: padanan dalam CSS dan dalam fungsi.
Run: `grep -n "expandableItem('bayaran'" index.html`
Expected: TIADA padanan (blok lama telah diganti).

- [ ] **Step 5: Sahkan sintaks skrip inline (risiko backtick bersarang)**

```bash
python - <<'PY'
import re
html=open('index.html',encoding='utf-8').read()
s=max(re.findall(r'<script(?![^>]*\bsrc=)[^>]*>(.*?)</script>',html,re.S),key=len)
open('.superpowers/sdd/inline.js','w',encoding='utf-8').write(s)
print(len(s))
PY
node --check .superpowers/sdd/inline.js && echo "SYNTAX OK"
```
Expected: cetak saiz + `SYNTAX OK`. Jika ralat sintaks, semak backtick bersarang dalam `headRow`/`detail`/`detailRow`.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: tab Bayaran jadi jadual ringkas boleh-kembang, baris hijau bila dibayar"
```

- [ ] **Step 7: Nota untuk pengesahan pelayar (controller)**

Subagent tidak boleh jalankan pelayar. Controller akan sahkan dalam pelayar: jadual muat lebar telefon tanpa scroll mendatar; tap baris buka butiran dengan RM/tanda-dibayar/resit; baris sudah-dibayar berlatar hijau; carian/penapis/ringkasan masih berfungsi; senarai kosong papar mesej. Nyatakan ini dalam laporan.

---

## Self-Review Notes

- **Spec coverage:** Bahagian A (sembunyi tab + auto-tukar) → Task 1. Bahagian B: jadual 6-lajur + muat skrin (Task 2 CSS `table-layout:fixed` + ellipsis), satu-baris-per-tempahan + kuantiti jumlah (Task 2 Step 3), tap-untuk-butiran guna `openRows.bayaran`/`toggleRow` (Task 2), guna semula tindakan bayaran (Task 2 detail), baris hijau bila dibayar (Task 2 CSS `.paid` + `--confirmed-bg`), carian/penapis/ringkasan kekal (dibiarkan tidak berubah), mesej kosong (Task 2 Step 3). Semua bahagian spec ada task.
- **Placeholder scan:** Tiada TBD/TODO; setiap langkah ada kod sebenar.
- **Type consistency:** `toggleRow('bayaran', id)`, `openRows.bayaran`, `bayaranBadge`, `formatRM` diguna dengan tandatangan sedia ada. Kelas CSS `.btr`, `.btr-detail`, `.paid`, `.ell`, `.bayaran-table`, `.bayaran-empty` konsisten antara CSS (Step 2) dan penjana (Step 3).
