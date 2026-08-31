# Aturan Agen

Dokumen ini berlaku untuk Codex, Google Antigravity, dan agen pemrograman lain yang mengerjakan repository ini.

## Tujuan kerja

Bangun aplikasi pembelajaran yang mudah dipelihara seorang solo developer. Pilih solusi paling sederhana yang memenuhi kebutuhan yang sudah disetujui. Jangan menambahkan pola enterprise, microservice, atau layanan eksternal tanpa kebutuhan nyata.

## Sumber kebenaran

Baca sebelum mengubah kode:

1. `docs/product-spec.md`
2. `docs/architecture.md`
3. `docs/design-system.md` untuk pekerjaan UI

Jika kode dan dokumen berbeda, jangan diam-diam memilih salah satunya. Catat perbedaannya dan perbarui dokumen bersama perubahan yang disetujui.

## Branch

- Kerjakan perubahan di `dev` atau feature branch yang dibuat dari `dev`.
- `produksi` hanya menerima perubahan stabil yang sudah diuji.
- Jangan menggunakan `main` untuk pengembangan atau deployment.
- Jangan force-push ke `produksi`.
- Pisahkan perubahan besar menjadi commit kecil dengan pesan Conventional Commits, misalnya `feat:`, `fix:`, `docs:`, `test:`, atau `chore:`.

## Batas aplikasi lama

- Jangan mengubah Google Apps Script atau Google Sheets lama dari repository ini.
- Jangan membuat sinkronisasi otomatis dengan aplikasi lama.
- Proses impor harus eksplisit, dapat dipratinjau, tervalidasi, dan aman diulang.
- Data impor tidak boleh menghapus atau memodifikasi sumber spreadsheet.

## Aturan implementasi

- Gunakan modular monolith Next.js, bukan microservice.
- Semua validasi otorisasi wajib dijalankan di server.
- Jangan mengandalkan tombol tersembunyi atau middleware saja sebagai pengamanan.
- Gunakan Drizzle migration untuk perubahan skema; jangan mengubah database produksi secara manual.
- Gunakan Zod pada batas masuk data.
- Jangan menyimpan password, PIN, token, atau secret dalam source code, fixture publik, log, maupun URL.
- Jangan menyimpan PIN sebagai teks biasa.
- Simpan versi jawaban yang sudah dikirim sebagai data immutable.
- Autosave tidak boleh membuat versi jawaban baru atau memenuhi audit log.
- Gunakan transaksi dan idempotency key untuk pengiriman jawaban.
- Tulis waktu database dalam UTC dan tampilkan sebagai `Asia/Jakarta`.
- Hindari satu file berukuran besar yang mencampur UI, query, validasi, dan aturan bisnis.

## UI

- Area murid: mobile-first Comic Neobrutalism edukatif.
- Area guru/admin: Academic Calm, desktop-first tetapi tetap responsif.
- Form membaca dan mengerjakan harus tenang; jangan menerapkan dekorasi komik pada setiap elemen.
- Gunakan token dari `docs/design-system.md`; jangan membuat warna dan shadow baru secara acak.
- Gunakan Bahasa Indonesia yang sederhana pada antarmuka murid.
- Sertakan state loading, kosong, gagal, tersimpan, dan akses ditolak.

## Pengujian dan kualitas

- Perubahan aturan bisnis harus disertai unit test.
- Perubahan akses data harus menguji isolasi peran dan kelas.
- Flow penting harus memiliki integration atau end-to-end test.
- Jalankan lint, type-check, test, dan build sebelum menyatakan selesai.
- Jangan menyatakan berhasil tanpa menunjukkan hasil verifikasi terbaru.
- Jangan memperbaiki test dengan mengurangi cakupan perilaku yang disyaratkan.

## Dokumentasi dan versioning

- Perbarui dokumentasi ketika perilaku, arsitektur, skema, atau token desain berubah.
- Versi mengikuti SemVer.
- Catat versi aplikasi pada `package.json`, footer, dan endpoint `/version` setelah aplikasi tersedia.
- Jangan membuat dokumen BRD/PRD/FRD/SRS terpisah kecuali kompleksitas proyek kelak benar-benar memerlukannya.

## Handoff agen

Jika pekerjaan tidak dapat diselesaikan dengan percaya diri dalam satu percakapan, membutuhkan inspeksi banyak file, atau berisiko kehilangan konteks, hentikan perubahan pada checkpoint yang aman dan sarankan melanjutkan melalui Google Antigravity atau Codex yang terhubung langsung ke repository. Sertakan branch, commit terakhir, pekerjaan selesai, hasil tes, pekerjaan tersisa, dan keputusan yang belum dibuat.

Jangan meninggalkan perubahan setengah jadi tanpa catatan handoff.
