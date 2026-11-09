# Data penelitian aktif

Folder ini memuat input penelitian dan CSV validasi ekstraksi ulang dari notebook `analisis_scweat_kbji.ipynb`.

| Berkas | Fungsi |
| --- | --- |
| [kbji_profesional.csv](kbji_profesional.csv) | Sumber 501 entri Profesional, nama asli, hierarki, halaman, dan flag seleksi; disaring menjadi 415 jabatan |
| [istilah_gender.csv](istilah_gender.csv) | Delapan pasangan atribut gender dan metadata validasi |
| [kbji_2014.pdf](kbji_2014.pdf) | Dokumen sumber KBJI 2014 |
| [validasi/kbji_profesional_sumber.csv](validasi/kbji_profesional_sumber.csv) | Pemeriksaan silang kode dan nama asli; teks preprocessing lama tidak dipakai |
| [validasi/istilah_gender_historis.csv](validasi/istilah_gender_historis.csv) | Audit perbedaan lexical historis; subset lama tidak menjadi pilihan analisis |
| [validasi/kbji_profesional_ekstraksi_ulang.csv](validasi/kbji_profesional_ekstraksi_ulang.csv) | Keluaran baru 501 entri dari PDF: nama asli/analisis, token, hierarki, halaman/baris dan flag/alasan seleksi; 415 terpilih |

Isi kelima input tidak diubah ketika diganti nama. Kolom provenance pada input, termasuk `source_document` dengan label asli `KBJI 2014.pdf` dan versi ekstraksi historis, tetap dipertahankan. Path fisik PDF sekarang adalah `data/kbji_2014.pdf`. CSV ekstraksi ulang memakai nama berkas dan versi preprocessing baru; audit tersimpan di [`outputs/ekstraksi_kbji/2026-11-10_00-18-50`](../outputs/ekstraksi_kbji/2026-11-10_00-18-50/). CSV utama tetap menjadi input SC-WEAT.

Sebelas CSV yang tidak dibaca pipeline aktif berada di [arsip data tidak relevan](../archive/2026-10-14_penataan_berkas/README.md). Nama lama tetap disimpan dalam [manifest penataan](../archive/2026-10-14_penataan_berkas/manifest_penataan.csv) dan metadata run sebelumnya.
