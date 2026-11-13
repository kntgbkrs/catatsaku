# Panduan Instalasi Catatsaku

Aplikasi keuangan pribadi: **frontend statis di GitHub Pages** + **Google Apps Script sebagai REST API** + **Google Sheets sebagai database**.

```
Browser (GitHub Pages)  ──fetch JSON──►  Apps Script Web App  ──►  Google Sheets / Drive / Gmail
catatsaku-web/                           backend/*.gs
```

---

## 0. Isi paket

| Folder / berkas | Untuk apa | Diunggah ke mana |
|---|---|---|
| `backend/Kode.gs`, `Keamanan.gs`, `Data.gs`, `Transaksi.gs`, `Jadwal.gs`, `appsscript.json` | Server (API, keamanan, trigger) | **Disalin-tempel** ke editor Apps Script |
| `catatsaku-web/` (berisi `index.html`, `css/`, `js/`) | Tampilan aplikasi | **GitHub Pages** — folder inilah yang di-`git init` |
| `catatsaku-web.zip` | Salinan `catatsaku-web/` dalam satu ZIP (`index.html` ada di akar ZIP) | Opsional |
| `dev/` | Simulator GAS, server lokal, data contoh, uji otomatis | **Jangan** diunggah ke mana pun |

> ⚠️ Berkas `.gs` **tidak boleh** masuk repository GitHub. Folder `catatsaku-web/` sudah bersih dari berkas backend.

---

## 1. Keputusan penting (berbeda dari PRD — mohon dibaca)

1. **Akses Web App = "Anyone", bukan "Only myself".** Frontend terpisah di GitHub Pages memanggil Apps Script lewat `fetch` lintas-domain; mode "Only myself" akan memblokirnya (browser diarahkan ke halaman login Google). Karena itu keamanan dipegang server:
   - PIN 6 digit, di-hash SHA-256 iteratif (300×) + salt, disimpan di **Script Properties** (tidak pernah di sheet).
   - Salah 5× → terkunci 15 menit (berlaku juga untuk kode aktivasi).
   - Token sesi bertanda tangan HMAC, umur maks. 12 jam, **habis setelah tidak aktif** (bawaan 15 menit, diperiksa di server).
   - PIN pertama hanya bisa dibuat dengan **kode aktivasi** sekali pakai dari log editor — orang lain yang menemukan URL tidak bisa mendahului Anda.
   - Ganti PIN mencabut semua sesi lain.
2. **Grafik dihitung di browser** dari data yang dikirim server (satu "snapshot"), supaya pindah menu & ganti periode instan tanpa menunggu server. Semua **validasi dan penyimpanan tetap di server** (saldo minus, kategori/dompet aktif, transfer asal ≠ tujuan, dll.).
3. **Simpan bersifat optimistis**: layar langsung berubah, server memvalidasi di belakang. Bila server menolak, perubahan dibatalkan otomatis dan muncul pesan.
4. **Ekspor PDF/Excel** dibuat server dari Google Sheets berformat + grafik bawaan Sheets (bukan gambar kanvas), angka dihitung dengan aturan yang sama dengan layar.
5. `awal_bulan` di sheet Pengaturan saat ini tetap tanggal 1.
6. Tambahan validasi: **penarikan target tidak boleh melebihi dana terkumpul**.
7. Data terakhir disimpan di **localStorage perangkat** agar aplikasi langsung tampil saat dibuka. Menu **Keluar** menghapusnya (gunakan di perangkat bersama).

---

## 2. Pasang backend (Google Apps Script)

1. Buka <https://script.google.com> → **Proyek baru** → ganti nama proyek menjadi `Catatsaku`.
2. **Project Settings** (ikon roda gigi) → centang **Show "appsscript.json" manifest file in editor**.
3. Kembali ke **Editor**, buka `appsscript.json` → hapus isinya → tempel isi `backend/appsscript.json`.
4. Ganti nama `Code.gs` menjadi `Kode` lalu tempel isi `backend/Kode.gs`.
5. Klik **+ → Script** untuk membuat berkas berikut, lalu tempel isinya masing-masing:
   `Keamanan`, `Data`, `Transaksi`, `Jadwal` (urutan berkas bebas).
6. Klik 💾 **Simpan**.
7. Pilih fungsi **`setupAppEnvironment`** di dropdown → **▶ Run** → **Review permissions** → pilih akun Anda → **Advanced → Go to Catatsaku (unsafe)** → **Allow**.
8. Buka **Execution log** dan **catat kode aktivasi**:
   ```
   ✅ Setup selesai.
      Spreadsheet : https://docs.google.com/spreadsheets/d/…
      Folder Drive: https://drive.google.com/drive/folders/…
   🔐 KODE AKTIVASI PIN (catat sekarang, sekali pakai): ABCD2345
   ```
   Setup membuat folder Drive `Catatsaku` (Struk, Backup, Laporan), spreadsheet database, 10 sheet + header, 14 kategori awal, pengaturan bawaan, dan 4 trigger. **Aman dijalankan ulang** — tidak menimpa data, PIN, atau rahasia.
