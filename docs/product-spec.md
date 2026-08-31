# Product Specification

Status: draft tervalidasi untuk review  
Target awal: `v0.1.0`  
Pemilik produk dan pengembang: solo developer

## 1. Ringkasan

belajarIT adalah aplikasi pembelajaran Informatika yang memindahkan alur inti dari Google Apps Script dan Google Sheets ke aplikasi web mandiri. Murid membaca materi, mengerjakan latihan, resume, refleksi, LKPD, dan asesmen langsung di web. Guru memantau kebutuhan bantuan, memeriksa versi jawaban terbaru, memberikan nilai dan umpan balik, serta membuka revisi tambahan jika diperlukan.

Aplikasi lama tetap berjalan secara independen. Aplikasi baru tidak melakukan sinkronisasi dan tidak menjadi pengganti otomatis.

## 2. Masalah yang diselesaikan

- Struktur spreadsheet berkembang menjadi 27 sheet, termasuk sheet kosong dan struktur yang saling tumpang tindih.
- Draft, autosave, revisi resmi, komentar, dan hasil penilaian belum memiliki batas data yang jelas.
- Guru membutuhkan antrean perhatian: bantuan aktif, jawaban belum diperiksa, dan revisi baru.
- Murid membutuhkan tempat belajar dan mengerjakan yang nyaman melalui HP.
- Sistem harus dapat berkembang lintas semester tanpa menggandakan seluruh riwayat operasional.

## 3. Sasaran produk

- Menyediakan alur belajar yang jelas dan ramah untuk murid.
- Menyediakan pemeriksaan dan bantuan yang efisien untuk guru.
- Menjaga jawaban, versi, nilai, dan riwayat tetap konsisten.
- Mendukung beberapa periode akademik serta arsipnya.
- Menekan biaya dan beban pemeliharaan solo developer.

## 4. Bukan sasaran fase pertama

- Menghentikan atau mengubah aplikasi Apps Script.
- Sinkronisasi Google Sheets dengan Neon.
- Modul P5, kelompok, kontribusi individu, checkpoint, dan blueprint game.
- Upload file dan penyimpanan bukti internal; fase pertama hanya menerima tautan HTTPS.
- Microservice, multi-tenant antarsekolah, chat real-time penuh, analitik lanjutan, dan aplikasi offline penuh.
- Aplikasi native Android/iOS.

## 5. Pengguna dan izin

### Murid

Login memakai kelas, nomor absen, dan PIN. Murid hanya dapat membaca materi kelasnya, mengubah draft sendiri, mengirim jawaban, meminta bantuan, serta melihat versi dan umpan balik sendiri.

### Guru

Login memakai email atau username dan password. Guru hanya dapat mengakses kelas yang ditugaskan. Guru dapat mengelola pertemuan dan aktivitas, melihat draft saat bantuan aktif, mengomentari dan menilai versi jawaban, mengatur kehadiran dan jurnal, serta membuka revisi tambahan.

### Admin

Admin dapat mengelola periode, kelas, akun, penugasan guru, impor, dan audit. Admin memiliki akses seluruh kelas untuk dukungan operasional.

## 6. Ruang lingkup fase pertama

### 6.1 Akademik dan akun

- Satu periode akademik aktif pada satu waktu.
- Periode lama tetap dapat dibaca sebagai arsip.
- Pembuatan periode baru dapat menyalin pertemuan, materi, template LKPD, dan template asesmen.
- Murid, jawaban, nilai, kehadiran, dan jurnal tidak ikut disalin.
- Akun dapat dinonaktifkan tanpa menghapus riwayat.

### 6.2 Pertemuan dan konten

- Guru membuat pertemuan, jadwal, judul, dan status akses.
- Satu pertemuan dapat memuat materi dan beberapa aktivitas.
- Jenis item: materi, latihan, resume, refleksi, LKPD, dan asesmen.
- Perubahan item yang telah terbit menghasilkan versi konten baru.

### 6.3 Builder aktivitas

Blok fase pertama:

- Instruksi.
- Jawaban singkat.
- Jawaban panjang.
- Pilihan tunggal.
- Pilihan jamak.
- Angka.
- URL.

Setiap blok dapat memiliki label, deskripsi, status wajib, indikator, dan konfigurasi penilaian. Urutan diubah dengan kontrol naik/turun; drag-and-drop bebas bukan kebutuhan fase pertama.

### 6.4 Draft dan autosave

- Draft disimpan otomatis setelah perubahan berhenti sejenak.
- Autosave tidak dihitung sebagai revisi resmi.
- Murid melihat status penyimpanan terakhir.
- Konflik dua tab dideteksi memakai `lock_version`.
- Fase pertama tidak menjamin pengeditan offline setelah tab ditutup.

### 6.5 Permintaan bantuan

