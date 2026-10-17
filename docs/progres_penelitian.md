# Progres penelitian

## Diagnostik penjelas — 18 Oktober 2026

**D1–D5 selesai** pada run [`2026-10-18_02-57-12`](../outputs/diagnostik/2026-10-18_02-57-12/), parent utama [`2026-10-14_23-08-25`](../outputs/analisis_utama/2026-10-14_23-08-25/). Hanya 13 cell kode diagnostik dijalankan dari kernel baru; 45 cell utama dan output sebelumnya dipertahankan. Model tidak dimuat ulang dan pipeline utama tidak dieksekusi ulang. Manifest diagnostik `complete`, durasi 11,72 detik, 16 QC PASS, sepuluh gambar PNG/SVG. Hasil, metode utama, dan pointer utama tetap; pointer diagnostik terpisah.

- **D1:** mean selisih cosine `pemuda–pemudi` +0,080524 dan `jejaka–gadis` −0,090329; 404 jabatan menunjukkan kedua komponen berlawanan sekaligus. Selisih merupakan komponen pembilang, bukan effect size pasangan tunggal.
- **D2:** 237 jabatan pernah membalik versus 178 tidak, mencakup 416 kejadian. Median besar skor awal masing-masing 0,111856 dan 0,383373, tanpa cutoff netralitas.
- **D3:** 3.320 komponen LOPO cocok sumber dalam toleransi numerik. Studi kasus memperlihatkan pembalikan `Sutradara Film` tanpa `jejaka–gadis` dan perubahan besar tanpa pembalikan `Juru Katalog` tanpa `pemuda–pemudi`; tanda skor ditentukan pembilang selama SD positif.
- **D4:** 48 kelompok–pasangan; pengeluaran `pemuda–pemudi` memberi mean delta negatif pada 6/6 kelompok, `jejaka–gadis` positif pada 6/6. Proporsi pembalikan terbesar 47,83% pada kelompok 21 tanpa `jejaka–gadis`, memakai denominator 138 jabatan kelompok tersebut.
- **D5:** tiga contoh dipilih transparan menurut besar skor; seluruh 12.870 statistik per contoh direkonstruksi. P-value 0,060140; 0,719814; 0,999223 cocok baseline; adjusted p tetap berasal dari BH global, tanpa pengujian tambahan.

Seluruh diagnostik bersifat deskriptif dan tidak mengubah target, model, preprocessing, pasangan baseline, `ddof=0`, permutasi, FDR, atau LOPO. [README](../README.md), [metode](metodologi.md), [kamus](kamus_data.md), dan [catatan pengerjaan](catatan_pengerjaan.md) memuat tabel/visual, provenance, cara menjalankan hanya diagnostik, dan batas interpretasi. Snapshot source disimpan pada audit run. Tidak ada D1–D5 yang masih terhalang; Full Run All notebook 68 cell tidak diklaim dijalankan ulang.

## Start fresh — 14 Oktober 2026

Arah penelitian aktif mengikuti prompt proposal revisi yang diotorisasi pengguna: **Analisis Asosiasi Gender dan Sensitivitas Skor SC-WEAT terhadap Pemilihan Pasangan Istilah Gender pada Klasifikasi Baku Jabatan Indonesia (KBJI) 2014 Menggunakan fastText Bahasa Indonesia.**

Titik masuk aktif adalah [`notebooks/analisis_scweat_kbji.ipynb`](../notebooks/analisis_scweat_kbji.ipynb), satu model `cc.id.300.bin`, `ddof=0`, permutasi eksak dua sisi 12.870, BH-FDR global 415 pengujian, perbandingan enam kelompok deskriptif, dan LOPO tanpa inferensi tambahan. Tiga RM, batas interpretasi, dan pemetaan hasil dijelaskan pada [`docs/acuan_proposal.md`](../docs/acuan_proposal.md).

Penataan telah mempertahankan seluruh eksperimen lama. [Manifest pemindahan](../archive/2026-10-14_eksperimen_lama/manifest_pemindahan.csv) memuat 82 move dan dua copy, seluruh SHA-256 sebelum/sesudah cocok. Seluruh 72 berkas tracked yang dipindahkan menggunakan `git mv`. Pada pengarsipan awal input dipertahankan. Penataan nama berikutnya memindahkan input tanpa mengubah isi sesuai instruksi pengguna; tidak dibuat commit.

Audit preparasi aktual menggunakan `data/kbji_profesional.csv`: 501 baris, 85 residual, satu kebocoran token atribut, 415 jabatan. Distribusi 21–26 adalah 138/74/42/29/26/106; preprocessing baru menghasilkan 42 satu token dan 373 multikata. Nama, kode, dan seleksi raw telah dibandingkan dengan `data/validasi/kbji_profesional_sumber.csv` dan sesuai. Model serta lexical set tetap input aktual; embedding dan skor lama tidak digunakan.

