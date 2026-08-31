# Implementation Roadmap

Roadmap ini membagi `v0.1.0` menjadi delivery kecil yang masing-masing dapat diuji dan ditinjau. Rencana rinci delivery berikutnya baru ditulis setelah delivery sebelumnya selesai agar mengikuti struktur kode yang benar-benar terbentuk.

## Delivery 1 — Foundation, Auth, and Academics

Rencana: `docs/superpowers/plans/2026-08-31-foundation-auth-academics.md`

Hasil yang dapat digunakan:

- Next.js berjalan di Node.js 24.
- Quality gate, test runner, database test, dan CI tersedia.
- Enam tabel identitas/akademik beserta migrasi.
- Login murid dan login guru/admin.
- Session cookie, lockout, role guard, dan isolasi kelas.
- Admin dapat mengelola periode, kelas, akun, enrollment, dan penugasan guru.
- Dashboard kosong yang benar untuk setiap peran.

## Delivery 2 — Meetings and Learning Content

Tabel: `meetings`, `learning_items`, `learning_item_versions`, `learning_outcomes`, dan `learning_item_outcomes`.

Hasil yang dapat digunakan:

- Guru membuat pertemuan dan materi.
- Builder blok sederhana untuk latihan, resume, refleksi, LKPD, dan asesmen.
- Versioning konten, publikasi, preview murid, dan arsip.
- Murid dapat membaca materi dan melihat aktivitas sesuai kelas.
- Template konten dapat disalin ke periode baru tanpa data operasional.

## Delivery 3 — Submissions and Assistance

Tabel: `student_activity_settings`, `submissions`, `submission_versions`, `help_requests`, dan `comments`.

Hasil yang dapat digunakan:

- Draft dan autosave dengan optimistic locking.
- Submit atomik dan idempotent.
- Maksimal tiga versi serta revisi tambahan.
- Riwayat versi immutable.
- Minta Bantuan, visibilitas draft sementara, dan komentar per blok.

## Delivery 4 — Review and Classroom Operations

Tabel: `reviews`, `attendance_records`, `journal_entries`, dan `audit_logs`.

Hasil yang dapat digunakan:

- Antrean pemeriksaan guru.
- Meja pemeriksaan, rubrik, nilai, keputusan, dan umpan balik per versi.
- Kehadiran dan jurnal guru.
- Audit perubahan penting tanpa log autosave massal.
- Dashboard guru/admin dengan data operasional.

## Delivery 5 — Import, Hardening, and Production Release

Hasil yang dapat digunakan:

- Pratinjau dan impor idempotent dari spreadsheet.
- Rehash PIN, mapping jawaban lama sebagai version 1, dan laporan baris bermasalah.
- Accessibility pass, security review, regression suite, dan smoke test.
- Neon production, Vercel `dev` preview, dan deployment `produksi`.
- Backup pra-impor/pra-migrasi, release `v0.1.0`, serta panduan rollback.

## Definition of Done setiap delivery

- Acceptance criteria delivery terpenuhi.
- Unit, integration, database, dan end-to-end test yang relevan lulus.
- `npm run lint`, `npm run typecheck`, `npm run test:run`, dan `npm run build` lulus.
- Dokumentasi dan nomor versi diperbarui bila perilaku publik berubah.
- Commit berada di `dev`; `produksi` hanya diperbarui setelah review dan preview berhasil.