9. **Deploy → New deployment** → ikon roda gigi → **Web app**:
   - Description: `Catatsaku v1.0.0`
   - Execute as: **Me**
   - Who has access: **Anyone**
   - **Deploy** → salin **Web app URL** (berakhiran `/exec`).
10. Uji: buka URL `/exec` di browser → harus tampil `{"success":true,"app":"Catatsaku API","v":"1.0.0"}`.

---

## 3. Pasang frontend (GitHub Pages)

> Folder kerja untuk semua perintah di bawah adalah **`catatsaku-web`** — bukan folder induk `CATATSAKU`. Pastikan `ls` menampilkan `index.html` di baris teratas sebelum `git init`.

1. Edit `catatsaku-web/js/config.js`, ganti URL dengan URL `/exec` dari langkah 2.9:
   ```js
   window.CATATSAKU_CONFIG = { GAS_URL: 'https://script.google.com/macros/s/AKfy…/exec' };
   ```
2. Di github.com: **+ → New repository** → nama mis. `catatsaku` → **Public** → **jangan** centang README → **Create**.
3. Buka Terminal, masuk ke folder frontend dan periksa isinya:
   ```bash
   cd "/Users/muhammadghaza/PROJECT CLAUDE/CATATSAKU/catatsaku-web"
   ```
   ```bash
   ls
   ```
   Harus terlihat: `index.html  README.md  PANDUAN-INSTALASI.md  css  js`.
4. Kirim ke GitHub (satu per satu):
   ```bash
   git init
   ```
   ```bash
   git add .
   ```
   ```bash
   git commit -m "Upload pertama Catatsaku"
   ```
   ```bash
   git branch -M main
   ```
   ```bash
   git remote add origin https://github.com/USERNAME/catatsaku.git
   ```
   ```bash
   git push -u origin main
   ```
   Saat diminta password, tempel **Personal Access Token** (github.com/settings/tokens → classic → centang `repo`), bukan password akun. Layar memang tidak menampilkan karakter saat token ditempel.
5. Repo → **Settings → Pages** → Source **Deploy from a branch** → Branch **main / (root)** → **Save** → centang **Enforce HTTPS**.
6. Tunggu 1–2 menit → buka `https://USERNAME.github.io/catatsaku/`.

---

## 4. Pertama kali memakai

1. Buka situs → layar **Buat PIN Baru** → isi **kode aktivasi** + PIN 6 digit (bukan berurutan / sama semua) → **Simpan PIN**.
2. **Dompet → Tambah Dompet**: rekening bank, e-wallet, tunai, dan dompet bertipe **Tabungan** (untuk target) beserta saldo awal & tanggalnya.
3. **+ Catat Transaksi** untuk mencatat pemasukan, pengeluaran, atau transfer.
4. Opsional: **Anggaran**, **Target Tabungan**, **Tagihan** (draf dibuat otomatis H-n sebelum jatuh tempo), **Pengaturan → Parameter Keuangan → Alokasi pemasukan**.

Lupa PIN → di editor Apps Script jalankan **`resetPin`** → salin kode aktivasi baru dari log → muat ulang aplikasi → Buat PIN.

---

## 5. Memperbarui aplikasi

**Backend** (setelah mengubah `.gs`): Simpan → **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. Jangan memilih *New deployment* (URL akan berubah).
Bila versi baru menambah kolom/sheet, jalankan `setupAppEnvironment` sekali lagi — migrasinya hanya **menambah** sheet/kolom yang belum ada (berdasarkan nama header), tidak pernah menghapus atau memindahkan data.

**Frontend**: naikkan angka `?v=1.0.0` di `index.html` (agar browser tidak memakai versi lama), lalu dari folder `catatsaku-web`:
```bash
git add .
```
```bash
git commit -m "Perbarui Catatsaku"
```
```bash
git push
```
Tekan **Ctrl+Shift+R** / **Cmd+Shift+R** setelah 1–2 menit.

---

## 6. Fungsi editor (jalankan manual dari Apps Script)

