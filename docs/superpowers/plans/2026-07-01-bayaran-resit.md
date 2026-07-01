# Bayaran & Attach Resit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Buang penjejakan "diambil"; selepas pengesahan, jejak bayaran setiap tempahan (nilai RM inline, tanda sudah dibayar, attach resit ke Firebase Storage) dengan jumlah belum dibayar di tab Bayaran.

**Architecture:** Satu fail `index.html` (HTML + CSS + JS terbenam) di atas Firebase RTDB (compat SDK v9.23.0). Tambah Firebase Storage compat untuk muat naik imej resit. Tiada rangka kerja ujian — pengesahan dibuat secara manual dalam pelayar terhadap Firebase langsung (data dikongsi di bawah node `tempahanMakanan`).

**Tech Stack:** HTML/CSS/vanilla JS, Firebase App/Database/Storage compat 9.23.0.

## Global Constraints

- SEMUA data di bawah node `tempahanMakanan` (const `DB_ROOT`). Jangan tulis di root lain.
- Versi Firebase compat mesti `9.23.0` (padan dengan skrip sedia ada).
- Bahasa UI: Melayu (ikut gaya sedia ada).
- Format wang: `RM ` + 2 titik perpuluhan (cth `RM 23.40`).
- `hargaRM` disimpan sebagai **nombor** (bukan string).
- Path Storage: `tempahanMakanan/resit/{orderId}/{timestamp}-{namafail-bersih}`.
- Tiada migrasi data lama; medan `claimStatus`/`diclaimPada` dibiar wujud tetapi tidak digunakan.
- Ikut gaya kod sedia ada (nama fungsi camelCase Melayu, `showToast`, `db.ref(...).update(...)`).

**Pengesahan manual:** Buka `index.html` dalam pelayar (double-click / Live Server). Masuk mod Pengusaha Kantin guna PIN `1234` (default). Buat tempahan ujian di sisi Guru untuk data.

**Nota branch:** Repo berada di branch `main`. Buat branch kerja dahulu (cth `git checkout -b feat/bayaran-resit`) sebelum commit pertama.

---

### Task 1: Tambah Firebase Storage SDK + utiliti RM

**Files:**
- Modify: `index.html` (bahagian `<head>` skrip; blok init Firebase; blok utiliti format)

**Interfaces:**
- Produces: `storage` (global, `firebase.storage()` atau `null`); `formatRM(n) -> string`; `parseRM(raw) -> number|null`.

- [ ] **Step 1: Tambah skrip Storage compat**

Selepas baris `firebase-database-compat.js` (kini baris ~19), tambah:

```html
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>
```

- [ ] **Step 2: Isytihar & inisialisasi `storage`**

Cari `let db = null;` dan tukar menjadi:

```js
let db = null;
let storage = null;
```

Dalam blok `try` selepas `db = firebase.database();`, tambah `storage = firebase.storage();`:

```js
    firebase.initializeApp(firebaseConfig);
    db = firebase.database();
    storage = firebase.storage();
    firebaseReady = true;
```

- [ ] **Step 3: Tambah utiliti `formatRM` & `parseRM`**

Selepas fungsi `formatMasaPenuh` (sebelum `escapeHtml`), tambah:

```js
function formatRM(n){
  if(n === undefined || n === null || n === '' || isNaN(Number(n))) return "-";
  return "RM " + Number(n).toFixed(2);
}
function parseRM(raw){
  const cleaned = String(raw).replace(/[^0-9.]/g,'');
  const v = parseFloat(cleaned);
  return (isNaN(v) || v <= 0) ? null : v;
}
```

- [ ] **Step 4: Verify (browser console)**

Buka `index.html`, buka DevTools Console, jalankan:

```js
formatRM(23.4)      // "RM 23.40"
formatRM(null)      // "-"
parseRM("RM 72.00") // 72
parseRM("abc")      // null
typeof storage      // "object" (bukan "undefined")
```

