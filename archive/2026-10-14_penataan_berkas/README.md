# Arsip penataan berkas — 14 Oktober 2026

Penataan ini mengikuti instruksi pengguna untuk memberi nama deskriptif pada input aktif dan mengelompokkan data lainnya. Data di folder `data_tidak_relevan/` tidak digunakan dalam pipeline penelitian aktif.

- `data_tidak_relevan/demo_sintetis/`: empat CSV data/atribut demo sintetis.
- `data_tidak_relevan/versi_lama/`: tujuh CSV ekstraksi, lexical set, atau hasil seleksi versi lama yang tidak dibaca notebook aktif.
- `notebooks/analisis_scweat_kbji_sebelum_penamaan.ipynb`: snapshot notebook yang dieksekusi pada run pertama; menjaga keterlacakan hash sumber kode pada manifest run tersebut.

[Manifest penataan](manifest_penataan.csv) mencatat path awal/akhir, peran, status tracked, dan SHA-256 sebelum/sesudah pemindahan. Lima input aktif dipindah/ganti nama tanpa perubahan isi. Dokumen dan kode aktif kemudian diperbarui untuk menggunakan path baru; kolom `integrity_at_move` menyatakan kesamaan saat pemindahan, sebelum pembaruan kode/dokumentasi.

Nama eksperimen pada data arsip dipertahankan sebagai jejak historis. Berkas ini tidak berarti datanya salah; hanya tidak relevan terhadap pipeline aktif. Notebook historis tidak dijadikan entry point atau dipakai menggantikan perhitungan baru.
