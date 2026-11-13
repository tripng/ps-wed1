# AGENTS.md — Panduan AI Agent Versi "Lebih Murah" + Backend RSVP

> Undangan pernikahan Amara & Raditya. Root proyek adalah folder ini
> (`amara_raka/`). Frontend-nya adalah single-file HTML "premium-efisien" —
> CSS/JS inline, Web Audio sintesis musik, canvas kelopak, kursor kustom 3D,
> gerbang amplop berputar. RSVP punya **backend PHP + SQLite**: data tamu
> persisten di `data/wishes.db`, dengan fallback `localStorage` bila server
> tidak tersedia. AI agent wajib ikuti panduan ini bila ditugaskan
> memelihara/mengembangkan proyek ini.

---

## Stack

Frontend: HTML, CSS, JavaScript (vanilla — tanpa framework), single-file
`Amara&Raditya.html` (semua CSS + JS + SVG sprite inline)
Backend: **PHP 8.5 + PDO** (3 endpoint kecil di `api/`) — disajikan via
`php -S` lokal atau web server produksi yang punya PHP; docroot = root proyek
Database: **SQLite** via PDO — file `data/wishes.db`, dibuat otomatis oleh
`api/config.php` pada permintaan pertama; skema `wishes(id, n, h, t, ts)` —
mirror persis model `localStorage ar-wishes {n,h,t,ts}` + `id` PK
Musik: Web Audio API sintesis (atau ganti static `.mp3` di v. murah)
Animasi: CSS transitions + Canvas (`requestAnimationFrame`) — lihat tabel
ekonomi fitur

---

## Perintah penting (terverifikasi)

| Kebutuhan | Perintah | Catatan |
|---|---|---|
| Validasi HTML | `npx html-validate 'Amara&Raditya.html'` | Auto-detect `.htmlvalidate.json`; lihat bagian "Config .htmlvalidate.json" |
| Cek error JS | Buka di browser, `F12 → Console` — kosong tiada `error` | Serve via PHP agar fetch `/api/*.php` tidak gagal |
| Serve lokal | `php -S 127.0.0.1:8000 -t .` | **Bukan** `python3 -m http.server` — file `.php` tidak akan dijalankan |
| URL lokal | `http://127.0.0.1:8000/Amara%26Raditya.html` | `%26` = karakter `&` pada nama file |
| Cek ukuran file | `wc -c Amara&Raditya.html` — target < 120 KB | Current: ~71 KB uncompressed |
| Cek endpoint list | `curl -sS http://127.0.0.1:8000/api/list_wishes.php` | HTTP 200 + array JSON |
| Cek endpoint submit | `curl -sS -X POST http://127.0.0.1:8000/api/submit_rsvp.php --data-urlencode 'inpName=Nama Tamu' --data-urlencode 'inpMsg=Semoga bahagia' -w '%{http_code}'` | Valid → HTTP 201; nama < 2 char → HTTP 400 |
| Cek DB | `sqlite3 data/wishes.db 'SELECT COUNT(*) FROM wishes;'` | Bersihkan data tes bila dipakai untuk uji |
| Commit (atomik) | `git add -A && git commit -m '...'` lalu `git push origin master` | Branch repo ini `master` (bukan `main`) |
| Cek struktur file | `ls -la Amara&Raditya.html api data .gitignore` | Versi murah + backend = HTML1 + api PHP3 + data/ |

> Tidak ada dev server bundler / npm scripts / package.json (PHP runtime
> perlu server). Project = 1 file HTML + 3 endpoint PHP. Jika sebuah
> pengecekan tak bisa dijalankan, tulis **"TIDAK DIJALANKAN"** + alasannya.
> Jangan pernah tulis "lulus" untuk cek yang tidak benar-benar dijalankan.

---

## Config `.htmlvalidate.json` — PERLU PERBAIKI

Config yang ada **sedang broken** untuk html-validate v11.17.0:

- Key `options.cache` tidak valid — harus **dihapus** (schema v11 tidak punya
  top-level `options`; caching dihapus atau dikelola via CLI flag lain)
- Rule `unique-title` tidak dikenali di v11 — harus **dihapus** (rule ini
  sekarang masuk ke `html-validate:recommended` otomatis, atau cek nama baru)

Config yang **valid** untuk v11:

