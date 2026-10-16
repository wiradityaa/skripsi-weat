# Catatan pengerjaan aktif

Tanggal menggunakan zona waktu Asia/Jakarta. Sumber spesifikasi adalah prompt start fresh yang diotorisasi pengguna pada 14 Oktober 2026; proposal revisi asli belum tersedia dalam repository. Riwayat eksperimen lama disimpan pada arsip dan tidak menjadi hasil aktif.

## 14 Oktober 2026 — penataan dan persiapan implementasi

**Perubahan dan alasan.** Arah aktif diselaraskan menjadi satu notebook, satu model fastText, tiga RM, `ddof=0`, permutasi eksak dua sisi 12.870, BH global 415 pengujian, RM 2 deskriptif, dan LOPO tanpa inferensi tambahan. Eksperimen lama diarahkan ke `archive/2026-10-14_eksperimen_lama/`; CSV/PDF/model input tetap di lokasi semula. Pedoman dan hasil lama dipertahankan sebagai sejarah, bukan dibaca kembali untuk menggantikan eksekusi baru.

**Keputusan sumber.** `data/kbji_profesional.csv` dipilih setelah pemeriksaan struktur: 501 baris, nama asli pada `occupation_name`, kode string, hierarki, halaman sumber, serta keputusan seleksi. Nama, kode, dan seleksinya telah dibandingkan dengan `data/validasi/kbji_profesional_sumber.csv` mentah dan sesuai. Teks preprocessing lama tidak dipakai. Jejak Profesional merujuk `data/kbji_2014.pdf`, halaman PDF 161–327.

**Audit preparasi aktual.** Sumber memuat 85 residual dan satu entri dengan token atribut; final 415 jabatan. Residual mencakup kode non-`.99` `2353.00` dan `2359.00`; pemeriksaan memakai akhiran kode atau nama `lainnya|ybdi|ytdl`. Kebocoran atribut adalah `2635.09 Organisator Kesejahteraan Perempuan`. Distribusi kelompok final 21–26 adalah 138, 74, 42, 29, 26, dan 106.

**Pemeriksaan sumber.** Audit read-only menemukan PDF 752 halaman dan memeriksa langsung delapan contoh, termasuk kode berekor nol, slash, titik akhir, dan entri leakage; seluruh contoh cocok dengan CSV. Semua halaman tercetak pada jejak input sama dengan nomor halaman PDF dikurangi 40. Pemeriksaan contoh ini tidak merupakan verifikasi menyeluruh semua nama atau sertifikasi salinan resmi KBJI.

**Keputusan preprocessing.** NFKC, lowercase, petik lurus/keriting tunggal/ganda dihapus, Unicode `Cc`/`Cf` menjadi spasi, pembatas `(`, `)`, `/`, `,`, `.` menjadi spasi, whitespace dirapikan, tanda hubung dan punctuation lain dipertahankan. Kata di dalam kurung tetap dipakai. Dengan aturan ini audit preparasi menghasilkan 42 nama satu token dan 373 multikata, sesuai acuan. Kebijakan ini merupakan keputusan implementasi, terutama perlakuan kurung/slash yang belum dirinci pada proposal.

**Lexical set dan model.** Lexical set dibaca dari `data/istilah_gender.csv`, delapan pasangan sesuai acuan; `pair_id` hasil aktif berbasis istilah dan penomoran lama disimpan sebagai jejak tanpa mengubah input. Model tunggal lokal `models/cc.id.300.bin` berukuran 7.236.808.873 byte; identitas SHA-256, dimensi, dan versi aktual direkam pada run.

**Dokumentasi.** Acuan, metode, kamus, dan catatan pengerjaan dibuat di `docs/`. README dan AGENTS diselaraskan dengan metode aktif setelah pedoman lama disalin ke arsip. Dependensi aktif dipin sesuai versi yang diperiksa pada environment eksekusi.

**Status eksekusi: selesai.** [snapshot notebook sebelum penataan](../archive/2026-10-14_penataan_berkas/notebooks/analisis_scweat_kbji_sebelum_penamaan.ipynb) memiliki 45 cell, termasuk 23 cell kode yang dijalankan berurutan dari kernel baru tanpa error; execution count 1–23 dan seluruh output disimpan. Run `scweat_20261014T224206_381549+0700` dimulai 14 Oktober 2026 pukul 22:42:06 WIB, selesai 22:44:04 WIB. `manifest_run.json` berstatus `complete`; durasi sampai QC akhir yang dicatat notebook 115,03 detik.