- Draft privat secara default.
- Murid dapat membuka permintaan bantuan beserta pertanyaan singkat.
- Selama permintaan berstatus terbuka, guru kelas dapat melihat draft yang terus diperbarui dan memberi komentar.
- Guru tidak dapat mengubah jawaban murid.
- Setelah bantuan ditutup, draft kembali privat; percakapan tetap tersimpan.

### 6.6 Pengiriman dan revisi

- Pengiriman pertama menghasilkan versi 1.
- Batas standar adalah tiga versi terkirim.
- Guru dapat memberikan tambahan revisi per murid dan aktivitas.
- Versi terkirim bersifat immutable.
- Versi lama dapat dilihat guru dan murid, tetapi tidak dapat diedit.
- Submit ganda dengan idempotency key yang sama hanya menghasilkan satu versi.

### 6.7 Pemeriksaan

- Guru melihat antrean bantuan aktif, jawaban belum diperiksa, dan revisi baru.
- Meja pemeriksaan menampilkan daftar murid, jawaban, riwayat versi, rubrik, nilai, komentar, dan tindak lanjut.
- Versi terbaru dibuka secara default.
- Komentar dan penilaian terhubung ke versi yang diperiksa.
- Nilai versi lama tidak menimpa hasil versi terbaru.

### 6.8 Kehadiran, jurnal, dan audit

- Kehadiran dicatat per murid dan pertemuan.
- Jurnal menyimpan fakta, observasi, refleksi, dan tindak lanjut.
- Audit hanya menyimpan perubahan penting, bukan setiap autosave.

### 6.9 Impor awal

- Spreadsheet menjadi sumber read-only.
- Admin melihat pratinjau jumlah valid, duplikat, dan bermasalah.
- Impor berjalan atomik atau dapat aman diulang.
- `Murid` dan `Akun_Demo` dipetakan ke akun/profil; PIN di-hash ulang dan teks asli tidak disimpan.
- `Respons_LKPD.server_revision` diperlakukan sebagai counter autosave, bukan jumlah revisi resmi.
- Jawaban terkirim yang ada diimpor sebagai submission version 1.

## 7. Alur layar

### Murid

Navigasi: Beranda, Belajar, Tugas, Riwayat. Beranda mengutamakan pertemuan aktif, tugas, bantuan, umpan balik, dan revisi. Ruang tugas menggabungkan instruksi, progress, blok jawaban, autosave, bantuan, submit, dan jumlah versi.

### Guru

Dashboard mengutamakan item yang membutuhkan perhatian. Ruang kelas memiliki Ringkasan, Pertemuan, Materi & Aktivitas, Murid, Kehadiran, dan Jurnal. Builder memakai langkah sederhana. Meja pemeriksaan memakai tiga panel di desktop dan tab di HP.

### Admin

Admin mengelola periode, kelas, pengguna, penugasan, impor, status aktivitas, audit, dan konfigurasi penting.

## 8. Aturan bisnis

- Waktu database disimpan dalam UTC dan ditampilkan sebagai `Asia/Jakarta`.
- Tenggat diverifikasi memakai waktu server.
- Pengecualian tenggat dan revisi diberikan per murid.
- Aktivitas dengan jawaban tidak dapat dihapus; hanya diarsipkan.
- Submit membuat snapshot jawaban dan audit dalam satu transaksi.
- Jawaban menyimpan referensi versi konten yang dikerjakan.
- Guru tidak dapat melihat draft privat melalui UI maupun query langsung aplikasi.

## 9. Persyaratan nonfungsional

- Mobile-first untuk murid; desktop-first responsif untuk guru/admin.
- Seluruh otorisasi diperiksa di server.
- Password dan PIN tidak disimpan sebagai teks biasa.
- Aplikasi menangani submit ganda dan konflik autosave.
- State loading, kosong, error, tersimpan, dan akses ditolak tersedia.
- Flow kritis dilindungi unit, integration, database, dan end-to-end test.
- Log tidak menyimpan secret atau isi lengkap jawaban.

## 10. Kriteria penerimaan fase pertama

Fase pertama dianggap siap digunakan ketika:

1. Ketiga peran dapat login dan hanya melihat data yang diizinkan.
2. Admin dapat membuat periode/kelas dan mengimpor roster dengan pratinjau.
3. Guru dapat menerbitkan pertemuan, materi, dan aktivitas berbasis blok.
4. Murid dapat mengisi, autosave, meminta bantuan, dan submit dari HP.
5. Batas tiga versi serta revisi tambahan bekerja secara atomik.
6. Guru dapat membandingkan versi, mengomentari, dan menilai versi terbaru.
7. Kehadiran, jurnal, dan audit perubahan penting tersedia.
8. Seluruh flow kritis lulus pengujian dan smoke test deployment.
9. Apps Script lama tetap berjalan tanpa perubahan.

## 11. Keputusan yang ditunda

- Desain dan implementasi modul P5.
- Penyimpanan upload internal.
- Notifikasi email/WhatsApp/push.
- Analitik lintas periode lanjutan.
- Otomasi backup eksternal di luar kemampuan paket database yang dipakai.
