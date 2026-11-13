# [Konten] Ganti foto lokal, pindah lokasi ke Gorontalo, tanggal 07-01-2027, rekening BRI & Jago

> Plan menggantikan plan.md lama (bugfix ornamen hero — sudah tereksekusi).

## 1. Tujuan

Memutakhirkan konten undangan `Amara&Raditya.html` sesuai permintaan user: (a) mengganti semua foto placeholder `picsum.photos` dengan foto lokal di `photo/galeri/` — `mempelai pria.webp` untuk kartu mempelai pria, `mempelai-wanita.webp` untuk mempelai wanita, dan sisanya (`galeri-001`–`galeri-011.webp`) untuk galeri + lightbox; (b) memindahkan seluruh lokasi acara dari Jakarta ke **Gorontalo** (meta description, hero, lokasi Akad & Resepsi, link peta, alamat hadiah fisik, footer); (c) menggeser tanggal pernikahan dari Sabtu 14 Februari 2026 menjadi **Kamis, 07 Januari 2027**, termasuk target countdown JS; (d) mengganti rekening BCA/Mandiri dengan **BRI 781101003760508** dan **Jago 500382460916**. Semua perubahan hanya menyentuh satu file frontend, tanpa dependency baru, tanpa mengubah skema database, dan tetap mengikuti konvensi `agent.md` (single-file, alt deskriptif, `loading="lazy"`).

## 2. Keputusan yang dikunci (debatable, diputuskan sekarang)