## 14 Oktober 2026 — hasil run baru dan validasi akhir

**RM 1.** Pengukuran aktual menghasilkan 415 baris: 105 positif, 310 negatif, nol skor nol. Mean effect size −0,170002, median −0,167623, rentang −0,935940 sampai 0,905523. P-value mentah minimum 0,060140; tidak ada hasil signifikan mentah atau setelah BH global. Seluruh adjusted p-value konstan 0,9992229992229992, dan grafik mempertahankan kondisi tersebut. Tidak signifikan tidak ditafsirkan sebagai bukti tidak ada asosiasi. Tabel hasil.

**RM 2.** Ringkasan enam kelompok berasal dari skor individual; semua mean/median negatif. Mean tertinggi kelompok 21 (−0,100979), terendah kelompok 23 (−0,341921). SD populasi antarjabatan dalam masing-masing kelompok berkisar 0,206298–0,312722; penyebaran terbesar pada kelompok 26. Seluruh kelompok mempunyai nol penolakan BH global. Perbedaan jumlah anggota ditangani melalui denominator kelompok dan perbandingan tetap deskriptif. Tabel lengkap.

**RM 3.** Detail LOPO mempunyai 3.320 kombinasi unik, ringkasan jabatan 415 baris, ringkasan pasangan delapan baris. Sebanyak 178 jabatan mempertahankan arah pada semua delapan kondisi dan 237 memiliki setidaknya satu pembalikan positif–negatif. `pemuda–pemudi` memberikan mean perubahan absolut terbesar 0,267038 (77 pembalikan); `jejaka–gadis` mean 0,233948 dengan pembalikan terbanyak 155. Perubahan maksimum 0,777140 pada `2654.03 Sutradara Film` ketika `jejaka–gadis` dikeluarkan. Spearman 0,926615–0,988565; perubahan arah tetap diperiksa meskipun korelasi tinggi. Tidak ada inferensi tambahan, kategori stabil/tidak stabil, atau penghapusan pasangan otomatis. Ringkasan pasangan.

**Model dan OOV aktual.** Model mempunyai 2.000.000 istilah vocabulary, dimensi 300, SHA-256 `46e6e4c3975c2386f76c608c896cb36fa9c90f9d7f8212988c7eeb5ff41d235f`. Seluruh vektor target/atribut valid. Lima jabatan memiliki token OOV dan 410 tidak; token `endodontist`, `paedodontist`, `prosthodontist`, `subtitler`, `unani` tetap menggunakan subword. Lexical historis v5 hanya dibaca untuk audit perbedaan dan dihash sebagai input audit, tanpa mengubah delapan pasangan aktif.

**Validasi numerik.** 16 QC PASS: cakupan/seleksi/nama mentah, delapan pasangan/16 istilah, vektor300/norma/finite, 36 cosine dibandingkan rumus umum, enumerator unik dan komplemen, sembilan jabatan melalui referensi skalar untuk seluruh permutasi, BH independen global, pertukaran label, hitung ulang setelah pengacakan baris/atribut, SD/ringkasan/LOPO, round-trip 21 tabel ekspor, dan keluaran lengkap. Pemeriksaan mencocokkan identitas kode dan mempertahankan `2111.10`/`2162.00`. SHA-256 source cell notebook cocok dengan manifest; output eksekusi tidak masuk fingerprint kode.

**Visual dan artefak.** Tersedia 37 gambar statis masing-masing PNG/SVG: 20 visual utama dan 17 halaman heatmap lengkap; HTML heatmap hanya pelengkap. Metadata semua PNG diverifikasi 300 dpi. Semua 20 visual utama diperiksa melalui contact sheet dan gambar individual yang padat; halaman pertama dan terakhir heatmap lengkap juga diperiksa langsung. Ukuran/SHA-256 100 berkas output cocok dengan manifest (manifest sendiri tidak masuk daftar checksum output). Tabel audit memuat caption aktual, schema, durasi, dan jejak sumber.

**Finalisasi repository.** README, AGENTS, acuan, metode, kamus, requirements, dan progres diselaraskan dengan run ini. Manifest arsip berisi 84 catatan (82 move, dua copy), seluruh SHA-256 cocok, 72 rename tracked mempertahankan konten. Data dan model tidak berubah terhadap Git HEAD; perubahan lokal pengguna dipertahankan. Tidak dibuat commit. Proposal asli dan verifikasi klasifikasi menyeluruh tetap merupakan keterbatasan sumber, tanpa menghalangi eksekusi ini.


