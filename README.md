# belajarIT

Pengembangan ulang aplikasi pembelajaran Informatika LABSchool sebagai aplikasi web mandiri. Aplikasi baru memakai Next.js, Vercel, dan Neon PostgreSQL serta tidak menggantikan, mengubah, atau menyinkronkan aplikasi Google Apps Script yang sudah berjalan.

## Status

Tahap saat ini: spesifikasi produk dan arsitektur awal untuk versi `0.1.0`.

Belum ada aplikasi produksi dari repository ini. Implementasi dimulai setelah spesifikasi dan rencana kerja disetujui.

## Sasaran fase pertama

- Login admin, guru, dan murid.
- Periode akademik, kelas, murid, dan penugasan guru.
- Pertemuan, materi, latihan, resume, refleksi, LKPD, dan asesmen.
- Jawaban langsung di web dengan autosave.
- Maksimal tiga versi jawaban, riwayat versi, dan revisi tambahan dari guru.
- Permintaan bantuan murid pada draft aktif.
- Komentar, rubrik, nilai, kehadiran, jurnal, dan audit perubahan penting.
- Impor satu kali data penting dari Google Sheets ke PostgreSQL.

Modul P5 ditunda sampai kebutuhan dan kode sumber `LearningHubP5Core` tersedia.

## Stack

- Next.js App Router dan TypeScript.
- PostgreSQL di Neon.
- Drizzle ORM dan migrasi skema.
- Zod untuk validasi.
- Vercel untuk preview dan produksi.

## Branch dan rilis

- `dev` — pengembangan aktif menggunakan Google Antigravity dan Codex.
- `produksi` — kode stabil yang menjadi sumber deployment live Vercel.
- `main` — baseline repository, bukan sumber deployment.

Alur normal: kerjakan di `dev` → uji preview → tinjau perubahan → gabungkan ke `produksi` → buat tag rilis.

Versi mengikuti Semantic Versioning. Nomor versi nantinya disimpan pada aplikasi, ditampilkan di footer, dan tersedia melalui `/version`.

## Dokumentasi

- [Product specification](docs/product-spec.md)
- [Architecture](docs/architecture.md)
- [Design system](docs/design-system.md)
- [Agent rules](AGENTS.md)

BRD, PRD, FRD, dan SRS tidak dipisah. Kebutuhan bisnis, produk, fitur, dan sistem dirangkum dalam `product-spec.md` agar realistis dipelihara oleh solo developer.

## Batas integrasi

Aplikasi Apps Script dan spreadsheet lama tetap hidup secara independen. Tidak ada sinkronisasi dua arah, shared database, atau mekanisme cutover otomatis. Spreadsheet hanya menjadi sumber impor awal yang dijalankan secara eksplisit oleh admin.
