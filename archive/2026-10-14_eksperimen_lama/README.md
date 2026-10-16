# Arsip eksperimen historis — 14 Oktober 2026

Folder ini menyimpan eksperimen sebelum penataan *start fresh* berdasarkan prompt proposal revisi. Notebook, skor, p-value, ringkasan, dan interpretasi di sini merupakan hasil historis dan **tidak menjadi acuan hasil aktif skripsi**.

Pipeline aktif tersedia melalui [README proyek](../../README.md) dan [notebook utama](../../notebooks/01_analisis_scweat_kbji.ipynb). Pipeline aktif harus menghitung embedding, cosine, skor, permutasi, FDR, dan sensitivitas dari input/model aktual; hasil dalam arsip ini tidak boleh dipakai sebagai pengganti eksekusi baru.

## Isi

- `notebooks/`: seluruh notebook demo dan eksperimen `demo_v*`/`test*_v*` dari root repository, termasuk notebook lokal pengguna yang belum tracked.
- `outputs/`: seluruh tabel, audit, dan metadata hasil lama, dengan struktur relatif semula dipertahankan.
- `reports/`: [riwayat progres](reports/VERSI_DAN_PROGRESS_legacy.md) yang dipindahkan serta salinan [README lama](reports/README_legacy.md) dan [AGENTS lama](reports/AGENTS_legacy.md) sebelum pedoman aktif diselaraskan.
- `imports/`: ZIP dan isi ekstraksi eksperimen v3 lokal pengguna. Seluruh struktur dan konten dipertahankan utuh.
- [manifest_pemindahan.csv](manifest_pemindahan.csv): sumber, tujuan, jenis berkas, alasan, status tracked, operasi move/copy, serta SHA-256 sebelum/sesudah untuk setiap berkas yang dipindahkan atau disalin.

Input dalam `data/` dan model dalam `models/` tetap di lokasi semula. Folder `.git`, konfigurasi, dan history Git tidak dipindahkan. Berkas tracked dipindahkan dengan `git mv`; pengarsipan tidak membuat commit.

Salinan dokumen legacy dapat memiliki tautan relatif atau instruksi lama yang merekam keadaan historis. Pedoman aktif adalah dokumen di root dan `docs/`, bukan pedoman legacy. Arsip ini tidak menjamin hasil eksperimen lama memenuhi proposal revisi.