## 14 Oktober 2026 — penataan nama dan pemisahan data tidak relevan

**Instruksi pengguna.** Nama aktif disederhanakan dan awalan eksperimen dihapus; berkas yang tidak dipakai dikelompokkan. Instruksi ini memperbarui ketentuan awal untuk mempertahankan nama/lokasi input.

**Penataan.** `data/kbji_profesional.csv`, `data/istilah_gender.csv`, dan `data/kbji_2014.pdf` menjadi input utama. `data/validasi/kbji_profesional_sumber.csv` dan `data/validasi/istilah_gender_historis.csv` dipakai untuk pemeriksaan silang/audit. Sebelas CSV lainnya diarsipkan: empat demo sintetis dan tujuh versi lama. Semua 16 berkas data mempertahankan SHA-256 isinya. Notebook menjadi `notebooks/analisis_scweat_kbji.ipynb`; dokumen pendukung memakai nama huruf kecil di `docs/`, termasuk `progres_penelitian.md`.

**Keterlacakan.** [Manifest penataan](../archive/2026-10-14_penataan_berkas/manifest_penataan.csv) mencatat 22 move dan satu copy snapshot dengan hash saat pemindahan. [Snapshot notebook run pertama](../archive/2026-10-14_penataan_berkas/notebooks/analisis_scweat_kbji_sebelum_penamaan.ipynb) mempertahankan kode/output sebelumnya. Isi CSV, label sumber asli, dan metadata ekstraksi historis tidak diubah. Metadata path pada run pertama tetap merekam lokasi saat run itu dibuat; pemetaan ke path sekarang tersedia dalam manifest penataan.

**Eksekusi ulang.** [Notebook aktif](../notebooks/analisis_scweat_kbji.ipynb) dijalankan dari folder `notebooks/` dengan kernel baru. Run [`2026-10-14_23-08-25`](../outputs/analisis_utama/2026-10-14_23-08-25/) dimulai 2026-10-14T23:08:25.053065+07:00 dan selesai 2026-10-14T23:10:20.882278+07:00; durasi sampai QC akhir 112.89 detik. Seluruh 23 cell kode selesai tanpa error, 16 QC PASS, ukuran hasil 415/6/3320/415/8, dan 37 gambar PNG/SVG. [Manifest run](../outputs/analisis_utama/2026-10-14_23-08-25/audit/manifest_run.json) berstatus `complete`.

**Kesetaraan hasil.** Semua 9 CSV pada `tables/` identik byte demi byte dengan run pertama; semua 37 PNG juga identik. Penataan mengubah nama/path dan format ID run, tanpa mengubah hasil atau metode penelitian. README, pedoman, konfigurasi notebook, serta tautan dokumentasi memakai nama baru. Tidak dibuat commit.

## Batas sumber dan reproduksi

Proposal asli belum tersedia, sehingga nomor persamaan dan ketentuan penelitian mengikuti spesifikasi prompt. Input ekstraksi lama dipakai sebagai data yang diaudit, bukan sebagai salinan resmi KBJI yang telah diverifikasi ulang menyeluruh. Model berasal dari biner lokal pretrained; SHA-256 pada manifest menjadi identitas tepatnya.

Run hasil menggunakan `outputs/analisis_utama/<run_id>/` baru, lengkap dengan `tables/`, `figures/`, dan `audit/`. Run pada halaman ini telah memenuhi eksekusi seluruh sel dari kernel bersih, hasil tiap RM, ekspor, dan pemeriksaan akhir. Tidak ada tahap analisis yang masih terhalang pada run ini. Eksekusi ulang tetap membentuk folder baru; [`LATEST_RUN.txt`](../outputs/analisis_utama/LATEST_RUN.txt) menunjuk run lengkap terbaru. Hasil historis tidak dipakai untuk mengisi hasil aktif.


Catatan keterlacakan: folder output run pertama `scweat_20261014T224206_381549+0700` sudah tidak tersedia pada working tree saat persiapan push 16 Oktober 2026. Snapshot notebooknya tetap tersimpan di arsip penataan berkas; hasil aktif tersedia pada run `2026-10-14_23-08-25`. Rujukan historis di atas dipertahankan sebagai teks.
