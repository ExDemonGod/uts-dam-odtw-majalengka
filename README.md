# UTS Data Acquisition & Management — Daftar ODTW Kabupaten Majalengka

Ujian Tengah Semester **Data Acquisition & Management** (Kelas E, Semester Ganjil 2026)
Program Studi Informatika — INSTIKI
Dosen: Dr. I Gede Totok Suryawan, S.Kom., M.T.

| | |
|---|---|
| **Nama** | Si Ngurah Rai Krisna Pranajaya |
| **NIM** | 2501010375 |
| **Notebook pipeline (Colab)** | https://colab.research.google.com/drive/10sMNJ1IaslC-j3wTyHi5xAHE-JZ4Qbme#scrollTo=PcDKFmazX6hv |
| **Repositori** | https://github.com/ExDemonGod/uts-dam-odtw-majalengka |

---

## 1. Topik

**Objek Daya Tarik Wisata (ODTW) di Kabupaten Majalengka, Jawa Barat, dalam konteks kecamatan dan kondisi cuaca.**

Sektor yang dipilih adalah **pariwisata**. Pertanyaan yang ingin dijawab dengan data gabungan: di kecamatan mana ODTW terkonsentrasi, dan bagaimana kondisi cuacanya pada periode prakiraan?

---

## 2. Tiga Sumber Data