Expected: nilai seperti komen di atas; tiada ralat init Firebase di console.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Firebase Storage SDK and RM formatting utilities"
```

---

### Task 2: Buang penjejakan "Diambil" (claim) sepenuhnya

**Files:**
- Modify: `index.html` (tab HTML, resit modal HTML, `setKantinTab`, listener, `claimBadge`, `renderKantinRekod`, `toggleClaim`, `submitOrder`, `confirmOrder`, `renderGuruOrders`, `renderKantinBayaran`, `openResit`)

**Interfaces:**
- Produces: tab kantin tanpa "Rekod Tempahan"; tiada rujukan `claimStatus`/`claimBadge`/`toggleClaim`/`renderKantinRekod` yang tinggal.

- [ ] **Step 1: Buang butang tab "Rekod Tempahan"**

Padam baris:

```html
<button id="tabBtn-rekod" onclick="setKantinTab('rekod')">Rekod Tempahan</button>
```

- [ ] **Step 2: Buang bahagian `kantinTab-rekod`**

Padam keseluruhan blok:

```html
<div id="kantinTab-rekod" class="hidden">
  <div class="list-header">
    <div class="list-toolbar">
      <input type="text" id="rekodSearch" placeholder="Cari nama guru / aktiviti..." oninput="renderKantinRekod()">
    </div>
  </div>
  <div id="kantinRekodList" class="card-grid"></div>
