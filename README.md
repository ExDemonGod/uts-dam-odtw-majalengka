# UTS Data Acquisition & Management: ODTW Kabupaten Majalengka

Topik: pariwisata. Mata kuliah Data Acquisition & Management, Kelas E, INSTIKI.

## Isi repo
- `notebook/UTS_DAM_Akuisisi_ODTW_Majalengka.ipynb`: notebook utama (jalankan di Google Colab)
- `data/raw/`: data mentah, tidak diubah
- `data/processed/`: hasil penggabungan dan metadata
- `docs/provenance.csv`: URL, tanggal akses, lisensi, dan MD5 tiap sumber
- `docs/ethics_checklist.md`: Data Ethics Checklist
- `docs/robots_check.txt`: bukti pemeriksaan robots.txt

## Sumber
1. Berkas: data.go.id, dataset ODTW Kabupaten Majalengka (diunduh 2026-11-18)
2. API: Open-Meteo (tanpa key)
3. Scraping: Wikipedia Indonesia, Kabupaten Majalengka (tabel kecamatan)

## Menjalankan
1. Buka notebook di Google Colab (File > Open notebook > GitHub, atau unggah).
2. Isi `CONTACT_EMAIL` dengan email kampus.
3. Runtime > Run all.
4. Jika ingin mendorong hasil ke GitHub dari Colab, simpan token GitHub di Colab Secrets dengan nama `GITHUB_TOKEN`, lalu isi `GH_REPO`.

Token dan key tidak pernah ditulis di notebook.