```json
{
  "extends": ["html-validate:recommended"],
  "rules": {
    "attr-case": "error",
    "element-case": "error",
    "no-dup-class": "warn",
    "no-dup-id": "warn"
  }
}
```

> **Bug asli (jangan fix kecuali diminta):** `hidden-focusable`, `no-inline-style`,
> dan `attribute-allowed-values` — semua ini error yang ada di **kedua** versi
> (single-file & separated). 7× `hidden-focusable` (SVG dengan `aria-hidden`
> di dalam focusable ancestor), 1× `no-inline-style` (`style="--d:..."` di hero),
> 1× `attribute-allowed-values` (`src=""` pada lightbox img). Catat tapi
> jangan perbaiki kecuali diminta.

---

## Struktur file

```
Amara&Raditya.html   ← satu file: HTML + CSS inline + JS inline + SVG sprite inline
.htmlvalidate.json   ← konfigurasi html-validate (lihat di atas)
agent.md             ← file ini
plan.md              ← dokumen rancangan backend RSVP (sudah tereksekusi)
.gitignore           ← ignore data/*.db* (repository tidak menyimpan DB runtime)
api/
├── config.php       ← bootstrap PDO SQLite; satu-satunya sumber koneksi DB
├── submit_rsvp.php  ← POST RSVP: validasi nama ≥ 2 char; 201 {ok,id,ts} / 400
└── list_wishes.php  ← GET daftar ucapan terbaru (max 200), JSON {n,h,t,ts}
data/
└── wishes.db        ← database SQLite (runtime, tidak di-commit)
```

### Struktur dalam `Amara&Raditya.html`

```
<head>
  <meta>, <title>, deskripsi, favicon SVG inline, Google Fonts (Cormorant Garamond, Jost)
  <style>            ← semua CSS inline (token warna + komponen)
</head>
<body class="locked">
  <svg style="display:none">   ← SVG sprite: ikon (i-music, i-copy, i-map, i-ig,
                                  i-check, i-x, i-arrow, i-minus, i-plus, i-chl, i-chr)
                                  + ornamen (orn-divider, orn-flourish, orn-corner)
  <div class="cursor-dot">     ← KURSOR KUSTOM (boleh dihapus di v. murah)
  <div class="cursor-ring">
  <div id="gate" class="gate"> ← GERBANG AMPLOP (bisa disederhanakan)
    ...envelope, wax-seal, flap...
  <header class="nav">         ← NAVIGASI (desktop links + mobile menu overlay)
  <div class="menu-overlay">   ← MENU OVERLAY (mobile)
  <canvas id="petals">         ← KANVAS KELOPAK (boleh dihapus di v. murah)
  <main id="page">
    <section id="beranda">     ← HERO (nama, tanggal, lokasi, scroll cue)
    <section id="countdown">   ← HITUNG MUNDUR (JS Date, target 2026-02-14)
    <section id="mempelai">    ← DUA KARTU MEMPELAI (bride + groom)
    <section id="acara">       ← RANGKAIAN ACARA (Akad + Resepsi) + dresscode
    <section id="galeri">      ← GALERI (6 foto, lightbox)
    <section id="kisah">       ← KISAH (4 chapter: 2018/2020/2023/2026)
    <section id="rsvp">        ← FORM RSVP + daftar ucapan (backend SQLite,
                                  fallback localStorage)
    <section id="hadiah">      ← REKENING TRANSFER (copy-to-clipboard)
  </main>
  <footer class="footer">      ← Footer
  <button id="musicBtn">       ← PEMUTAR MUSIK (boleh diganti/hapus)
  <div id="toasts">            ← Toast notifikasi
  <div id="lightbox">          ← Lightbox galeri (bisa disederhanakan)
  <script>                     ← semua JS inline; RSVP memanggil /api/*.php
  </body>
```

---

## RSVP: alur data

- Judul section RSVP menampilkan jumlah ucapan; daftar diurutkan **terbaru
  dulu** (`ORDER BY ts DESC, id DESC`, max 200).
- Buka halaman → `fetch('/api/list_wishes.php')`; gagal → fallback
  `localStorage('ar-wishes')` + SEED. Offline-first: bila server mati, list
  tetap muncul (localStorage/SEED) dan tombol submit tetap berfungsi dengan
  mirror localStorage.
