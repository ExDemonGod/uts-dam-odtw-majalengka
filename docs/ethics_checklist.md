# Data Ethics Checklist

Dataset: Daftar Potensi Objek Daya Tarik Wisata (ODTW) di Kabupaten Majalengka
Sumber: data.go.id | Diunduh: 2026-11-18

Catatan: checklist ini disusun dari prinsip umum etika data. Jika dosen menyediakan template resmi, isi dipindahkan ke template tersebut.

| No | Butir | Status | Bukti |
|---|---|---|---|
| 1 | Lisensi dan izin penggunaan | Belum pasti | Lisensi tertulis "tidak diketahui" saat unduh. Tindakan: cek halaman dataset dan catat lisensi resminya. Data dipakai hanya untuk tugas akademik. |
| 2 | Atribusi sumber | Terpenuhi | URL dan tanggal akses tercatat di `docs/provenance.csv`. Sumber dicantumkan di README. |
| 3 | Data pribadi (PII) langsung | Terpenuhi | Notebook Soal 2 memeriksa email, nomor telepon, dan NIK 16 digit: hasil nol. Kolom berisi nama wisata dan kode wilayah, bukan data orang. |
| 4 | Quasi-identifier | Dicatat | Kolom `alamat` (tingkat desa) dan kecamatan adalah level wilayah publik. Tidak ada kombinasi yang mengarah ke individu. |
| 5 | Kepatuhan robots.txt dan ketentuan situs | Terpenuhi | `docs/robots_check.txt` berisi hasil pemeriksaan `can_fetch` untuk halaman Wikipedia yang diambil. |
| 6 | Beban server dan rate limit | Terpenuhi | Setiap request memakai timeout 20 detik, jeda 2 detik, dan maksimal 3 retry dengan backoff. |
| 7 | Keamanan kredensial | Terpenuhi | Open-Meteo tidak memerlukan key. Tidak ada token di notebook. `.gitignore` memuat `.env`. |
| 8 | Akurasi dan kualitas | Dicatat | Tahun data tidak seragam (2020 sampai 2023). Ada 2 baris identik kecuali tahun (TALAGA PANCAR). Koordinat API adalah perkiraan pusat wilayah. |
| 9 | Penggunaan dan publikasi hasil | Terpenuhi | Hasil hanya untuk keperluan UTS. Tidak ada klaim bahwa data mewakili kondisi terkini tanpa verifikasi. |
