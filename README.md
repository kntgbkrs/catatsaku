# Catatsaku — Frontend

Aplikasi web pencatatan keuangan pribadi (pemasukan, pengeluaran, transfer, anggaran, target tabungan, tagihan, laporan).
Frontend statis (HTML/CSS/JS murni, tanpa build) untuk GitHub Pages; backend berupa Google Apps Script REST API + Google Sheets.

- **Konfigurasi:** satu-satunya nilai yang perlu diisi adalah `GAS_URL` di `js/config.js`.
- **Tema:** Gelap ("Warm Ember"), Terang ("Warm Cream"), atau Ikuti sistem — dipasang sebelum render pertama.
- **Kinerja:** satu snapshot data dari server (cache berversi), disimpan di perangkat; pindah menu tanpa memanggil server; simpan bersifat optimistis dengan pembatalan otomatis bila server menolak.
- **Grafik:** Chart.js 4.4.1 (cdnjs), dimuat async.
- **Pengukuran:** buka DevTools Console → `Perf.table()` atau `Perf.summary()`.

Struktur:

```
index.html
css/style.css          token tema gelap & terang + komponen
js/config.js           GAS_URL
js/util.js, calc.js    format Rupiah/WITA, terbilang, perhitungan baku PRD 4.4
js/api.js, store.js    fetch ke GAS (text/plain), dedupe, antrean simpan optimistis
js/ui.js, charts.js    komponen UI, pembungkus Chart.js
js/auth.js, app.js     layar PIN, router, header/nav, sesi idle
js/aksi.js             aksi bersama (hapus/urungkan, konfirmasi tagihan, kontribusi)
js/pages/*.js          dashboard, catat, riwayat, dompet, anggaran, target, tagihan, laporan, pengaturan
```

Panduan lengkap: `PANDUAN-INSTALASI.md`.