| # | Cara akuisisi | Sumber | Isi | Lisensi | Berkas mentah |
|---|---|---|---|---|---|
| 1 | **Unduh berkas dataset terbuka** | [data.go.id — Daftar Potensi ODTW di Kabupaten Majalengka](https://data.go.id/dataset/dataset/daftar-potensi-objek-daya-tarik-wisata-odtw-di-kabupaten-majalengka) | 250 baris × 13 kolom, tahun 2020–2023 | Tidak tercantum di halaman portal (lihat `docs/data_ethics_checklist.csv`) | `data/raw/dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv` |
| 2 | **API** | [Open-Meteo](https://open-meteo.com/) (Geocoding API + Forecast API), tanpa API key | Koordinat kecamatan dan prakiraan cuaca harian 7 hari (suhu maks/min, curah hujan) | CC BY 4.0 (atribusi Open-Meteo.com) | `data/raw/api/forecast_*.json` (salinan juga di cache) |
| 3 | **Web scraping halaman statis** | [Wikipedia ID — Kabupaten Majalengka](https://id.wikipedia.org/wiki/Kabupaten_Majalengka) | Tabel kecamatan (nama dan atribut wilayah) | CC BY-SA 4.0 | `data/raw/scrape/wikipedia_kabupaten_majalengka.html`, `data/raw/scrape/wikipedia_tabel_kecamatan.csv` |

Tidak ada data dari Booking.com, Agoda, Traveloka, tiket.com, Airbnb, Tripadvisor, Google Maps, maupun media sosial.

### Provenance dan integritas (MD5)

Berkas mentah di `data/raw/` disimpan **apa adanya** dan tidak diedit. Setiap berkas dicatat di `docs/provenance.csv` (URL, tanggal akses, lisensi, ukuran, MD5).

Provenance dibuat ulang setelah pemeriksaan akhir menemukan bahwa MD5 halaman portal (`bukti_halaman_portal_data_go_id.html`) tidak lagi cocok dengan catatan awal, karena halaman tersebut diunduh ulang lewat cache. Catatan yang sekarang mencerminkan keadaan berkas saat ini.

| Berkas | Ukuran (byte) | MD5 |
|---|---:|---|
| `dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv` | 32.843 | `8ed945d4c9603d756f2053d7054fd073` |
| `api/forecast_*.json` | 588 | f872e2f0fe1fb51d64eff90b058ff0f6 |
| `scrape/wikipedia_kabupaten_majalengka.html` | 470.014 | e9783a6fb819fb99019c2322d13a27f0 |
| `scrape/wikipedia_tabel_kecamatan.csv` | 4.578 | e0cbb2f37be5619aeb471ea57610a335 |

Nilai API dan scraping diisi dari `docs/provenance.csv` setelah notebook dijalankan.

---

## 3. Struktur Repositori

```
uts-dam-odtw-majalengka/
├── notebook/
│   ├── UTS_DAM_ODTW_Majalengka.ipynb           # pipeline Soal 1–5
│   └── UTS_DAM_ODTW_Majalengka_lanjutan.ipynb  # cache & refresh, visualisasi, verifikasi, push
├── README.md
├── .gitignore                                  # cache/ dan kredensial tidak ikut ter-commit
├── data/
│   ├── raw/                                    # data mentah, tidak diubah
│   │   ├── dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv
│   │   ├── bukti_halaman_portal_data_go_id.html
│   │   ├── api/                                # JSON respons Open-Meteo
│   │   └── scrape/                             # HTML dan tabel kecamatan
│   └── processed/
│       ├── api_cuaca_normalized.csv
│       └── odtw_gabungan.csv
├── cache/                                      # salinan respons sementara (tidak di-commit)
│   └── manifest.json
└── docs/
    ├── provenance.csv
    ├── data_ethics_checklist.csv
    ├── metadata_gabungan.json
    ├── verifikasi_akhir.csv
    └── figures/                                # grafik hasil visualisasi (PNG)
```

---

## 4. Ringkasan Pengerjaan per Soal

### Soal 1 — Pembacaan berkas, identifikasi data, estimasi storage

Berkas dibaca dengan `pd.read_csv(sep=",", encoding="utf-8", na_values=["()", "-", ""], dtype={...kode wilayah sebagai string...})`. Kode wilayah dibaca sebagai teks agar angka 0 di depan dan titik tidak hilang.

| Aspek | Hasil |
|---|---|
| Jenis data | **Terstruktur** (CSV tabular, skema kolom tetap) |
| Jumlah baris × kolom | 250 × 13 |
| Satuan | LOKASI (setiap baris satu objek wisata) |
| Wilayah | Kabupaten Majalengka, Jawa Barat, 24 kecamatan (menurut BPS) |
| Waktu | Tahun 2020 (192 baris), 2021 (12), 2022 (37), 2023 (9) |
| Ukuran berkas | 32.843 byte (±32,08 KB) |
| Nilai hilang | 0 |

**Estimasi storage 1 tahun**

Asumsi: tiap bulan menyimpan satu salinan berkas dengan ukuran sama, ditambah cadangan 30% untuk backup dan metadata.

- Berkas CSV: 32.843 B × 12 = 394.116 B, lalu × 1,3 = **512.351 B (±500 KB, ±0,49 MB)**

**Temuan kualitas data**

- **Duplikasi antar-tahun:** `TALAGA PANCAR` (id 209 dan 243) memiliki alamat dan kecamatan yang sama, hanya berbeda tahun (2022 dan 2023). Dicatat, tidak dihapus, karena data mentah tidak boleh diubah.
- **Nama objek sama di kecamatan sama:** 6 pasangan nama dengan id berbeda (misalnya `KERAJINAN ROTAN` di Sindangwangi). Isinya berbeda, sehingga tidak dianggap duplikasi.
- Kolom `alamat` berisi tingkat desa, bukan alamat pribadi.

### Soal 2 — Etika data

- **Lisensi dan ketentuan portal:** halaman data.go.id tidak mencantumkan lisensi. Status ditulis **"Belum pasti"** dan pemakaian dibatasi untuk tugas akademik. Bukti pemeriksaan halaman ada di `docs/bukti_halaman_portal_data_go_id.html`.
- **PII:** dipindai dengan pola regex (email, telepon, NIK 16 digit). Hasilnya **nol** pada semua kolom teks.
- **Quasi-identifier:** `bps_nama_kecamatan`, `alamat`, dan `tahun`. Data berupa objek wisata dan lokasinya, bukan individu, sehingga risiko re-identifikasi rendah.
- **Data Ethics Checklist 9 butir:** `docs/data_ethics_checklist.csv`, lengkap dengan bukti tiap butir.

### Soal 3 — Akuisisi lewat API (Open-Meteo)

Portal data.go.id menyediakan halaman metadata, tetapi tidak menyediakan API data baris untuk dataset ini. Karena itu dipakai **Open-Meteo**, API publik tanpa key.

1. **Pemilihan kecamatan:** enam kecamatan dengan ODTW terbanyak dipilih dari dataset.
2. **Geocoding:** koordinat dicari untuk tiap kecamatan, dengan hasil difilter ke Jawa Barat. Kecamatan yang koordinatnya tidak ditemukan dilewati dan dicatat di output.
3. **Forecast:** prakiraan harian 7 hari (suhu maks, suhu min, curah hujan) untuk tiap koordinat. Zona waktu `Asia/Jakarta` (WIB).
4. **Normalisasi JSON → tabel:** satu baris per kecamatan per tanggal. Hasil pipeline: **35 baris** untuk **5 kecamatan** (7 hari per kecamatan), karena satu kecamatan ([nama kecamatan yang dilewati]) tidak berhasil di-geocode. Hasil: `data/processed/api_cuaca_normalized.csv`.

Penanganan request: `Retry(total=3, backoff_factor=1.5)` untuk status 429, 500, 502, 503, 504. `timeout=20` detik, jeda 2 detik antar-request, dan status non-200 dilaporkan dengan pesan jelas. Tidak ada API key.

### Soal 4 — Web scraping

- Target: halaman statis Wikipedia ID, Kabupaten Majalengka.
- Hanya satu halaman yang diambil, dengan User-Agent berisi email kampus, timeout, dan jeda antar-request.
- Tabel diambil dengan BeautifulSoup (mencari tabel `wikitable` yang header-nya memuat kata "kecamatan"), lalu diubah menjadi DataFrame dengan `pd.read_html`.

### Soal 5 — Penggabungan data

Sumber yang digabung: **(1) berkas ODTW** dan **(3) tabel kecamatan hasil scraping**. Kunci gabung adalah nama kecamatan yang dinormalisasi (huruf besar, tanpa tanda baca, spasi tunggal).

| Merge | Kiri | Kanan | Kunci | how | `validate` |
|---|---|---|---|---|---|
| 1 | Berkas ODTW (250 baris) | Tabel kecamatan (scraping) | `kunci_kecamatan` | left | `many_to_one` |

Relasi banyak ODTW ke satu kecamatan, sehingga `many_to_one` dipakai agar tabel kanan dipastikan unik per kunci.

**Bukti tidak ada duplikasi**

| Pemeriksaan | Hasil |
|---|---:|
| Baris ODTW sebelum merge | 250 |
| Baris setelah merge | 250 |
| Duplikat id di hasil | 0 |
| Duplikat penuh di hasil | 0 |
| Persentase baris ter-match | [isi setelah notebook dijalankan] |

Keluaran: `data/processed/odtw_gabungan.csv` beserta `docs/metadata_gabungan.json` (sumber, kunci gabung, jenis gabung, jumlah baris input dan output, MD5 keluaran).

### Tambahan — Cache dan refresh

Notebook lanjutan menyimpan respons API dan halaman portal ke folder `cache/`, dengan masa berlaku per sumber:

| Grup sumber | Masa berlaku | Yang disimpan |
|---|---|---|
| `api` | 6 jam | Forecast Open-Meteo per kecamatan |
| `halaman` | 7 hari | Halaman portal data.go.id |

- Data dari cache dipakai selama masih segar. Setelah kedaluwarsa, sumber diunduh ulang.
- `FORCE_REFRESH = True` memaksa unduh ulang semua sumber.
- `cache/manifest.json` mencatat waktu unduh dan SHA-256 tiap berkas. Pemeriksaan verifikasi memastikan SHA-256 berkas cache masih cocok dengan manifest.
- Folder `cache/` tidak di-commit (tercantum di `.gitignore`).

### — Visualisasi

Lima grafik dibuat dari data hasil pipeline dan disimpan ke `docs/figures/`:

1. `01_top10_kecamatan.png`: sepuluh kecamatan dengan ODTW terbanyak
2. `02_odtw_per_tahun.png`: jumlah ODTW per tahun
3. `03_nilai_hilang.png`: persentase nilai hilang per kolom
4. `04_suhu_7_hari.png`: prakiraan suhu maks dan min 7 hari per kecamatan
5. `05_curah_hujan_7_hari.png`: total curah hujan 7 hari per kecamatan

### — Verifikasi akhir

Semua pemeriksaan dijalankan ulang dari berkas di disk, bukan dari variabel sesi. Hasilnya disimpan di `docs/verifikasi_akhir.csv`.

| Kelompok | Pemeriksaan |
|---|---|
| Integritas output | MD5 output sama dengan metadata; jumlah baris input = output = data aktual; `id` unik; tidak ada duplikat penuh; persentase match ≥ 90%; kolom wajib tersedia; tahun dalam rentang wajar |
| PII | Tidak ada email, nomor telepon, atau NIK pada kolom teks |
| Data API | Minimal 30 baris; tepat 7 tanggal per kecamatan; suhu minimum ≤ maksimum; curah hujan tidak negatif dan tidak kosong |
| Provenance | Semua berkas raw tercatat; MD5 berkas raw cocok dengan provenance |
| Etika | Checklist berisi 9 butir; tidak ada butir berstatus "Perlu tindakan" |
| Cache | SHA-256 berkas cache cocok dengan manifest |
| Rahasia | Tidak ada token, kunci, atau kredensial di URL pada berkas proyek |

Hasil terakhir: [isi jumlah pemeriksaan lulus, misalnya 17/17].

## 6. Keamanan dan Etika

- Tidak ada API key atau token di notebook. Open-Meteo tidak memerlukan key.
- Token GitHub disimpan di Colab Secrets dengan nama `GITHUB_TOKEN`. Token tidak ditulis ke URL atau perintah, dan diteruskan lewat credential helper agar tidak tampil di log.
- Push dilakukan tanpa force push. Remote diambil dan di-rebase terlebih dahulu.
- Setiap request memakai **timeout**, **User-Agent berisi email kampus**, dan **jeda** antar-request.
- Data mentah tidak dimodifikasi. Perbaikan hanya dilakukan di `data/processed/`.
- Setelah tugas selesai, token GitHub dicabut atau dihapus dari Secrets.

## 7. Keterbatasan

- Prakiraan cuaca Open-Meteo bersifat prakiraan model dan koordinatnya adalah titik perkiraan per kecamatan, bukan pengukuran stasiun.
- Prakiraan hanya mencakup 7 hari dan hanya untuk 5 kecamatan yang koordinatnya berhasil ditemukan.
- Dataset ODTW memiliki tahun pencatatan yang tidak seragam (2020–2023), sehingga jumlah per tahun tidak sebanding secara langsung.
- Tabel kecamatan dari Wikipedia mengikuti data sumber Wikipedia pada saat diambil.
- Cache bersifat sementara. Data cuaca yang sudah lebih dari 6 jam akan diunduh ulang saat notebook dijalankan.

## 8. Atribusi

- Dataset ODTW: Pemerintah Kabupaten Majalengka melalui data.go.id.
- Data cuaca: [Open-Meteo.com](https://open-meteo.com/), CC BY 4.0.
- Tabel kecamatan: [Wikipedia ID](https://id.wikipedia.org/), CC BY-SA 4.0. Karena share-alike, turunan data dari Wikipedia di repositori ini mengikuti lisensi yang sama.