- Submit → `fetch('/api/submit_rsvp.php', POST x-www-form-urlencoded)` dengan
  body `{inpName, h ('ya'/'tidak'), inpMsg, ts}` → sukses (201) → wish
  ditampilkan langsung oleh klien + mirror localStorage; gagal → tulis ke
  localStorage saja (perilaku asal).
- Server menerima alias field (`inpName`/`n`, `inpMsg`/`t`, `h`) dan membaca
  JSON body bila dikirim sebagai JSON. Validasi server: `mb_strlen(n) >= 2`,
  `ts` server-side bila klien tidak mengirim valid.
- Field mapping disengaja **tanpa bergantung ejaan `name` radio** — server
  memindai nilai `ya`/`tidak` bila key `h` tidak ada.
- Output JSON aman: `json_encode(..., JSON_UNESCAPED_SLASHES |
  JSON_UNESCAPED_UNICODE)`; INSERT via prepared statement; render klien pakai
  `textContent` — bukan `innerHTML`.

---

## Ekonomi fitur: apa yang "mahal" vs "murah"

Ringkasan biaya relatif tiap fitur vs alternatif murah. AI agent **wajib**
memilih kolom kanan bila menstrukturkan versi murah.

| Fitur premium (mahal) | Kenapa mahal | Alternatif murah |
|---|---|---|
| **Canvas kelopak jatuh** (IIFE `Petals`, `requestAnimationFrame` 60fps, 10-20 partikel) | CPU/GPU berkelanjutan, baterai drop, mobile jank | Hapus seluruhnya, atau ganti 3-4 CSS `.confetti` statis + `@keyframes` |
| **Web Audio sintesis musik** (IIFE `Music`: AudioContext, delay, biquad, arpeggio 4-chord) | Kompleks, heap audio, sering blocked autoplay | Ganti pakai 1 file `.mp3` `<audio loop>` atau hapus music sama sekali |
| **Kursor kustom split** (`cursor-dot`/`cursor-ring` + `requestAnimationFrame` follow loop) | Loop rerender tiap frame, kursor tak presisi | Hapus; pakai `cursor: auto` native |
| **Gerbang amplop 3D** (`env-flap` rotateX, `wax-seal` break anim, envelope rise, 4× setTimeout 420/1250/2500/3600ms) | Choreografi JS+CSS timing-sensitif, banyak repaint layer | Ganti jadi 1 elemen, fade-out + slide-up lewat CSS `@keyframes` tunggal (≤800ms) |
| **Hero parallax** (pointermove → transform ornamen `will-change:transform`) | Event handler tiap gerakan mouse | Hapus atau ganti CSS `transform: rotate` statis saja |
| **Marquee `-webkit-text-stroke`** (clip + text masking) | Render berulang tiap frame | Ganti jadi warna solid + `transform translateX` anim (murah) |
| **Lightbox keyboard navigation** (ArrowLeft/Right + Escape listener) | Interaktif kompleks | Lightbox tetap boleh, tapi hapus keyboard listener; cukup klik tutup |
| **IntersectionObserver reveal** tiap `.reveal` | 1 observer + kelas transisi | Pertahankan (murah — 1 IO observer) |
| **RSVP backend SQLite + fallback localStorage** | Simpan data interaksi tamu | Pertahankan; sederhanakan bila diminta (mis. tanpa SEED) |
| **Toast notifikasi animasi** | Transisi CSS ringan | Pertahankan (murah) |

### Prioritas strip (urutan)

1. **Wajib hapus dulu**: canvas petals, Web Audio music, kursor kustom. Ini 3
   konsumen resource terbesar.
2. **Boleh sederhanakan**: gerbang amplop (fade saja), parallax hero, keyboard
   lightbox.
3. **Pertahankan penuh**: countdown, scroll reveal, RSVP form, copy-to-clipboard,
   marquee (tapi pakai warna solid), semua ikon SVG, tipografi & warna token.

---

## Aturan wajib umum

- Hasil akhir tetap **single file HTML** (`Amara&Raditya.html`), semua CSS +
  JS inline. Jangan pisahkan ke file eksternal di v. murah.
- Backend tetap di `api/*.php` (3 endpoint kecil, tanpa framework, tanpa
  dependency). Jangan tambah composer/npm/etc.