- **Hanya `Amara&Raditya.html` yang diubah.** Tidak ada file di `photo/` yang dipindah, di-rename, atau diedit. Nama file dipakai apa adanya.
- **Tanggal baru = Kamis, 07 Januari 2027.** 7 Januari 2027 jatuh hari **Kamis** — semua label "Sabtu" ikut diganti "Kamis". Waktu acara (08.00–10.00 WIB Akad, 11.00–14.00 WIB Resepsi) tidak berubah.
- **Format rekening:** tampilan grup dibaca-manusiya (BRI `7811 0100 3760 508`, Jago `5003 8246 0916`) tetapi `data-copy` tetap raw digit tanpa spasi (`781101003760508`, `500382460916`) — konsisten dengan perilaku copy-to-clipboard yang ada.
- **Mapping bank ke a.n.:** BRI menggantikan posisi BCA (a.n. Amara Larasati), Jago menggantikan posisi Mandiri (a.n. Raditya Pramudya) — urutan kartu tidak berubah. *(Lihat Pertanyaan terbuka #3 untuk konfirmasi a.n.)*
- **Lokasi generic dulu:** semua "Jakarta Selatan" diganti "Gorontalo". Karena nama gedung & alamat lengkap di Gorontalo belum diketahui, teks venue memakai bentuk placeholder netral: Akad = `Kediaman Mempelai Wanita — Gorontalo`, Resepsi = `Balai Kota/venue — Gorontalo` *(lihat Pertanyaan terbuka #2)*, link peta `https://maps.google.com/?q=Gorontalo`. Alamat hadiah fisik diganti `Gorontalo` (placeholder satu baris). User tinggal menyetel nama gedung/jalan kemudian tanpa struktur berubah.
- **Label timeline kisah `ch-year` "2026" TIDAK diubah** — itu tahun bab cerita (lamaran), bukan tanggal pernikahan.
- **Caption galeri (Anyer, Bandung, dst.) TIDAK diubah** — user tidak memintanya; hanya `src`/`data-full`/`alt` yang disentuh bila foto berganti.
- **Waktu di countdown** tetap `T08:00:00+07:00`, hanya tanggalnya jadi `2027-01-07`.
- **Tidak ada perubahan `agent.md`** yang dilakukan agent. Perubahan yang menyentuh hal-hal yang terdokumentasi di `agent.md` dilaporkan ke user agar user yang mengedit `agent.md` (lihat catatan di bagian akhir plan).
- Setiap tugas = satu commit atomik (konvensi `agent.md`).

## 3. File yang boleh diubah / dibuat

- `Amara&Raditya.html` — satu-satunya file yang diedit (markup `<img>`, teks tanggal/lokasi, `href` peta, kartu rekening, konstanta countdown di JS baris ~1120).
- `plan.md` — file ini (sudah di-rewrite dari nol).

## 4. File yang TIDAK boleh diubah

- `agent.md` — **hanya user** yang boleh mengeditnya (user akan mengupdate sendiri; daftar bagian yang perlu diupdate user ada di bawah).
- `api/config.php`, `api/submit_rsvp.php`, `api/list_wishes.php` — backend tidak tersentuh.
- `data/wishes.db` — skema & data tidak tersentuh.
- `photo/**` — 13 file foto tidak dipindah, di-rename, atau diedit (termasuk tidak me-rename `mempelai pria.webp` → `mempelai-pria.webp`).
- `.gitignore` — tidak tersentuh.
- Bagian `Amara&Raditya.html` yang bukan target: SVG sprite, CSS token warna, gerbang amplop, marquee, RSVP form + fallback localStorage, lightbox (perilaku), `SEED` ucapan, timeline kisah (2018/2020/2023/2026), dresscode, music, petal canvas, kursor kustom.

## 5. Tugas atomik (maksimal 6; satu commit per tugas)

- [ ] **T1 — Foto mempelai.** Ganti 2 `<img>` placeholder:
      - Mempelai wanita (baris ~624): `https://picsum.photos/seed/amara-portret/720/960.jpg` → `photo/galeri/mempelai-wanita.webp` (alt tidak berubah).
      - Mempelai pria (baris ~639): `https://picsum.photos/seed/raditya-portret/720/960.jpg` → `photo/galeri/mempelai%20pria.webp` *(nama file asli ada spasi — lihat Pertanyaan terbuka #1)*.
      Commit: `feat(mempelai): ganti potret placeholder picsum → foto lokal photo/galeri`
- [ ] **T2 — Galeri + lightbox.** Ganti `src` & `data-full` 6 `<figure class="g-item g1..g6">` (baris ~706–727) dari picsum ke foto `photo/galeri/galeri-0XX.webp`. Pemilihan & urutan foto **menunggu jawaban Pertanyaan terbuka #2** (11 foto kandidat untuk 6 slot; jangan menebak). Rasio frame CSS (`aspect-ratio` per `.g1..g6` + `object-fit:cover`) tidak diubah — foto apa pun dipotong aman.
      Commit: `feat(galeri): ganti 6 foto galeri + lightbox → aset photo/galeri`
- [ ] **T3 — Tanggal 07 Januari 2027.** Ganti semua kemunculan tanggal + label hari:
      - `<meta name="description">` (baris 7)
      - Teks gerbang `ec-date` (baris 510), menu overlay `mo-foot` (549), hero side (561), hero date (571), marquee 3× span (585–587), `e-date` Akad & Resepsi (667, 679), footer `f-date` (861): `14 · 02 · 2026` / `14 Februari 2026` → `07 · 01 · 2027` / `Kamis, 07 Januari 2027`.
      - JS countdown (baris ~1120): `new Date('2026-02-14T08:00:00+07:00')` → `new Date('2027-01-07T08:00:00+07:00')`.
      Commit: `feat(tanggal): geser hari-h ke Kamis 07 Januari 2027 (incl. countdown)`
- [ ] **T4 — Lokasi Gorontalo.** Hilangkan semua Jakarta:
      - Baris 7: `Jakarta Selatan` → `Gorontalo` (digabung dalam commit T3 bila tanggal & lokasi diedit berdampingan — meta edit sekali).
      - Hero side (560): `Jakarta Selatan — Indonesia` → `Gorontalo — Indonesia`.
      - Hero `hero-loc` (574): `Balai Kartini — Jakarta Selatan` → placeholder `Venue Resepsi — Gorontalo` *(menunggu Pertanyaan terbuka #2b)*.
      - `e-loc` Akad (669) & Resepsi (681): alamat Kemang/Balai Kartini → `Gorontalo` (placeholder, lihat keputusan).
      - `e-map` hrefs (672, 684): query → `Gorontalo` (placeholder alamat final).
      - Hadiah fisik (849–850): alamat Kemang → `Gorontalo` + `data-copy` ikut diganti.
      - Footer `f-date` (861): `— Jakarta Selatan` → `— Gorontalo`.
      Commit: `feat(lokasi): pindahkan semua lokasi Jakarta → Gorontalo`
- [ ] **T5 — Rekening BRI & Jago.** Kartu 1 (839–841): `BCA / 1230 4567 89` → `BRI / 7811 0100 3760 508`, `data-copy="781101003760508"`, a.n. tetap Amara Larasati. Kartu 2 (844–846): `Mandiri / 1370 9876 54` → `Jago / 5003 8246 0916`, `data-copy="500382460916"`, a.n. tetap Raditya Pramudya.
      Commit: `feat(hadiah): ganti rekening BCA/Mandiri → BRI & Jago`
- [ ] **T6 — Verifikasi** (tanpa commit; perintah di bagian 6). Bila ada regresi: **berhenti & laporkan**, jangan patch tambahan.

*(Penggabungan T3+T4 di baris meta description diizinkan agar meta tidak diedit dua kali; kedua commit tetap terpisah untuk bagian lainnya.)*

## 6. Tugas verifikasi (perintah persis)

```bash
# A. Serve via PHP (wajib — bukan python http.server)
php -S 127.0.0.1:8000 -t .

# B. Ukuran file tidak membengkak (foto via URL lokal relatif, bukan base64)
wc -c 'Amara&Raditya.html'          # harus < 120000 byte

# C. HTML valid — hanya known issues yang tercatat di agent.md
#    (hidden-focusable ×7, no-inline-style ×1, attribute-allowed-values ×1)
npx html-validate 'Amara&Raditya.html'

# D. Tidak ada sisa placeholder Jakarta / tanggal lama / picsum / bank lama
grep -n 'picsum\|Jakarta\|2026-02-14\|BCA\|Mandiri\|1230456789\|1370987654' 'Amara&Raditya.html'
#    → output HARUS kosong (exit 1)
grep -c 'photo/galeri/' 'Amara&Raditya.html'   # harus 14 (2 mempelai + 6 src + 6 data-full)

# E. Foto mempelai & galeri benar-benar ada (URL-decode spasi!)
test -f 'photo/galeri/mempelai-wanita.webp' && test -f 'photo/galeri/mempelai pria.webp' && echo OK
```

Pengecekan browser (`http://127.0.0.1:8000/Amara%26Raditya.html`, buka gerbang `#sealBtn`, tunggu choreography selesai):

```text
1. Console kosong — tidak ada error, terutama 404 foto (semua src lokal harus resolve).
2. Section Mempelai: 2 kartu menampilkan foto mempelai.webp & mempelai-wanita.webp (bukan picsum).
3. Section Galeri: 6 foto tampil; klik satu → lightbox menampilkan versi full lokal; tutup.
4. Countdown: target 07-01-2027 — hitung mundur berjalan per detik, tidak "00:00:00".
5. Section Acara: tanggal "Kamis, 07 Januari 2027", lokasi tanpa kata "Jakarta";
   klik "Petunjuk Arah" → maps query Gorontalo.
6. Section Tanda Kasih: BRI 781101003760508 & Jago 500382460916; tombol Salin → clipboard
   berisi digit TANPA spasi + toast sukses.
7. Semua teks hero/footer/marquee/menu overlay memuat "07 · 01 · 2027" / "Gorontalo".
8. Uji RSVP tetap sehat: submit 1 wish test → muncul; lalu bersihkan data tes:
   sqlite3 data/wishes.db "DELETE FROM wishes WHERE n='Nama Tamu';"
```

Kesuksesan: (D) grep kosong & hitungan 14; (C) hanya known issues; poin browser 1–8 lulus; `wc -c` < 120 KB; `git status` bersih kecuali `plan.md`.

## 7. Di luar cakupan

- Rename `photo/galeri/mempelai pria.webp` (dengan spasi) → nama tanpa spasi, kecuali user memilih demikian (Pertanyaan terbuka #1).
- Menulis nama venue/alamat final Gorontalo yang belum diketahui (hanya placeholder; user isi kemudian).
- Mengubah caption galeri (`Senja pertama di Pantai Anyer`, `Hujan di Bandung…`) — tetap seperti aslinya.
- Mengubah nama mempelai, nama orang tua, handle Instagram, dresscode, isi kisah/SEED.
- Backend RSVP (`api/*.php`, `data/wishes.db`, skema, fallback localStorage) — dilarang diubah.
- Fitur "mahal" (petal canvas, Web Audio, kursor, gerbang amplop) dan `.htmlvalidate.json`.
- Menambah dependency/npm/font, memecah single-file HTML, mengubah warna token.
- Edit `agent.md` — milik user.

## Pertanyaan terbuka (info kurang — tidak ditebak)

1. **Nama file mempelai pria.** User menulis `mempelai-pria.webp`, tetapi file aktual di `photo/galeri/` bernama **`mempelai pria.webp`** (dengan spasi). Plan ini memakai `photo/galeri/mempelai%20pria.webp` tanpa me-rename file. Konfirmasi: biarkan spasi (dengan URL-encode), atau user sendiri me-rename file jadi `mempelai-pria.webp` dan src diubah mengikuti?
2. **Pemilihan & urutan 6 foto galeri.** Tersedia 11 foto (`galeri-001`–`galeri-011.webp`) untuk 6 slot (`g1`–`g6`); slot punya rasio berbeda (4/5, 3/2, 3/4, 4/5, 3/4, 16/9). Which 6, dan urutan mana? *(Kalau tidak dijawab: eksekusi T2 menunggu; foto landscape di tata letak portrait pun aman karena `object-fit:cover`, tapi pemilihannya tetap keputusan user.)*
3. **A.n. rekening.** Plan memakai BRI a.n. **Amara Larasati** dan Jago a.n. **Raditya Pramudya** (mengikuti posisi kartu lama). Konfirmasi apakah pemilik rekening memang demikian.
4. **Alamat & venue Gorontalo final.** Nama gedung Akad/Resepsi, alamat lengkap, dan alamat kirim hadiah fisik di Gorontalo belum diketahui — placeholder dipakai dulu; user bisa mengirim teks final untuk satu commit kecil menyusul.

---

### Catatan untuk user: bagian `agent.md` yang berubah relevansinya (user yang mengedit)

1. **Baris "Target: `2026-02-14T08:00:00+07:00`"** di ceklist verifikasi (bagian "Verifikasi akhir") dan komentar struktur `#countdown` → target "2026-02-14" → jadi `2027-01-07`.
2. **Struktur file** di `agent.md` tidak mencantumkan folder `photo/` (13 webp) — sekarang jadi bagian dari proyek.
3. Bagian "Struktur dalam Amara&Raditya.html" menyebut galeri "6 foto" — tetap 6, tapi sumbernya kini lokal `photo/galeri/`.
