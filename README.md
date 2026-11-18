# UTS Data Acquisition & Management — Daftar ODTW Kabupaten Majalengka

Ujian Tengah Semester **Data Acquisition & Management** (Kelas E, Semester Ganjil 2026)
Program Studi Informatika — INSTIKI
Dosen: Dr. I Gede Totok Suryawan, S.Kom., M.T.

| | |
|---|---|
| **Nama** | [NAMA LENGKAP] |
| **NIM** | [NIM] |
| **Notebook (Colab)** | [URL Colab] |
| **Repositori** | https://github.com/[USERNAME]/uts-dam-odtw-majalengka |

---

## 1. Topik

**Objek Daya Tarik Wisata (ODTW) di Kabupaten Majalengka, Jawa Barat, dalam konteks kecamatan dan kondisi cuaca.**

Sektor yang dipilih adalah **pariwisata**. Pertanyaan yang ingin dijawab dengan data gabungan: di kecamatan mana ODTW terkonsentrasi, dan bagaimana kondisi cuacanya pada periode prakiraan?

---

## 2. Tiga Sumber Data

| # | Cara akuisisi | Sumber | Isi | Lisensi | Berkas mentah |
|---|---|---|---|---|---|
| 1 | **Unduh berkas dataset terbuka** | [data.go.id — Daftar Potensi ODTW di Kabupaten Majalengka](https://data.go.id/dataset/dataset/daftar-potensi-objek-daya-tarik-wisata-odtw-di-kabupaten-majalengka) | 250 baris × 13 kolom, tahun 2020–2023 | Tidak diketahui (lihat `docs/data_ethics_checklist.csv`) | `data/raw/dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv` |
| 2 | **API** | [Open-Meteo](https://open-meteo.com/) (Geocoding API + Forecast API), tanpa API key | Koordinat kecamatan dan prakiraan cuaca harian 7 hari (suhu maks/min, curah hujan) | CC BY 4.0 (atribusi Open-Meteo.com) | `data/raw/api/forecast_*.json` |
| 3 | **Web scraping halaman statis** | [Wikipedia ID — Kabupaten Majalengka](https://id.wikipedia.org/wiki/Kabupaten_Majalengka) | Tabel kecamatan (nama dan atribut wilayah) | CC BY-SA 4.0 | `data/raw/scrape/wikipedia_kabupaten_majalengka.html`, `data/raw/scrape/wikipedia_tabel_kecamatan.csv`, `data/raw/scrape/bukti_robots_id_wikipedia.txt` |

Tidak ada data dari Booking.com, Agoda, Traveloka, tiket.com, Airbnb, Tripadvisor, Google Maps, maupun media sosial.

### Provenance dan integritas (MD5)

Berkas mentah di `data/raw/` disimpan **apa adanya** dan tidak diedit. Setiap berkas dicatat di `docs/provenance.csv` (URL, tanggal akses, lisensi, ukuran, MD5).

| Berkas | Ukuran (byte) | MD5 |
|---|---:|---|
| `dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv` | 32.843 | `8ed945d4c9603d756f2053d7054fd073` |
| `api/forecast_*.json` | [isi dari provenance.csv] | [isi dari provenance.csv] |
| `scrape/wikipedia_kabupaten_majalengka.html` | [isi dari provenance.csv] | [isi dari provenance.csv] |
| `scrape/wikipedia_tabel_kecamatan.csv` | [isi dari provenance.csv] | [isi dari provenance.csv] |

Nilai API dan scraping diisi dari `docs/provenance.csv` setelah notebook dijalankan.

---

## 3. Struktur Repositori

```
uts-dam-odtw-majalengka/
├── notebook/
│   └── UTS_DAM_ODTW_Majalengka.ipynb  # notebook utama (semua sel sudah dijalankan)
├── README.md
├── .gitignore                         # .env dan kredensial tidak ikut ter-commit
├── data/
│   ├── raw/                           # data mentah, tidak diubah
│   │   ├── dftr_ptns_bjk_dy_trk_wst_dtw_d_kbptn_mjlngk_2.csv
│   │   ├── api/                       # JSON respons Open-Meteo
│   │   └── scrape/                    # HTML, tabel, dan bukti robots.txt
│   └── processed/
│       ├── api_cuaca_normalized.csv
│       └── odtw_gabungan.csv
└── docs/
    ├── provenance.csv
    ├── data_ethics_checklist.csv
    ├── metadata_gabungan.json
    └── bukti_halaman_portal_data_go_id.html
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

1. **Geocoding:** enam kecamatan dengan ODTW terbanyak dicari koordinatnya, dengan hasil difilter ke Jawa Barat.
2. **Forecast:** prakiraan harian 7 hari (suhu maks, suhu min, curah hujan) untuk tiap koordinat. Zona waktu `Asia/Jakarta` (WIB).
3. **Normalisasi JSON → tabel:** satu baris per kecamatan per tanggal, minimal 30 baris (6 kecamatan × 7 hari = 42 baris). Hasil: `data/processed/api_cuaca_normalized.csv`.

Penanganan request: `Retry(total=3, backoff_factor=1.5)` untuk status 429, 500, 502, 503, 504. `timeout=20` detik, jeda 2 detik antar-request, dan status non-200 dilaporkan dengan pesan jelas. Tidak ada API key.

### Soal 4 — Web scraping etis

- Target: halaman statis Wikipedia ID, Kabupaten Majalengka.
- `robots.txt` diperiksa dengan `urllib.robotparser`. Hasil `can_fetch` dan baris Disallow yang relevan disimpan di `data/raw/scrape/bukti_robots_id_wikipedia.txt`.
- Hanya satu halaman yang diambil, dengan User-Agent berisi email kampus, timeout, dan jeda.
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

---

## 5. Cara Menjalankan Ulang

**Google Colab**

1. Buka notebook dari tautan Colab di atas.
2. Isi `CONTACT_EMAIL` di sel setup dengan email kampus.
3. Jalankan semua sel (*Runtime → Run all*). Pastikan berkas CSV sudah ada di `data/raw/`.

**Lokal**

```bash
git clone https://github.com/[USERNAME]/uts-dam-odtw-majalengka.git
cd uts-dam-odtw-majalengka
pip install pandas requests beautifulsoup4 lxml
jupyter notebook notebook/UTS_DAM_ODTW_Majalengka.ipynb
```

---

## 6. Keamanan dan Etika

- Tidak ada API key atau token di notebook. Open-Meteo tidak memerlukan key. Token GitHub, jika dipakai untuk push, disimpan di Colab Secrets dengan nama `GITHUB_TOKEN`.
- Setiap request memakai **timeout**, **User-Agent berisi email kampus**, dan **jeda** antar-request.
- Data mentah tidak dimodifikasi. Perbaikan hanya dilakukan di `data/processed/`.

## 7. Keterbatasan

- Prakiraan cuaca Open-Meteo bersifat prakiraan model dan koordinatnya adalah titik perkiraan per kecamatan, bukan pengukuran stasiun.
- Dataset ODTW memiliki tahun pencatatan yang tidak seragam (2020–2023), sehingga jumlah per tahun tidak sebanding secara langsung.
- Tabel kecamatan dari Wikipedia mengikuti data sumber Wikipedia pada saat diambil.

## 8. Atribusi

- Dataset ODTW: Pemerintah Kabupaten Majalengka melalui data.go.id.
- Data cuaca: [Open-Meteo.com](https://open-meteo.com/), CC BY 4.0.
- Tabel kecamatan: [Wikipedia ID](https://id.wikipedia.org/), CC BY-SA 4.0. Karena share-alike, turunan data dari Wikipedia di repositori ini mengikuti lisensi yang sama.