- Konten (nama, tanggal, alamat, doa, rekening) **jangan diganti** kecuali
  dimintakan secara eksplisit.
- Font eksternal boleh tetap (Google Fonts `Cormorant Garamond`, `Jost`) —
  itu murah. Jangan tambah font baru.
- Font ikon SVG sprite (`symbol#i-…`) **harus dipertahankan** — reusable & nol
  request ekstra.
- Warna token (`--ink`, `--ink-soft`, `--paper`, `--paper-2`, `--paper-3`,
  `--gold`, `--gold-2`, `--wax`, `--line`, `--line-gold`, `--serif`, `--sans`,
  `--ease`) jaga agar konsisten dengan versi premium.
- Setiap perubahan atomik → commit terpisah.
- Perubahan menyangkut RSVP boleh menyentuh `Amara&Raditya.html` dan
  `api/*.php`. Perubahan non-RSVP hanya menyentuh `Amara&Raditya.html`
  (dan `.htmlvalidate.json` bila perlu memperbaiki config).
- Bila rancangan (daftar fitur murah) ternyata salah, **berhenti & laporkan**.
  Jangan improvisasi.

---

## Konvensi kode (ikuti gaya file asli)

- Dollar-selector ringan `$(s)` / `$$(s)` — jaga.
- `RM` = `matchMedia('(prefers-reduced-motion: reduce)').matches` — semua
  animasi berat **harus** respect ini. Canvas, parallax, musik, kursor kustom
  semuanya cek `if (!RM)` atau `if (RM)` sebelum inisialisasi.
- `toast(msg, type)` — pakai utilitas yang sudah ada: `i-x` untuk error,
  `i-check` untuk success.
- Semua string UI tetap Bahasa Indonesia.
- Semua `aria-label`, `aria-live`, `role="dialog"`, `aria-hidden` — jaga
  aksesibilitas.
- Class nama: kebab-case, semantik (`.sec`, `.sec-alt`, `.sec-ink`, `.reveal`,
  `.gift-item`, `.wax-seal`, `.gate`, `.menu-overlay`, `.lightbox`, `.wish`,
  `.w-name`, `.w-badge`, `.w-text`).
- CSS: gunakan class yang sudah ada sebelum membuat yang baru. Hindari
  `!important` kecuali untuk responsive override (lihat media query
  `max-width:760px` → `.g-item{grid-column:1/-1 !important}`).
- HTML: valid — tutup tag, `lang="id"` di `<html>`, `alt` pada `<img>`,
  `label` untuk input form, `type="button"` pada tombol non-submit.

---

## Aturan JavaScript (ikuti gaya IIFE asli)

- Semua JS di-wrap IIFE: `(function(){ 'use strict'; ... })();`
- Pakai `const`/`let`, jangan `var`.
- Pakai `addEventListener` bukan inline `onclick`.
- Pakai `requestAnimationFrame` untuk animasi canvas.
- Error handling: semua `try/catch` untuk fitur yang bisa gagal (fetch,
  clipboard, localStorage, AudioContext). Fetch RSVP/list wajib punya fallback.
- Semua `setTimeout`/`setInterval` simpan referensinya agar bisa `clear`.
- Web Audio hanya `start()` dari user gesture (klik seal button) — jangan auto.
- Semua string dinamis pakai `textContent` (bukan `innerHTML`) kecuali untuk
  SVG markup yang diketahui aman.

---

## Aturan PHP

- PHP 8.5; setiap file mulai `declare(strict_types=1);` + komentar header
  peran file.
- `api/config.php` = **sumber koneksi tunggal**; endpoint `require __DIR__ .
  '/config.php';` — jangan buka PDO sendiri di endpoint.
- Path DB dihitung dari lokasi file (`dirname(__DIR__) . '/data/wishes.db'`)
  — jangan hardcoded; pastikan menunjuk ke dalam root proyek ini.
- Semua akses DB: prepared statement (cegah SQLi).
- Validasi input di server, bukan hanya di klien; `mb_strlen($name) >= 2`.
- Jangan bocorkan detail error internal (path, stack) ke klien — balas JSON
  generik `{ok:false, msg:'...'}`.
- Header JSON: `application/json; charset=utf-8`; output escaping cukup oleh
  `json_encode` (Unicode unescaped).