</div>
```

- [ ] **Step 3: Kemas kini `setKantinTab` array**

Tukar:

```js
  ['menunggu','rekod','bayaran','menu','tetapan'].forEach(t => {
```

kepada:

```js
  ['menunggu','bayaran','menu','tetapan'].forEach(t => {
```

- [ ] **Step 4: Buang panggilan `renderKantinRekod()` dalam listener**

Dalam listener `db.ref(\`${DB_ROOT}/tempahan\`).on('value', ...)`, padam baris `renderKantinRekod();` supaya tinggal:

```js
    renderGuruOrders();
    renderKantinPending();
    renderKantinBayaran();
```

- [ ] **Step 5: Buang fungsi `claimBadge`, `renderKantinRekod`, `toggleClaim`**

Padam ketiga-tiga fungsi ini sepenuhnya (`function claimBadge(...)`, `function renderKantinRekod(){...}`, `function toggleClaim(id,newStatus){...}`).

- [ ] **Step 6: Buang tulisan `claimStatus` dalam `submitOrder` & `confirmOrder`**

Dalam objek `order` di `submitOrder`, padam baris `claimStatus: 'belum',`.
Dalam `confirmOrder` `update({...})`, padam baris `claimStatus: 'belum',`.

- [ ] **Step 7: Buang badge claim dalam `renderGuruOrders`**

Tukar bahagian badge-row:

```js
<div class="badge-row">${statusBadge(o.status)}${o.status==='disahkan' ? claimBadge(o.claimStatus) : ''}</div>
```

kepada (sementara — dikemas kini penuh di Task 5):

```js
<div class="badge-row">${statusBadge(o.status)}</div>
```

- [ ] **Step 8: Buang butang "Tanda Belum Diambil" dalam `renderKantinBayaran`**

Padam baris butang:

```js
<button class="btn btn-ghost btn-sm" onclick="toggleClaim('${id}','belum')">↺ Tanda Belum Diambil</button>
```

- [ ] **Step 9: Ubah tapisan senarai Bayaran kepada semua tempahan disahkan**

Tukar:

```js
  let list = Object.entries(orders).filter(([id,o]) => o.status === 'disahkan' && o.claimStatus === 'sudah');
```

kepada:

```js
  let list = Object.entries(orders).filter(([id,o]) => o.status === 'disahkan');
```

- [ ] **Step 10: Buang badge claim dalam resit modal (HTML)**

Tukar blok status-line:

```html
<div style="margin-top:8px; display:flex; gap:6px; justify-content:center; flex-wrap:wrap;"><span id="resitClaimBadge" class="badge badge-claim-pending">BELUM DIAMBIL</span><span id="resitBayaranBadge" class="badge badge-pending">BELUM DIBAYAR</span></div>
```

kepada:

```html
<div style="margin-top:8px; display:flex; gap:6px; justify-content:center; flex-wrap:wrap;"><span id="resitBayaranBadge" class="badge badge-pending">BELUM DIBAYAR</span></div>
```

- [ ] **Step 11: Buang logik `resitClaimBadge` dalam `openResit`**

Padam blok:

```js
  const claimBadgeEl = document.getElementById('resitClaimBadge');
  if(o.claimStatus === 'sudah'){
    claimBadgeEl.textContent = 'SUDAH DIAMBIL';
    claimBadgeEl.className = 'badge badge-claim-done';
  } else {
    claimBadgeEl.textContent = 'BELUM DIAMBIL';
    claimBadgeEl.className = 'badge badge-claim-pending';
  }
```

- [ ] **Step 12: Verify (browser)**

Muat semula app; masuk mod Kantin. Expected:
- Tab hanya: `Menunggu Pengesahan | Bayaran | Urus Menu | Tetapan`.
- Sahkan satu tempahan menunggu → ia terus muncul dalam tab **Bayaran** (tanpa langkah "diambil").
- Console tiada ralat (`renderKantinRekod is not defined` dsb.).
- Kad guru & resit modal tiada lagi badge "DIAMBIL".

- [ ] **Step 13: Commit**

```bash
git add index.html
git commit -m "refactor: remove claim/collection tracking, confirmed orders go straight to Bayaran"
```

---

### Task 3: Nilai RM inline + jumlah belum dibayar + syarat bayar

**Files:**
- Modify: `index.html` (HTML `kantinTab-bayaran`, CSS, `renderKantinBayaran`, `toggleBayaran`, tambah `saveHarga`)

**Interfaces:**
- Consumes: `formatRM`, `parseRM` (Task 1); senarai disahkan (Task 2 Step 9).
- Produces: `saveHarga(id)`; medan `hargaRM` (nombor) pada order; ringkasan jumlah di `#bayaranSummary`.

- [ ] **Step 1: Tambah elemen ringkasan di HTML**

Dalam `<div id="kantinTab-bayaran" class="hidden">`, tepat selepas tag pembuka, sebelum `<div class="list-header">`, tambah:

```html
<div id="bayaranSummary"></div>
```

- [ ] **Step 2: Tambah CSS**

Sebelum blok `/* ---------- modal / receipt ---------- */`, tambah:

```css
  .pay-summary{
    background:var(--primary); color:#fff; border-radius:var(--radius);
    padding:16px 20px; margin-bottom:16px; box-shadow:var(--shadow);
  }
  .pay-summary-label{font-size:12.5px; opacity:.85; letter-spacing:.02em;}
  .pay-summary-amount{font-family:var(--font-display); font-size:26px; font-weight:600; margin-top:2px;}
  .pay-summary-note{font-size:12px; opacity:.85; margin-top:6px;}
  .pay-line{display:flex; align-items:center; gap:8px; margin:10px 0;}
  .rm-prefix{font-family:var(--font-mono); font-weight:600; color:var(--ink-soft); font-size:13px;}
  .rm-input{
    width:110px; padding:8px 11px; border:1px solid var(--border); border-radius:9px;
    background:#fff; font-family:var(--font-mono); font-size:14px; text-align:right;
  }
  .rm-input:focus{border-color:var(--primary); box-shadow:0 0 0 3px rgba(31,77,58,.12); outline:none;}
```

- [ ] **Step 3: Tambah fungsi `saveHarga`**

Sebelum `function toggleBayaran(...)`, tambah:

```js
function saveHarga(id){
  const el = document.getElementById('harga-'+id);
  if(!el) return;
  const v = parseRM(el.value);
  if(v === null){ showToast('Nilai RM tidak sah. Contoh: 23.40', true); return; }
  db.ref(`${DB_ROOT}/tempahan/`+id).update({hargaRM: v})
    .then(() => showToast('Nilai RM disimpan.'))
    .catch(err => showToast('Ralat: ' + err.message, true));
}
```

- [ ] **Step 4: Tambah syarat RM dalam `toggleBayaran`**

Tukar awal fungsi `toggleBayaran`:

```js
function toggleBayaran(id, newStatus){
  const updates = { bayaranStatus: newStatus };
```

kepada:

```js
function toggleBayaran(id, newStatus){
  const o = orders[id];
  if(newStatus === 'sudah' && !(o && o.hargaRM > 0)){
    showToast('Sila isi nilai RM dahulu sebelum tanda sudah dibayar.', true);
    return;
  }
  const updates = { bayaranStatus: newStatus };
```

- [ ] **Step 5: Tulis semula `renderKantinBayaran` (ringkasan + medan RM)**

Ganti keseluruhan fungsi `renderKantinBayaran` dengan:

```js
function renderKantinBayaran(){
  const wrap = document.getElementById('kantinBayaranList');
  if(!wrap) return;
  const search = (document.getElementById('bayaranSearch')?.value || '').trim().toLowerCase();
  const filter = document.getElementById('bayaranFilter')?.value || 'semua';

  const allConfirmed = Object.entries(orders).filter(([id,o]) => o.status === 'disahkan');
  const unpaid = allConfirmed.filter(([id,o]) => (o.bayaranStatus||'belum') !== 'sudah');
  const totalBelum = unpaid.reduce((s,[id,o]) => s + (o.hargaRM > 0 ? o.hargaRM : 0), 0);
  const tanpaNilai = unpaid.filter(([id,o]) => !(o.hargaRM > 0)).length;

  const tabBtn = document.getElementById('tabBtn-bayaran');
  if(tabBtn) tabBtn.textContent = unpaid.length > 0 ? `Bayaran (${unpaid.length})` : 'Bayaran';

  const summary = document.getElementById('bayaranSummary');
  if(summary){
    summary.innerHTML = `<div class="pay-summary">
      <div class="pay-summary-label">Jumlah Belum Dibayar</div>
      <div class="pay-summary-amount">${formatRM(totalBelum)}</div>
      ${tanpaNilai > 0 ? `<div class="pay-summary-note">${tanpaNilai} tempahan belum ditetapkan nilai (tidak dikira dalam jumlah)</div>` : ''}
    </div>`;
  }

  let list = allConfirmed;
  if(search){
    list = list.filter(([id,o]) => (o.guruNama||'').toLowerCase().includes(search) || (o.aktiviti||'').toLowerCase().includes(search));
  }
  if(filter !== 'semua'){
    list = list.filter(([id,o]) => (o.bayaranStatus||'belum') === filter);
  }
  list.sort((a,b) => (b[1].disahkanPada||0) - (a[1].disahkanPada||0));

  if(list.length === 0){
    wrap.innerHTML = `<div class="empty-state"><strong>Tiada rekod</strong>Tempahan yang disahkan akan muncul di sini untuk jejak bayaran.</div>`;
    return;
  }

  wrap.innerHTML = list.map(([id,o]) => {
    const paid = (o.bayaranStatus||'belum') === 'sudah';
    const hargaVal = (o.hargaRM > 0) ? Number(o.hargaRM).toFixed(2) : '';
    const payBtn = paid
      ? `<button class="btn btn-unclaim btn-sm" onclick="toggleBayaran('${id}','belum')">↺ Tanda Belum Dibayar</button>`
      : `<button class="btn btn-claim btn-sm" onclick="toggleBayaran('${id}','sudah')">✓ Tanda Sudah Dibayar</button>`;
    return `<div class="order-card">
      <div class="order-card-top">
        <div>
          <h3>${escapeHtml(o.aktiviti)}</h3>
          <div class="order-meta">${escapeHtml(o.guruNama)} · Diperlukan ${formatTarikh(o.tarikhPerlu)} · ${escapeHtml(o.resitNo||'')}</div>
        </div>
        <div class="badge-row">${bayaranBadge(o.bayaranStatus)}</div>
      </div>
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
      </div>
    </div>`;
  }).join('');
}
```

- [ ] **Step 6: Verify (browser)**

Mod Kantin → tab Bayaran. Dengan sekurang-kurangnya satu tempahan disahkan:
- Kepala papar "Jumlah Belum Dibayar: RM 0.00" pada mulanya + nota "N tempahan belum ditetapkan nilai".
- Taip `23.40` dalam medan RM satu kad → "Simpan Nilai" → toast; muat semula, nilai kekal; jumlah kepala bertambah RM 23.40; kiraan nota berkurang.
- Tekan "Tanda Sudah Dibayar" pada kad tanpa nilai RM → toast amaran, status tak berubah.
- Tekan "Tanda Sudah Dibayar" pada kad ada nilai → badge jadi SUDAH DIBAYAR; jumlah kepala berkurang; label tab berkurang.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: inline RM value, unpaid total header, and paid gating in Bayaran tab"
```

---

### Task 4: Attach resit ke Firebase Storage

**Files:**
- Modify: `index.html` (`renderKantinBayaran` blok butang, tambah `uploadResit`, `removeResit`)

**Interfaces:**
- Consumes: `storage` (Task 1); kad Bayaran (Task 3).
- Produces: `uploadResit(id, input)`, `removeResit(id)`; medan `resitBayaran:{url,path,dimuatNaikPada}` pada order.

- [ ] **Step 1: Tambah fungsi `uploadResit` & `removeResit`**

Selepas `saveHarga`, tambah:

```js
function uploadResit(id, input){
  const file = input.files && input.files[0];
  if(!file){ return; }
  if(!file.type.startsWith('image/')){ showToast('Sila pilih fail imej.', true); input.value=''; return; }
  if(file.size > 5*1024*1024){ showToast('Saiz imej terlalu besar (maksimum 5MB).', true); input.value=''; return; }
  if(!storage){ showToast('Firebase Storage belum sedia.', true); return; }
  const safe = file.name.replace(/[^a-zA-Z0-9._-]/g,'_');
  const path = `${DB_ROOT}/resit/${id}/${Date.now()}-${safe}`;
  const ref = storage.ref(path);
  showToast('Memuat naik resit...');
  ref.put(file)
    .then(snap => snap.ref.getDownloadURL())
    .then(url => db.ref(`${DB_ROOT}/tempahan/${id}`).update({
      resitBayaran: { url, path, dimuatNaikPada: firebase.database.ServerValue.TIMESTAMP }
    }))
    .then(() => showToast('Resit bayaran dimuat naik.'))
    .catch(err => showToast('Ralat muat naik: ' + err.message, true))
    .finally(() => { input.value=''; });
}
function removeResit(id){
  const o = orders[id];
  if(!o || !o.resitBayaran) return;
  if(!confirm('Buang resit bayaran ini?')) return;
  const path = o.resitBayaran.path;
  const clearDb = () => db.ref(`${DB_ROOT}/tempahan/${id}/resitBayaran`).remove()
    .then(() => showToast('Resit dibuang.'))
    .catch(err => showToast('Ralat: ' + err.message, true));
  if(path && storage){
    storage.ref(path).delete().then(clearDb).catch(clearDb); // buang rekod DB walaupun padam Storage gagal
  } else {
    clearDb();
  }
}
```

- [ ] **Step 2: Tambah kawalan resit pada kad Bayaran**

Dalam `renderKantinBayaran`, dalam `list.map(...)`, tambah pembolehubah selepas `const payBtn = ...`:

```js
    const resitCtrl = o.resitBayaran && o.resitBayaran.url
      ? `<a class="btn btn-ghost btn-sm" href="${o.resitBayaran.url}" target="_blank" rel="noopener">🧾 Lihat Resit Bayaran</a>
         <label class="btn btn-ghost btn-sm">Ganti<input type="file" accept="image/*" style="display:none" onchange="uploadResit('${id}', this)"></label>
         <button class="btn btn-ghost btn-sm" onclick="removeResit('${id}')">Buang Resit</button>`
      : `<label class="btn btn-ghost btn-sm">📎 Attach Resit<input type="file" accept="image/*" style="display:none" onchange="uploadResit('${id}', this)"></label>`;
```

Kemudian dalam blok `order-card-actions`, tambah `${resitCtrl}` selepas butang "Lihat Resit":

```js
      <div class="order-card-actions">
        ${payBtn}
        <button class="btn btn-ghost btn-sm" onclick="openResit('${id}')">📄 Lihat Resit</button>
        ${resitCtrl}
      </div>
```

- [ ] **Step 3: Aktifkan Firebase Storage & tetapkan Rules (tindakan manual pengguna)**

Di Firebase Console projek `sk-sri-aman-database`:
1. **Build → Storage → Get started** (aktifkan bucket).
2. **Storage → Rules**, benarkan baca/tulis di bawah laluan app ini sahaja. Tampal (selaras dengan model terbuka RTDB sedia ada):

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /tempahanMakanan/resit/{orderId}/{fileName} {
      allow read, write: if true;
    }
  }
}
```

Nota: setara dengan keterbukaan RTDB semasa (tiada auth, PIN sahaja). Rujuk memori `shared-firebase-rtdb`.

- [ ] **Step 4: Verify (browser)**

Mod Kantin → tab Bayaran → satu kad disahkan:
- Tekan "📎 Attach Resit" → pilih satu imej → toast "Memuat naik..." kemudian "dimuat naik".
- Kad kini papar "🧾 Lihat Resit Bayaran" (buka imej dalam tab baharu), "Ganti", "Buang Resit".
- Semak di Firebase Console → Storage: fail wujud di `tempahanMakanan/resit/{id}/...`.
- "Buang Resit" → sahkan → kawalan kembali ke "Attach Resit"; fail hilang dari Storage.
- Cuba fail bukan imej / > 5MB → toast amaran, tiada muat naik.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: attach/replace/remove payment receipt via Firebase Storage"
```

---

### Task 5: Papar bayaran & resit di sisi guru + resit modal

**Files:**
- Modify: `index.html` (`renderGuruOrders`, `openResit`, resit modal HTML)

**Interfaces:**
- Consumes: `bayaranBadge`, `formatRM`, `resitBayaran` (Task 3/4).
- Produces: kad guru papar status bayaran + pautan resit bayaran; modal resit papar jumlah RM.

- [ ] **Step 1: Papar badge bayaran + resit bayaran dalam `renderGuruOrders`**

Tukar badge-row (yang ditinggalkan di Task 2 Step 7):

```js
<div class="badge-row">${statusBadge(o.status)}</div>
```

kepada:

```js
<div class="badge-row">${statusBadge(o.status)}${o.status==='disahkan' ? bayaranBadge(o.bayaranStatus) : ''}</div>
```

Kemudian tukar blok `actions` untuk tempahan disahkan:

```js
    if(o.status === 'disahkan'){
      actions = `<div class="order-card-actions"><button class="btn btn-ghost btn-sm" onclick="openResit('${id}')">📄 Lihat Resit</button></div>`;
    }
```

kepada:

```js
    if(o.status === 'disahkan'){
      const resitLink = (o.resitBayaran && o.resitBayaran.url)
        ? `<a class="btn btn-ghost btn-sm" href="${o.resitBayaran.url}" target="_blank" rel="noopener">🧾 Lihat Resit Bayaran</a>`
        : '';
      actions = `<div class="order-card-actions"><button class="btn btn-ghost btn-sm" onclick="openResit('${id}')">📄 Lihat Resit</button>${resitLink}</div>`;
    }
```

- [ ] **Step 2: Tambah baris "Jumlah" dalam resit modal (HTML)**

Selepas baris resit `Catatan` dalam `#resitModal`, tambah:

```html
<div class="resit-row"><span>Jumlah</span><span id="resitJumlah">-</span></div>
```

- [ ] **Step 3: Isi `resitJumlah` dalam `openResit`**

Selepas `document.getElementById('resitCatatan').textContent = o.catatan || '-';`, tambah:

```js
  document.getElementById('resitJumlah').textContent = formatRM(o.hargaRM);
```

- [ ] **Step 4: Verify (browser)**

- Sisi Guru: isi nama guru sama seperti tempahan disahkan. Kad papar badge "BELUM/SUDAH DIBAYAR". Jika resit di-attach oleh kantin, pautan "🧾 Lihat Resit Bayaran" muncul & boleh dibuka.
- Tekan "Lihat Resit" → modal papar baris "Jumlah: RM 23.40" (atau "-" jika belum ditetapkan) dan badge bayaran betul; tiada badge "DIAMBIL".

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: show payment status and receipt on teacher side and in digital receipt"
```

---

## Self-Review Notes

- **Spec coverage:** buang claim (Task 2) ✓; single tab Bayaran (Task 2) ✓; RM inline + label tetap (Task 3) ✓; syarat RM sebelum bayar (Task 3) ✓; total belum dibayar + nota nilai belum ditetapkan (Task 3) ✓; attach resit via Storage + ganti/buang (Task 4) ✓; Storage SDK/init/rules (Task 1/4) ✓; guru nampak bayaran + resit (Task 5) ✓; resit modal buang claim + tambah jumlah (Task 2/5) ✓.
- **Type consistency:** `hargaRM` nombor di mana-mana; `resitBayaran.{url,path,dimuatNaikPada}` konsisten Task 4 & 5; `saveHarga/uploadResit/removeResit/toggleBayaran` nama padan antara HTML onclick & definisi.
- **Placeholder scan:** tiada TBD/TODO; setiap langkah kod ada blok kod penuh.