**Status aktif: implementasi dan eksekusi selesai.** Run [`2026-10-14_23-08-25`](../outputs/analisis_utama/2026-10-14_23-08-25/) menjalankan seluruh 23 cell kode dari kernel baru tanpa error, output disimpan, dan 16 QC PASS. Keluaran aktual: 415 hasil utama, enam ringkasan kelompok, 3.320 detail LOPO, 415 ringkasan jabatan, delapan ringkasan pasangan, serta 37 gambar PNG/SVG termasuk 17 halaman heatmap lengkap.

RM 1 menghasilkan 105 skor positif, 310 negatif, nol skor nol; mean −0,170002 dan median −0,167623. Tidak ada penolakan p-value mentah maupun BH global. RM 2 menunjukkan mean tertinggi kelompok 21 (−0,100979), terendah kelompok 23 (−0,341921), dengan perbandingan deskriptif dan keputusan FDR global. RM 3 menunjukkan 178 jabatan mempertahankan arah seluruh kondisi, 237 mengalami setidaknya satu pembalikan; mean perubahan absolut terbesar pada `pemuda–pemudi` (0,267038), sedangkan pembalikan terbanyak pada `jejaka–gadis` (155). Spearman 0,926615–0,988565; tidak ditambah uji signifikansi LOPO.

Angka ini berasal dari eksekusi baru, terpisah dari hasil historis. Ringkasan, tautan output, dan batas interpretasi tersedia di [`README.md`](../README.md); validasi dan keputusan lengkap pada [`docs/catatan_pengerjaan.md`](../docs/catatan_pengerjaan.md). Manifest run berstatus `complete`; tidak ada kendala eksekusi yang tersisa. Proposal asli dan verifikasi klasifikasi menyeluruh tetap belum tersedia.

## Riwayat sebelum start fresh

Riwayat rinci beserta angka eksperimen lama dipindahkan utuh ke [`VERSI_DAN_PROGRESS_legacy.md`](../archive/2026-10-14_eksperimen_lama/reports/VERSI_DAN_PROGRESS_legacy.md). Pedoman dan README lama tersedia pada [`reports/`](../archive/2026-10-14_eksperimen_lama/reports/README_legacy.md). Seluruh artefak ini merupakan sejarah pengembangan dan bukan hasil final skripsi aktif.

| Keluarga historis | Arah pada saat itu | Status sekarang |
| --- | --- | --- |
| `demo_v1`, `demo_v2` | Data sintetis dan pemeriksaan awal alur kode | Arsip, bukan bukti empiris |
| `test_v1`, `test_v2` | Ekstraksi KBJI dan SC-WEAT fastText, sensitivitas/audit lama | Arsip; CSV mentah terpilih tetap input dengan audit baru |
| `test_v3` | Perbandingan model fastText | Arsip, di luar pipeline aktif |
| `test_v4` | IndoBERT dan LPBS | Arsip, di luar pipeline aktif |
| `test_v5` | Preprocessing KBJI, IndoBERT/w-SEAT/robustness | Notebook/hasil diarsip; CSV sumber 501 baris tetap input aktif yang diaudit |

Notebook/output historis tidak dibaca sebagai sumber skor, p-value, embedding, atau interpretasi baru. Perubahan metode dan preprocessing dapat mengubah hasil; validasi aktif berasal dari hitung ulang dan identitas data, bukan kecocokan dengan angka historis.


## Penataan nama dan data — 14 Oktober 2026

Notebook aktif memakai `notebooks/analisis_scweat_kbji.ipynb`. Tiga input utama bernama `kbji_profesional.csv`, `istilah_gender.csv`, dan `kbji_2014.pdf`; dua input pemeriksaan berada di `data/validasi/`. Sebelas CSV yang tidak dibaca dipindahkan ke [arsip data tidak relevan](../archive/2026-10-14_penataan_berkas/README.md). Nama dokumen pendukung disederhanakan di `docs/`, dan folder run baru menggunakan tanggal/waktu yang mudah dibaca.

Run `2026-10-14_23-08-25` dijalankan dari folder `notebooks/` melalui kernel baru. Seluruh 23 cell kode selesai tanpa error dan 16 QC PASS. Semua 9 tabel ilmiah identik byte demi byte dengan run pertama; isi data dan metode tetap sama. Kelima input dan sebelas CSV arsip mempertahankan hash isinya. [Manifest penataan](../archive/2026-10-14_penataan_berkas/manifest_penataan.csv) memuat 22 pemindahan dan satu salinan snapshot.

Run pertama `scweat_20261014T224206_381549+0700` dan [snapshot notebooknya](../archive/2026-10-14_penataan_berkas/notebooks/analisis_scweat_kbji_sebelum_penamaan.ipynb) dipertahankan sebagai riwayat eksekusi. Model resmi mempertahankan nama `cc.id.300.bin`.


Catatan keterlacakan: folder output run pertama `scweat_20261014T224206_381549+0700` sudah tidak tersedia pada working tree saat persiapan push 16 Oktober 2026. Snapshot notebooknya tetap tersimpan di arsip penataan berkas; hasil aktif tersedia pada run `2026-10-14_23-08-25`. Rujukan historis di atas dipertahankan sebagai teks.