- Method yang tidak didukung → HTTP 405; payload tidak valid → HTTP 400.

---

## Aturan HTML

- Di JavaScript, gunakan `textContent` untuk menampilkan teks, bukan
  `innerHTML`, kecuali sudah diketahui aman (SVG icon).
- Pakai `lang="id"` di `<html>`.
- Setiap `<img>` wajib punya `alt` yang deskriptif.
- Setiap `<form>` field wajib punya `<label>` yang terkait (`for` + `id`).
- `aria-hidden="true"` hanya untuk elemen dekoratif, jangan pada elemen yang
  bisa fokus (button, input, link).
- `loading="lazy"` pada semua `<img>` galeri.
- `rel="noopener"` pada semua link `target="_blank"`.

---

## Verifikasi akhir (ceklist sebelum commit)

- [ ] Buka di Chrome & Firefox desktop — layout rapi, tiada horizontal scroll.
- [ ] Buka di HP (lebar ~390px) — semua text terbaca, tombol tappable (≥44px),
      nav menu collapse ke hamburger.
- [ ] Console kosong — tidak ada `error` / `promise rejection`.
- [ ] Countdown hitung mundur tiap detik (atau show `00:00:00` bila lewat).
      Target: `2026-02-14T08:00:00+07:00`.
- [ ] RSVP form: empty name (⚠️ < 2 char) → error; submit → wish muncul di
      list + tersimpan di SQLite (`sqlite3 data/wishes.db 'COUNT(*)' naik`);
      reload → wish tampil dari server. Server mati (Ctrl-C) → reload →
      fallback localStorage/SEED, submit tetap berjalan, Console kosong.
- [ ] Tombol Salin (rekening & alamat) → `navigator.clipboard.writeText` +
      toast sukses. Fallback `document.execCommand('copy')` bila gagal.
- [ ] Endpoint: list `HTTP 200` + array JSON; submit valid `HTTP 201`;
      submit invalid `HTTP 400`; `sqlite3 .schema` cocok skema wishes.
- [ ] `npx html-validate 'Amara&Raditya.html'` — hanya error "known issues"
      di atas.
- [ ] `wc -c Amara&Raditya.html` < 120 KB.
- [ ] Data tes di DB dibersihkan (`DELETE FROM wishes WHERE ...` atau hapus
      `data/wishes.db`) sebelum commit — file `.db` tidak di-commit.
- [ ] `git add -A && git commit -m '...' && git push origin master`.

---

## Contoh commit message

```
perf: strip canvas petals + Web Audio musik → static audio + CSS confetti
```

```
refactor: gerbang amplop 3D → CSS fade-slide, hapus kursor kustom
```

```
perf: ganti marquee text-stroke → solid warna, hapus parallax hero
```

```
docs(agent): record PHP/SQLite backend + RSVP persistence
```

---

## Catatan tambahan

- Root proyek adalah folder **`amara_raka/`** — versi terpisah premium
  (`index.html`/`css/style.css`/`js/main.js`/`assets/icons.svg`) sudah
  dihapus dari repo; yang tetap hidup adalah `Amara&Raditya.html`
  + `api/*.php`.
- RSVP punya backend (`api/submit_rsvp.php`/`api/list_wishes.php` → SQLite
  `data/wishes.db`, dibuat runtime oleh `api/config.php`). LocalStorage
  bukan tempat data resmi — hanya mirror/fallback offline.
- Skema DB mirror persis model localStorage `ar-wishes {n,h,t,ts}` + `id` PK.
  Jangan ubah skema/data tanpa izin.
- Server lokal dev = `php -S 127.0.0.1:8000 -t .` — docroot = root proyek,
  sehingga path `/api/*.php` dan root halaman valid.
- `body.locked` pada load pertama — hanya body yang terkunci (scroll), tidak
  seluruh viewport. Gerbang amplop tetap dapat klik.
- `sessionStorage` tidak dipakai — pakai `localStorage` agar mirror bertahan
  refresh.
- File ini (agent.md) dan `plan.md` adalah panduan AI agent. Jika ada
  kebingungan, selalu merujuk ke kode aktual (`Amara&Raditya.html`,
  `api/*.php`) sebagai sumber kebenaran tunggal.
