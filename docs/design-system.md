# Design System

## 1. Tujuan

belajarIT memakai dua ekspresi visual dalam satu produk:

- **Comic Neobrutalism Edukatif** untuk murid: ramah, energik, mudah dikenali, tetapi tetap nyaman untuk membaca dan mengerjakan.
- **Academic Calm** untuk guru/admin: tenang, efisien, padat informasi, dan tidak bergaya komik.

Keduanya berbagi fondasi aksesibilitas, spacing, type scale, icon, form behavior, dan status semantics.

## 2. Prinsip bersama

- Hierarki lebih penting daripada dekorasi.
- Satu layar memiliki satu tindakan utama yang jelas.
- Warna tidak menjadi satu-satunya penanda status.
- Form selalu memiliki label; placeholder bukan pengganti label.
- State wajib: default, hover, focus, active, disabled, loading, success, warning, error.
- Fokus keyboard terlihat jelas.
- Teks panjang memakai lebar baca nyaman dan line-height longgar.
- Animasi singkat serta menghormati `prefers-reduced-motion`.

## 3. Fondasi

### Typography

- Body: sans-serif yang sangat terbaca.
- Display murid: sans-serif ekspresif untuk judul pendek saja.
- Guru/admin tetap menggunakan sans-serif netral.
- Body minimum murid: 16px.
- Jangan memakai uppercase untuk paragraf atau instruksi panjang.

### Spacing

Gunakan skala `4, 8, 12, 16, 24, 32, 48, 64` px. Hindari nilai acak kecuali diperlukan untuk alignment optik.

### Radius

- Murid: 0–12px tergantung karakter komponen; panel utama cenderung tegas.
- Guru/admin: 8–12px konsisten.

### Status semantics

- Info: biru.
- Sukses/selesai: hijau atau mint.
- Peringatan/tenggat: kuning/oranye.
- Error/ditolak: merah.
- Revisi: merah muda/ungu dengan label teks.

## 4. Comic Neobrutalism Edukatif

### Token utama

```css
--neo-ink: #111827;
--neo-paper: #fff7df;
--neo-white: #fffef9;
--neo-yellow: #ffd84d;
--neo-blue: #74c7ff;
--neo-mint: #7ee7c4;
--neo-pink: #ff8fab;
--neo-border: 3px solid var(--neo-ink);
--neo-shadow-lg: 6px 6px 0 var(--neo-ink);
--neo-shadow-sm: 3px 3px 0 var(--neo-ink);
```

### Penggunaan

Gunakan gaya komik pada:

- Navigasi dan header murid.
- Judul pertemuan.
- Kartu tugas dan progress.
- Badge status.
- Tombol utama.
- Panel bantuan dan umpan balik penting.

Jangan gunakan berlebihan pada:

- Setiap paragraf materi.
- Setiap input dalam LKPD panjang.
- Tabel data.
- Modal konfirmasi kompleks.
- Seluruh background sekaligus.

### Komponen murid

- **Comic card:** border 3px, hard shadow, satu warna aksen, judul singkat.
- **Reading panel:** background putih hangat, border lebih tenang, tanpa shadow besar.
- **Answer block:** panel stabil dengan nomor blok, label, bantuan, dan error dekat input.
- **Status badge:** ikon + teks; tidak mengandalkan warna.
- **Primary action:** kontras tinggi, shadow mengecil saat ditekan.
- **Help panel:** warna biru/mint; menunjukkan status guru dapat melihat draft.

Dekorasi seperti burst, garis gerak, stiker, atau speech bubble dibatasi maksimal satu atau dua aksen per viewport.

## 5. Academic Calm

### Token awal

```css
--calm-bg: #f6f8fb;
--calm-surface: #ffffff;
--calm-text: #172033;
--calm-muted: #667085;
--calm-border: #dce2ea;
--calm-primary: #3155a4;
--calm-primary-soft: #eaf0ff;
--calm-success: #18794e;
--calm-warning: #a15c00;
--calm-danger: #b42318;
--calm-shadow: 0 4px 16px rgb(23 32 51 / 8%);
```

### Penggunaan

- Background netral dan surface putih.
- Border 1px serta shadow lembut.
- Warna primer untuk tindakan utama dan navigasi aktif.
- Warna status hanya untuk data yang memerlukan perhatian.
- Tabel memiliki header jelas, sticky header jika panjang, filter, pencarian, dan empty state.
- Dashboard mengutamakan antrean kerja, bukan grafik dekoratif.

### Meja pemeriksaan

- Desktop: daftar murid, jawaban, dan panel review.
- Tablet/HP: ketiganya menjadi tab.
- Versi terbaru diberi label jelas; versi lama read-only.
- Komentar muncul dekat blok jawaban terkait.
- Aksi simpan nilai dan buka revisi tidak diletakkan berdekatan tanpa pembeda.

## 6. Layout responsif

- Murid dimulai dari lebar 360px.
- Navigasi murid menggunakan bottom navigation pada HP dan sidebar pada desktop.
- Guru/admin memakai sidebar di desktop dan drawer pada layar kecil.
- Tabel besar boleh berubah menjadi card list pada HP jika kolom utama tetap terlihat.
- Sticky action tidak boleh menutupi input atau keyboard virtual.

## 7. Bahasa dan microcopy

Gunakan istilah murid:

- `Jawaban tersimpan`, bukan `Draft persisted`.
- `Versi 2 dari 3`, bukan `Revision counter`.
- `Minta bantuan`, bukan `Open assistance request`.
- `Perlu diperbaiki`, bukan `Rejected` jika konteksnya revisi belajar.

Pesan error menjelaskan tindakan berikutnya. Contoh: `Jawaban belum tersimpan. Periksa koneksi, lalu coba lagi.`

## 8. Aksesibilitas

- Target sentuh minimum 44×44px.
- Kontras teks memenuhi WCAG AA.
- Semua field memiliki label dan pesan error terhubung.
- Dialog mengelola focus masuk dan kembali.
- Status autosave diumumkan secara tidak mengganggu oleh assistive technology.
- Ikon selalu memiliki label atau teks pendamping untuk tindakan penting.

## 9. Batas pengembangan

- Jangan menambahkan mode gelap pada fase pertama.
- Jangan membuat library komponen generik di luar kebutuhan layar yang sedang dibangun.
- Jangan menggunakan banyak font dekoratif.
- Jangan membuat ilustrasi AI sebagai ketergantungan fungsi utama.
- Screenshot atau mockup bukan sumber token; perubahan token harus ditulis kembali di dokumen ini.