| Fungsi | Kegunaan |
|---|---|
| `setupAppEnvironment` | Setup awal / migrasi skema aman (idempoten) |
| `resetPin` | Lupa PIN — hapus PIN, cabut semua sesi, buat kode aktivasi baru |
| `bukaKunciPin` | Buka penguncian setelah 5× salah |
| `rotasiSecretSesi` | Paksa semua perangkat memasukkan PIN lagi |
| `pasangTrigger` | Pasang ulang 4 trigger |
| `matikanKeepWarm` / `nyalakanKeepWarm` | Matikan/nyalakan trigger jaga-hangat 10 menit |
| `cekPerforma` | Status cache, trigger, ukuran Script Properties, backup terakhir |

**Trigger terjadwal (WITA):** draf tagihan & pengingat harian 06:00 · backup mingguan Minggu 02:00 (simpan 12 terakhir) · ringkasan email tgl 1 pukul 08:00 · jaga cache hangat tiap 10 menit (±144 eksekusi singkat/hari, jauh di bawah kuota 90 menit/hari akun Gmail).

---

## 7. Kinerja (diukur di simulator lokal, latensi GAS tiruan 900 ms)

| Situasi | Panggilan server | Waktu tampil |
|---|---|---|
| Buka aplikasi (sesi masih aktif) | 1 (`sync`, kosong bila tidak ada perubahan) | Instan dari data tersimpan |
| Masuk dengan PIN | 3 (`status`, `login`, `bootstrap`) | ±1 dtk per panggilan di GAS nyata |
| Pindah antar 8 menu | **0** | ±20–35 ms per halaman |
| Ganti periode / filter / klik grafik | **0** | Instan |
| Simpan transaksi | 1 (di belakang layar) | Instan (optimistis) |
| Snapshot 6.000 transaksi | — | ±890 KB; dari cache server ±10 ms (gzip berpotongan) |

Cache server berversi: setiap penyimpanan menaikkan versi data, jadi cache tidak pernah basi. Tombol **Pengaturan → Integrasi Google → Segarkan data dari Sheets** membaca ulang bila Anda mengedit spreadsheet langsung.

---

## 8. Hasil uji

- **Backend: 35/35 lulus** (`node dev/test-backend.js`) — termasuk PRD §9 #1–#8, #11, #13, setup idempoten, allowlist pengaturan, ganti PIN mencabut token lama, sesi idle, rotasi backup 12, ekspor, ringkasan email, skala 6.000 transaksi. Simulator memuat tiap `.gs` sebagai skrip terpisah seperti GAS V8 dan meniru konversi otomatis Sheets (mis. "2026-11" → tanggal) untuk memastikan kolom teks aman.
- **Perhitungan: 8/8 lulus** (`node dev/test-calc.js`) — #9 riwayat saldo = hitung manual, #10 pemasukan 0 → "–", #12 angka grafik = riwayat terfilter, heatmap, terbilang.
- **Browser (manual di pane)**: login PIN, dashboard, catat optimistis → ID server, hapus + urungkan, konfirmasi tagihan, kontribusi target, ekspor, penolakan server → rollback, kunci idle, tema gelap/terang, ponsel 360 px tanpa scroll horizontal (#15, #16).
- **Belum diuji di Google sungguhan**: simulator meniru layanan GAS, tetapi deploy nyata (izin OAuth, kuota, ekspor PDF via `UrlFetchApp`) perlu dicoba sekali setelah instalasi.

---

## 9. Menjalankan secara lokal (opsional, untuk pengembangan)

```bash
cd "/Users/muhammadghaza/PROJECT CLAUDE/CATATSAKU" && node dev/seed.js && node dev/server.js
```
Buka <http://localhost:5173> → PIN contoh `258036`. Server menjalankan backend `.gs` asli di simulator; data tersimpan di `dev/data/db.json`.

---

## 10. Kendala umum

| Gejala | Penyebab & solusi |
|---|---|
| "URL backend belum diisi" | Isi `GAS_URL` di `js/config.js`, push ulang |
| "Koneksi ke server bermasalah" | URL salah / deployment bukan **Anyone** / belum **New version** setelah edit |
| "Database belum disiapkan" | Jalankan `setupAppEnvironment` |
| "Kode aktivasi belum ada" | Jalankan `resetPin`, ambil kode dari log |
| Terkunci 15 menit | Tunggu, atau jalankan `bukaKunciPin` |
| Data di layar tidak sama dengan sheet setelah edit manual | Pengaturan → Integrasi Google → **Segarkan data dari Sheets** |
| Situs GitHub Pages 404 | `git init` dijalankan di folder yang salah — harus di `catatsaku-web` |
| Tampilan versi lama setelah push | Naikkan `?v=` di `index.html`, lalu Ctrl+Shift+R |
| Email tidak terkirim | Isi email di Pengaturan → Parameter Keuangan; cek kuota Gmail harian |
