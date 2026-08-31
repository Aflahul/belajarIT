# belajarIT

Pengembangan ulang aplikasi pembelajaran menggunakan Next.js, Vercel, dan Neon PostgreSQL. Aplikasi ini berdiri sendiri dan tidak menggantikan atau menyinkronkan aplikasi Google Apps Script yang sudah berjalan.

## Branch

- `dev` — pengembangan aktif menggunakan Google Antigravity dan Codex.
- `produksi` — kode stabil untuk deployment live Vercel.
- `main` — branch inisialisasi repository, bukan sumber deployment.

Alur perubahan: kerjakan di `dev` → uji melalui Vercel Preview → pull request ke `produksi` → rilis.

## Versioning

Rilis menggunakan Semantic Versioning dan Git tag:

- `v0.x.y` untuk pengembangan awal.
- `v1.0.0` untuk versi stabil pertama.
- Patch untuk perbaikan, minor untuk fitur kompatibel, dan major untuk perubahan tidak kompatibel.

## Dokumentasi ringkas

Dokumentasi solo-dev akan dipusatkan pada:

- `docs/product-spec.md` — gabungan kebutuhan bisnis, produk, fitur, dan persyaratan sistem.
- `docs/architecture.md` — arsitektur, skema database, autentikasi, dan aliran data.
- `AGENTS.md` — aturan kerja untuk Antigravity dan Codex.

Dokumen BRD, PRD, FRD, dan SRS tidak dipisah agar tidak menambah beban pemeliharaan.
