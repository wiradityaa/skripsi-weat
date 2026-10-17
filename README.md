# Analisis Asosiasi Gender dan Sensitivitas SC-WEAT pada KBJI 2014

**Analisis Asosiasi Gender dan Sensitivitas Skor SC-WEAT terhadap Pemilihan Pasangan Istilah Gender pada Klasifikasi Baku Jabatan Indonesia (KBJI) 2014 Menggunakan fastText Bahasa Indonesia.**

Proyek ini mengukur asosiasi intrinsik gender pada representasi 415 nama jabatan Profesional KBJI 2014 dalam satu model pretrained fastText Bahasa Indonesia. Jabatan adalah unit pengukuran dan pengujian; enam Subgolongan Pokok menjadi unit ringkasan deskriptif. Positif berarti relatif lebih dekat ke atribut laki-laki, negatif relatif ke perempuan, dan nol menunjukkan perbedaan relatif nol menurut kebijakan numerik. Hasil tidak menjadi bukti langsung diskriminasi, komposisi tenaga kerja, atau kelayakan gender suatu pekerjaan.

## Status start fresh — 14 Oktober 2026

Repository telah ditata mengikuti prompt proposal revisi yang diotorisasi pengguna. [Arsip eksperimen lama](archive/2026-10-14_eksperimen_lama/README.md) menyimpan 12 notebook, 60 output, satu laporan progres, satu ZIP, dan delapan berkas ekstraksi. Sebanyak 82 berkas dipindahkan dan dua pedoman lama disalin; [manifest](archive/2026-10-14_eksperimen_lama/manifest_pemindahan.csv) memuat 84 catatan dengan SHA-256 sebelum/sesudah yang sama. Seluruh 72 berkas tracked yang dipindahkan menggunakan `git mv`; tidak dibuat commit. Input aktif kemudian diberi nama deskriptif sesuai instruksi pengguna; pemetaan path lama tersedia pada manifest penataan. Model mempertahankan nama resmi.

**Implementasi dan eksekusi analisis utama selesai.** Bagian utama notebook valid menurut `nbformat`; seluruh 23 cell kodenya dijalankan berurutan dari kernel baru tanpa error dan output disimpan. Run **`2026-10-14_23-08-25`** dimulai 14 Oktober 2026 pukul 23:08:25 WIB dan selesai 23:10:20 WIB; [manifest](outputs/analisis_utama/2026-10-14_23-08-25/audit/manifest_run.json) berstatus `complete`, seluruh **16 QC PASS**. Tersedia 415 hasil individual, enam ringkasan kelompok, 3.320 detail LOPO, 415 ringkasan jabatan, delapan ringkasan pasangan, dan 37 gambar dalam PNG/SVG.

Semua embedding, cosine, skor, p-value, FDR, serta sensitivitas dihitung dari input dan model aktual pada run baru. Angka eksperimen lama tidak menjadi hasil baru. Proposal asli tidak tersedia dalam repository, sehingga [acuan implementasi](docs/acuan_proposal.md) merangkum spesifikasi dari prompt, bukan salinan naskah proposal.

**Diagnostik D1–D5 selesai pada 18 Oktober 2026.** Run [`2026-10-18_02-57-12`](outputs/diagnostik/2026-10-18_02-57-12/) menjalankan 13 cell kode diagnostik dari kernel baru, dengan **16 QC PASS** dan sepuluh gambar PNG/SVG. Sumbernya run utama `2026-10-14_23-08-25`; model tidak dimuat ulang dan pipeline utama tidak dieksekusi ulang. Output 45 cell utama, folder sumber, dan pointer utama dipertahankan.

## Tiga rumusan masalah

1. **RM 1:** bagaimana arah, besar effect size, dan signifikansi asosiasi gender pada 415 nama jabatan?
2. **RM 2:** bagaimana perbandingan deskriptif distribusi effect size pada enam Subgolongan Pokok melalui jumlah, mean, median, SD, rentang, proporsi arah, dan jumlah hasil signifikan setelah BH global?
3. **RM 3:** bagaimana perubahan skor, arah, dan peringkat saat setiap pasangan gender dikeluarkan bergantian melalui leave-one-pair-out (LOPO)?

RM 2 merupakan analisis utama tersendiri tanpa uji beda inferensial. LOPO merupakan diagnostik deskriptif tanpa p-value atau FDR tambahan.

## Struktur dan titik masuk

```text
notebooks/analisis_scweat_kbji.ipynb   seluruh pipeline aktif
data/                                 CSV analisis dan PDF sumber bernama deskriptif
  validasi/                           dua CSV pemeriksaan silang
models/cc.id.300.bin                   model tunggal, dipertahankan
outputs/analisis_utama/<run_id>/
  tables/                             hasil penuh CSV
  figures/                            PNG 300 dpi dan SVG, heatmap lengkap
  audit/                              jejak preparasi, vektor, validasi, manifest
outputs/diagnostik/<diagnostic_run_id>/
  tables/                             diagnostik D1–D5 dari run utama lengkap
  figures/                            gambar diagnostik PNG 300 dpi dan SVG
  audit/                              input/hash, validasi, manifest diagnostik
docs/                                 acuan, metode, kamus, catatan pengerjaan
archive/2026-10-14_eksperimen_lama/     eksperimen historis
archive/2026-10-14_penataan_berkas/     11 CSV tidak relevan dan snapshot notebook
```

Notebook utama: [`notebooks/analisis_scweat_kbji.ipynb`](notebooks/analisis_scweat_kbji.ipynb). Riwayat arah penelitian: [`docs/progres_penelitian.md`](docs/progres_penelitian.md). Pedoman pengembangan: [`AGENTS.md`](AGENTS.md).

## Tata nama dan pengelompokan data

Input aktif memakai nama berdasarkan fungsinya, tanpa awalan eksperimen. [Panduan data](data/README.md) menjelaskan kelima berkas yang dibaca notebook:

| Lokasi | Fungsi |
| --- | --- |
| `data/kbji_profesional.csv` | Data jabatan sumber untuk seleksi dan analisis |
| `data/istilah_gender.csv` | Delapan pasangan atribut gender |
| `data/kbji_2014.pdf` | PDF sumber |
| `data/validasi/kbji_profesional_sumber.csv` | Pemeriksaan silang nama dan kode asli |
| `data/validasi/istilah_gender_historis.csv` | Audit perbedaan lexical historis |

Sebelas CSV lainnya dikelompokkan dalam [arsip data tidak relevan](archive/2026-10-14_penataan_berkas/README.md): empat CSV demo sintetis dan tujuh CSV versi lama. Sebutan tidak relevan berarti tidak digunakan pada pipeline aktif. [Manifest penataan](archive/2026-10-14_penataan_berkas/manifest_penataan.csv) menyimpan pemetaan nama lama/baru dan hash sebelum/sesudah pemindahan. Isi input tetap sama; label provenance historis di dalam CSV dipertahankan.

Notebook aktif bernama `analisis_scweat_kbji.ipynb`; dokumen pendukung memakai nama huruf kecil di `docs/`. Folder run baru memakai format `YYYY-MM-DD_HH-MM-SS` waktu Asia/Jakarta. Run sebelumnya dan snapshot notebook sebelum penataan dipertahankan untuk keterlacakan.

## Sumber data dan audit cakupan

Input utama [`data/kbji_profesional.csv`](data/kbji_profesional.csv) memuat 501 baris, nama asli pada `occupation_name`, identitas hierarki, halaman sumber, dan flag seleksi. Nama/kode/seleksi telah diperiksa terhadap [`data/validasi/kbji_profesional_sumber.csv`](data/validasi/kbji_profesional_sumber.csv) mentah dan sesuai. Teks preprocessing lama tidak menjadi representasi aktif. Jejak sumber menunjuk [`data/kbji_2014.pdf`](data/kbji_2014.pdf), halaman PDF 161–327.

Audit input aktual memverifikasi **501 − 85 residual − 1 kebocoran atribut = 415 kode jabatan unik**. Residual diperiksa melalui kode `.99` atau nama `lainnya|ybdi|ytdl`, termasuk `2353.00` dan `2359.00` yang bukan `.99`; kebocoran atribut adalah `2635.09`. Kode dibaca/ditulis sebagai string; nol akhir seperti `2111.10` dan `2162.00` dipertahankan. Nama sama pada kode berbeda tetap unit berbeda. Klasifikasi hasil ekstraksi tidak diklaim sebagai salinan resmi KBJI tanpa pemeriksaan sumber tambahan.

| Kode | Subgolongan Pokok | Jabatan audit aktual |
| --- | --- | ---: |
| 21 | Ahli Ilmu Pengetahuan dan Teknik | 138 |
| 22 | Profesional Kesehatan | 74 |
| 23 | Profesional Pendidikan | 42 |
| 24 | Profesional Bisnis dan Administrasi | 29 |
| 25 | Profesional Teknologi Informasi dan Komunikasi | 26 |
| 26 | Profesional Hukum, Sosial, dan Budaya | 106 |
| Total | | 415 |

Lexical set dibaca dari [`data/istilah_gender.csv`](data/istilah_gender.csv). Delapan pasangan tetap dan urutan kanonis adalah `laki-laki–perempuan`, `pria–wanita`, `ayah–ibu`, `suami–istri`, `putra–putri`, `pemuda–pemudi`, `cowok–cewek`, dan `jejaka–gadis`. ID angka historis dipetakan menjadi ID term-based pada hasil aktif; input tidak diubah. Metadata menyebutnya atribut operasional yang belum tervalidasi linguistik final.

## Metode aktif

Satu model [`models/cc.id.300.bin`](models/cc.id.300.bin), 300 dimensi, pretrained fastText Bahasa Indonesia dari Common Crawl dan Wikipedia, digunakan tanpa pelatihan ulang. Model menggunakan CBOW dengan bobot posisi; [sumber resmi fastText](https://fasttext.cc/docs/en/crawl-vectors.html). Eksekusi memverifikasi ukuran 7.236.808.873 byte, vocabulary 2.000.000 istilah, dimensi 300, serta SHA-256 streaming pada [manifest run](outputs/analisis_utama/2026-10-14_23-08-25/audit/manifest_run.json). File model tetap disimpan lokal.

Nama asli disimpan terpisah. Normalisasi aktif: Unicode NFKC, lowercase, hapus petik lurus/keriting tunggal/ganda, karakter tidak tercetak `Cc`/`Cf` menjadi spasi, `(`, `)`, `/`, `,`, `.` menjadi spasi, lalu rapikan whitespace dan tokenisasi spasi. Semua kata dipertahankan, termasuk `umum` dalam kurung; tanda hubung dan punctuation lain dipertahankan. Penggantian titik hanya pada teks, tidak pada kode. Tidak ada stemming, stopword removal, terjemahan, atau penggantian sinonim. Audit aturan ini menghasilkan 42 nama satu token dan 373 multikata, sesuai acuan. Perlakuan kurung/slash dan daftar punctuation merupakan keputusan teknis implementasi.

Vektor token mentah dikonversi `float64`, dirata-ratakan dengan koefisien sama, kemudian dinormalisasi L2. Token tidak dinormalisasi sebelum rata-rata. Atribut dinormalisasi sebelum cosine. Token OOV tetap digunakan karena biner fastText mendukung subkata, dan statusnya diaudit. Rata-rata tidak menangkap urutan/konteks; koefisien sama tidak menjamin pengaruh geometris yang sama.

SC-WEAT adalah selisih mean cosine delapan atribut laki-laki dan perempuan, dibagi SD seluruh **16 cosine satu jabatan**, `ddof=0`. Permutasi eksak dua sisi memakai statistik jumlah, seluruh **12.870** pembagian berlabel, termasuk pengamatan/komplemen, nilai sama ekstrem, tanpa Monte Carlo atau koreksi tambah satu. Toleransi numerik `atol=rtol=1e-12` dicatat; arah menggunakan `zero_atol=0.0` sehingga nol hanya untuk skor tepat nol yang belum dibulatkan. BH-FDR diterapkan sekali pada 415 p-value dengan `q=0.05`.

Ringkasan enam kelompok memakai SD antarjabatan `ddof=0` dan keputusan BH global. LOPO mengeluarkan pasangan serentak, menghitung ulang mean tujuh/tujuh dan penyebut 14 cosine `ddof=0`, lalu menyandingkan skor, arah, dan ranking bertanda dengan average ties berdasarkan kode. Spearman dilaporkan deskriptif. Tidak ada kategori stabil/tidak stabil yang dibuat sendiri. Perbandingan model, `wiki.id.bin`, IndoBERT/LPBS/w-SEAT, dan inferensi antarsubgolongan tidak menjadi pipeline aktif.

Definisi dan urutan perhitungan lengkap: [`docs/metodologi.md`](docs/metodologi.md).

## Environment dan menjalankan dari kernel bersih

Environment aktual menggunakan **Python 3.13.15, 64 bit, Windows**. Paket aktif dipin pada [`requirements.txt`](requirements.txt): NumPy 2.5.3, pandas 3.0.5, SciPy 1.18.1, statsmodels 0.15.0, Matplotlib 3.11.1, seaborn 0.13.2, fasttext-community 0.11.7, nbformat 5.11.1, nbclient 0.11.0, ipykernel 7.3.0, Jupyter 1.1.1, nbconvert 7.17.1, dan Pillow 12.3.0 untuk inspeksi gambar. Notebook merekam versi Python penuh dan paket analisis pada manifest; sel notebook tidak memasang paket tersembunyi.

Jalankan PowerShell dari root repository. Gunakan Python 3.13, buat environment, lalu pasang paket sebelum membuka notebook:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m ipykernel install --user --name skripsi-scweat --display-name "Python 3.13 (SC-WEAT)"
.\.venv\Scripts\python.exe -m jupyter notebook notebooks/analisis_scweat_kbji.ipynb
```

Pilih kernel environment tersebut, periksa sel konfigurasi path input/lexical/model, lalu **Restart Kernel and Run All**. Root dideteksi melalui `pathlib` sehingga notebook dapat dijalankan dari root atau `notebooks/`, tanpa path absolut komputer pengembang. Letakkan biner asli di `models/cc.id.300.bin` bila belum tersedia; notebook tidak menggantinya dengan `.vec` atau model lain. `RANDOM_SEED=42` digunakan untuk pengacakan validasi dan jitter visual.

Run yang dilaporkan menggunakan environment yang sudah tersedia dan versi paketnya diperiksa. Instalasi ulang pada venv baru belum dijalankan; perintah di atas merupakan panduan setup untuk reproduksi.

Eksekusi noninteraktif dari root dengan environment yang sama:

```powershell
.\.venv\Scripts\python.exe -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 --ExecutePreprocessor.kernel_name=skripsi-scweat notebooks/analisis_scweat_kbji.ipynb
```

Setiap eksekusi membentuk `run_id` baru berdasarkan waktu Asia/Jakarta dan folder output baru. Jangan menjalankan beberapa proses notebook yang menulis berkas yang sama bersamaan. Keberhasilan implementasi berbeda dari keberhasilan eksekusi: hasil dianggap selesai setelah seluruh sel, ekspor, manifest, dan QC akhir tersedia.

## Tabel, gambar, dan jejak hasil

| Keluaran | Fungsi |
| --- | --- |
| `tables/data_analisis.csv` | 415 kode, nama asli/analisis, token, kelompok, sumber |
| `tables/lexicon_gender.csv` | Delapan pasangan dengan ID kanonis dan metadata |
| `tables/cosine_atribut.csv` | Cosine seluruh 415 target terhadap 16 atribut |
| `tables/hasil_scweat.csv` | Komponen SC-WEAT, statistik T, p-value, BH, arah, ranking |
| `tables/ringkasan_subgolongan.csv` | Enam ringkasan lengkap RM 2 |
| `tables/lopo_detail.csv` | 3.320 kombinasi kode × pasangan, skor/delta/arah/ranking |
| `tables/lopo_ringkasan_jabatan.csv` | 415 ringkasan sensitivitas individual |
| `tables/lopo_ringkasan_pasangan.csv` | Delapan ringkasan pasangan dan Spearman deskriptif |
| `audit/audit_preparasi.csv`, `audit/audit_vektor.csv` | Seleksi/perubahan teks dan validitas/OOV vektor |
| `audit/validasi.csv`, `audit/manifest_run.json` | Bukti pemeriksaan dan parameter/provenance/durasi/status run |
| `figures/` | Grafik audit, RM 1, RM 2, RM 3; PNG 300 dpi dan SVG serta heatmap lengkap |

Gambar audit menjelaskan seleksi, jumlah anggota, token, dan OOV. RM 1 menyandingkan skor/cosine/signifikansi; RM 2 membandingkan distribusi enam kelompok; RM 3 menunjukkan perubahan skor, arah, peringkat, dan jabatan paling sensitif. Heatmap delta menggunakan skala divergen terhadap nol; warna menunjukkan arah perubahan. Grafik notebook hanya menampilkan subset heatmap terbesar dengan kriteria yang dijelaskan, dan keluaran lengkap disediakan terpisah.

Run aktual: [`outputs/analisis_utama/2026-10-14_23-08-25/`](outputs/analisis_utama/2026-10-14_23-08-25/). [`LATEST_RUN.txt`](outputs/analisis_utama/LATEST_RUN.txt) menunjuk run lengkap terbaru. [Schema keluaran](outputs/analisis_utama/2026-10-14_23-08-25/audit/schema_output.csv) mencatat kolom/tipenya. Kamus kolom dan relasi: [`docs/kamus_data.md`](docs/kamus_data.md). Catatan kerja dan status run: [`docs/catatan_pengerjaan.md`](docs/catatan_pengerjaan.md).

## RM 1 — arah, skor, dan signifikansi individual

Dari 415 jabatan terdapat **105 positif, 310 negatif, dan 0 nol**. Mean effect size **−0,170002**, median **−0,167623**, rentang **−0,935940 sampai 0,905523**. Nilai paling negatif adalah `2320.02 Instruktur Tata Rias`; paling positif `2652.02 Komposer`. Ini merupakan asosiasi relatif dalam embedding, bukan pernyataan tentang orang yang menjalankan pekerjaan tersebut.

**Tidak ada hasil signifikan**, baik p-value mentah maupun setelah BH global. P-value mentah minimum 0,060140; seluruh adjusted p-value 0,999223. Hasil ini tidak membuktikan tidak ada asosiasi: arah/skor tetap dilaporkan, sementara bukti pengujian terhadap pengacakan label atribut tidak mencapai taraf yang ditetapkan.

[Hasil penuh RM 1](outputs/analisis_utama/2026-10-14_23-08-25/tables/hasil_scweat.csv), [histogram](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm1_histogram_effect_size.png), [jabatan ekstrem](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm1_jabatan_ekstrem.png), dan [kurva BH](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm1_pvalue_bh.png).

## RM 2 — distribusi deskriptif enam Subgolongan Pokok

| Kode | n | Mean | Median | SD `ddof=0` | Negatif | Signifikan BH global |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 21 | 138 | −0,1010 | −0,0767 | 0,2350 | 68,84% | 0 |
| 22 | 74 | −0,2433 | −0,2193 | 0,2550 | 82,43% | 0 |
| 23 | 42 | −0,3419 | −0,3211 | 0,2063 | 95,24% | 0 |
| 24 | 29 | −0,1569 | −0,1203 | 0,2342 | 75,86% | 0 |
| 25 | 26 | −0,2688 | −0,1913 | 0,2512 | 88,46% | 0 |
| 26 | 106 | −0,1199 | −0,1660 | 0,3127 | 65,09% | 0 |

Semua mean dan median negatif. Kelompok 21 mempunyai mean tertinggi (−0,100979), kelompok 23 terendah (−0,341921) dan proporsi negatif terbesar. Kelompok 26 mempunyai penyebaran terbesar (SD 0,312722), rentang −0,902061 sampai 0,905523, sehingga mean kelompok tidak mewakili semua jabatan. Semua kelompok memiliki nol skor nol dan nol penolakan BH global. Proporsi memakai denominator kelompok; perbandingan ini deskriptif tanpa klaim beda signifikan antarkelompok.

[Tabel lengkap RM 2](outputs/analisis_utama/2026-10-14_23-08-25/tables/ringkasan_subgolongan.csv) memuat minimum/maksimum dan seluruh proporsi. [Boxplot dengan semua observasi](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm2_boxplot_observasi.png), [mean–median](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm2_mean_median.png), [proporsi arah](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm2_proporsi_arah.png), dan [ECDF](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm2_ecdf.png).

## RM 3 — sensitivitas pemilihan pasangan atribut

**178 jabatan mempertahankan arah pada semua delapan kondisi**, sedangkan **237 mengalami sedikitnya satu pembalikan positif–negatif**. Rata-rata perubahan absolut terbesar terjadi ketika `pemuda–pemudi` dikeluarkan (**0,267038**, 77 pembalikan); `jejaka–gadis` menghasilkan mean perubahan absolut **0,233948** dan pembalikan terbanyak (**155**). Pasangan tidak dihapus otomatis.

Perubahan absolut maksimum **0,777140**, pada `2654.03 Sutradara Film` ketika `jejaka–gadis` dikeluarkan. Spearman baseline–LOPO berkisar **0,926615–0,988565**. Korelasi peringkat tinggi hadir bersama pembalikan arah individual, sehingga perubahan skor, arah, dan ranking dibaca bersama. Tidak ada uji signifikansi tambahan atau kategori stabil/tidak stabil.

[Detail 3.320 kombinasi](outputs/analisis_utama/2026-10-14_23-08-25/tables/lopo_detail.csv), [ringkasan 415 jabatan](outputs/analisis_utama/2026-10-14_23-08-25/tables/lopo_ringkasan_jabatan.csv), [ringkasan delapan pasangan](outputs/analisis_utama/2026-10-14_23-08-25/tables/lopo_ringkasan_pasangan.csv), [scatter delapan kondisi](outputs/analisis_utama/2026-10-14_23-08-25/figures/rm3_scatter_delapan_kondisi.png), dan [heatmap lengkap HTML](outputs/analisis_utama/2026-10-14_23-08-25/figures/lopo_heatmap_lengkap.html). Heatmap seluruh 415 jabatan juga tersedia sebagai **17 halaman PNG/SVG** di folder gambar; HTML hanya pelengkap.

## Diagnostik penjelas hasil D1–D5

Tambahan yang diotorisasi 18 Oktober 2026 menjelaskan hasil RM 1–RM 3 dari CSV run utama lengkap. Model, preprocessing, 415 target, delapan pasangan, SC-WEAT `ddof=0`, permutasi, BH global, dan LOPO tetap. Diagnostik tidak memuat ulang model dan tidak menambah p-value LOPO, uji beda kelompok, cutoff netralitas, atau kategori stabil/tidak stabil.

| Bagian | Pertanyaan penjelas | Hubungan dengan RM |
| --- | --- | --- |
| D1 — selisih cosine per pasangan | Pasangan mana menyumbang selisih cosine positif/negatif pada pembilang? | RM 1 dan RM 3 |
| D2 — besar skor awal dan pembalikan | Bagaimana pembalikan berhubungan secara deskriptif dengan besar skor baseline? | RM 3 |
| D3 — pembilang dan penyebut | Bagaimana mean cosine, `D`, SD, dan `D / SD` berubah setelah pengeluaran pasangan? | RM 1 dan RM 3 |
| D4 — enam subgolongan | Apakah pergeseran/pembalikan muncul lintas kelompok atau terkonsentrasi? | RM 2 dan RM 3 |
| D5 — distribusi permutasi | Seberapa ekstrem pengamatan dibandingkan 12.870 pembagian label baseline? | RM 1 |

Metode lengkap dan batas setiap ukuran tersedia pada [bagian diagnostik metodologi](docs/metodologi.md). Selisih cosine pasangan merupakan komponen pembilang, bukan effect size pasangan tunggal. Selama SD positif, pembalikan tanda skor terjadi karena tanda pembilang berubah; SD hanya memengaruhi besarnya. Pembalikan tidak otomatis membuat hasil tidak valid atau pasangan salah.

Untuk menjalankan **hanya diagnostik**, buka notebook aktif, restart kernel tanpa menghapus output tersimpan, lalu jalankan berurutan seluruh cell mulai judul **Diagnostik penjelas hasil** sampai QC diagnostik terakhir. Cell bagian ini diberi tag `diagnostics`; bootstrap mengimpor paket dan membaca CSV secara eksplisit dari `outputs/analisis_utama/LATEST_RUN.txt`, lalu memeriksa manifest sumber `complete`. Cara ini tidak membutuhkan variabel bagian utama atau model yang sudah dimuat.

Keluaran diagnostik menggunakan `outputs/diagnostik/<diagnostic_run_id>/{tables,figures,audit}` dan mencatat `parent_run_id` serta lingkup eksekusi. Run utama dan pointer `LATEST_RUN.txt` utama dipertahankan. Jika memilih Run All seluruh notebook, bagian utama membentuk run baru terlebih dahulu dan diagnostik memakai run lengkap tersebut. Lingkup eksekusi harus dibaca pada manifest; menjalankan diagnostik saja tidak berarti pengukuran utama dijalankan ulang.

### Run diagnostik dan keluaran aktual

Run **`2026-10-18_02-57-12`** berlangsung **18 Oktober 2026 pukul 02:57:12–02:57:24 WIB**, durasi manifest **11,72 detik**, berstatus `complete`. Notebook kini mempunyai 68 cell: 45 cell utama yang dipertahankan dan 23 tambahan bertag `diagnostics`, termasuk 13 cell kode yang benar-benar dieksekusi pada tambahan ini. [Manifest diagnostik](outputs/diagnostik/2026-10-18_02-57-12/audit/manifest_diagnostik.json) menyatakan `execution_scope=diagnostics_only_from_saved_csv` dan `model_loaded=false`.

| Bagian | Tabel penuh | Visual penjelas |
| --- | --- | --- |
| D1 | [3.320 selisih pasangan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_selisih_pasangan_detail.csv), [delapan ringkasan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_selisih_pasangan_ringkasan.csv) | [Boxplot selisih](outputs/diagnostik/2026-10-18_02-57-12/figures/d1_boxplot_selisih_pasangan.png), [mean–median](outputs/diagnostik/2026-10-18_02-57-12/figures/d1_mean_median_selisih.png) |
| D2 | [415 jabatan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_skor_awal_pembalikan.csv), [dua kelompok kejadian](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_ringkasan_kejadian_pembalikan.csv), [distribusi 0–8 pembalikan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_distribusi_jumlah_pembalikan.csv) | [Skor awal–jumlah pembalikan](outputs/diagnostik/2026-10-18_02-57-12/figures/d2_skor_awal_jumlah_pembalikan.png), [skor awal–perubahan maksimum](outputs/diagnostik/2026-10-18_02-57-12/figures/d2_skor_awal_perubahan_maksimum.png) |
| D3 | [415 komponen baseline](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_komponen_baseline.csv), [3.320 komponen LOPO](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_komponen_scweat_lopo.csv), [18 baris studi kasus](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_studi_kasus_komponen.csv) | [Perubahan D–SD](outputs/diagnostik/2026-10-18_02-57-12/figures/d3_perubahan_pembilang_penyebut.png), [dua ilustrasi komponen](outputs/diagnostik/2026-10-18_02-57-12/figures/d3_studi_kasus_komponen.png) |
| D4 | [48 kelompok–pasangan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_sensitivitas_subgolongan.csv) | [Heatmap mean delta](outputs/diagnostik/2026-10-18_02-57-12/figures/d4_heatmap_mean_delta.png), [heatmap proporsi pembalikan](outputs/diagnostik/2026-10-18_02-57-12/figures/d4_heatmap_proporsi_pembalikan.png) |
| D5 | [Tiga contoh terpilih](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_permutasi_contoh.csv), [38.610 statistik permutasi](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_permutasi_distribusi.csv) | [Tiga distribusi permutasi](outputs/diagnostik/2026-10-18_02-57-12/figures/d5_distribusi_permutasi.png) |

**D1 — dua komponen berlawanan.** Mean selisih cosine terbesar ke arah positif adalah `pemuda–pemudi` **+0,080524**; paling negatif `jejaka–gadis` **−0,090329**. Pada **404 jabatan**, kedua pasangan tersebut sekaligus memberikan selisih positif dan negatif. Keduanya dapat saling mengimbangi dalam rata-rata pembilang. [Subset lengkap pasangan berlawanan](outputs/diagnostik/2026-10-18_02-57-12/tables/diag_pasangan_berlawanan_contoh.csv) tersedia, meskipun notebook menampilkan sepuluh contoh menurut jumlah absolut selisih. Ini tidak membuktikan frekuensi kata, usia, register, atau kekerabatan sebagai penyebab.

**D2 — pembalikan dan besar perubahan berbeda.** Ada **237 jabatan unik pernah membalik**, **178 tidak pernah**, dan **416 kejadian jabatan–pasangan**. Median besar skor awal masing-masing **0,111856** dan **0,383373**. Mean perubahan absolut maksimum masing-masing **0,308123** dan **0,293133**. Pada data ini kelompok yang pernah membalik mempunyai skor awal lebih kecil, sementara besar perubahan maksimum tetap perlu dibaca terpisah; rentang kedua kelompok tumpang tindih. Tidak dibuat cutoff netralitas atau klaim sebab-akibat.

**D3 — tanda berasal dari pembilang.** Seluruh **3.320 effect size LOPO cocok dalam toleransi numerik** dengan sumber. Pada `2654.03 Sutradara Film`, pengeluaran `jejaka–gadis` mengubah `D` **−0,002875 → +0,018217**, SD **0,037568 → 0,026002**, dan skor **−0,076525 → +0,700614**. Tanda berubah karena `D` melewati nol; SD tetap positif. Pada `2622.02 Juru Katalog`, pengeluaran `pemuda–pemudi` mengubah `D` **−0,017640 → −0,030621**, SD **0,034987 → 0,025306**, dan skor **−0,504195 → −1,210036**: perubahan besar tanpa pembalikan. Dua ilustrasi ini bukan sampel representatif dan tidak membagi pengaruh kausal pembilang/penyebut.

**D4 — pergeseran melintasi keenam kelompok.** Pengeluaran `pemuda–pemudi` memberi mean delta negatif pada **6/6 kelompok**, sedangkan pengeluaran `jejaka–gadis` memberi mean delta positif pada **6/6 kelompok**. Besarnya bervariasi: mean delta tanpa `pemuda–pemudi` berkisar **−0,347034 sampai −0,206149**. Proporsi pembalikan terbesar adalah **66/138 = 47,83%** pada kelompok **21** ketika `jejaka–gadis` dikeluarkan. Denominator adalah anggota kelompok, sehingga jumlah kejadian mentah tidak disamakan dengan proporsi. Ini perbandingan deskriptif tanpa uji beda kelompok.

**D5 — pengamatan terhadap distribusi label.** Contoh dipilih dari besar skor tertinggi, tepat median besar skor, dan terendah; ties menurut kode, tanpa pengulangan. Median besar skor seluruh jabatan **0,214813**.

| Kriteria | Kode dan jabatan | Besar skor awal | Kejadian ekstrem / 12.870 | p mentah | Adjusted p global |
| --- | --- | ---: | ---: | ---: | ---: |
| Tertinggi | 2320.02 Instruktur Tata Rias | 0,935940 | 774 | 0,060140 | 0,999223 |
| Terdekat median | 2330.02 Guru Pendidikan Menengah Atas | 0,214813 | 9.264 | 0,719814 | 0,999223 |
| Terendah | 2120.02 Ahli Matematika Terapan | 0,000538 | 12.860 | 0,999223 | 0,999223 |

Untuk contoh terbesar sekalipun, **774 dari 12.870** statistik label setidaknya sama ekstrem dengan pengamatan, sehingga p dua sisi **0,060140** belum mencapai `0.05`. Dua contoh lainnya mempunyai lebih banyak kejadian ekstrem. Adjusted p diambil dari BH global 415 pengujian yang sudah ada. D5 memvisualisasikan ulang pengujian baseline; p bukan probabilitas hipotesis nol benar, dan tidak signifikan tidak membuktikan semua asosiasi tidak ada.

[16 pemeriksaan diagnostik PASS](outputs/diagnostik/2026-10-18_02-57-12/audit/validasi_diagnostik.csv) memverifikasi identitas, sembilan checksum tabel, rekonstruksi baseline/selisih/LOPO, kejadian pembalikan, 48 denominator kelompok–pasangan, tiga permutasi termasuk referensi jumlah langsung, join setelah pengacakan, round-trip kode, serta keutuhan **11 input** dan pointer utama. [Schema](outputs/diagnostik/2026-10-18_02-57-12/audit/schema_diagnostik.csv), [caption](outputs/diagnostik/2026-10-18_02-57-12/audit/caption_diagnostik.csv), [interpretasi](outputs/diagnostik/2026-10-18_02-57-12/audit/interpretasi_diagnostik.json), dan [snapshot source notebook](outputs/diagnostik/2026-10-18_02-57-12/audit/notebook_source.json) menyimpan jejak lengkap. [`LATEST_DIAGNOSTIC_RUN.txt`](outputs/diagnostik/LATEST_DIAGNOSTIC_RUN.txt) menunjuk run diagnostik lengkap terbaru; pointer utama tetap `2026-10-14_23-08-25`.

Seluruh sepuluh visual diagnostik diperiksa keterbacaannya, termasuk label grafik yang padat; semua PNG mempunyai metadata 300 dpi. Seluruh 40 checksum output diagnostik cocok dengan manifest. Hasil utama dan 100 checksum output parent tetap utuh.

## Validasi analisis utama dan batas yang tersisa

[Seluruh 16 pemeriksaan PASS](outputs/analisis_utama/2026-10-14_23-08-25/audit/validasi.csv). Pemeriksaan mencakup cakupan/seleksi/nama sumber; dimensi/norma/finite vektor; 36 perbandingan cosine dengan rumus umum; 12.870 partisi unik dan simetri komplemen; skor/p-value referensi skalar pada sembilan jabatan; BH independen; pertukaran label; hitung ulang cosine/skor/p-value/FDR/LOPO setelah acak baris dan atribut lalu merge berdasarkan kode; SD/ringkasan/LOPO; serta round-trip 21 tabel ekspor dengan kode berekor nol tetap utuh. [Referensi numerik](outputs/analisis_utama/2026-10-14_23-08-25/audit/validasi_referensi.csv) dan [invariansi](outputs/analisis_utama/2026-10-14_23-08-25/audit/validasi_invariansi.csv) menyimpan buktinya.

Vektor seluruh target/atribut valid. Lima jabatan memiliki token OOV (`endodontist`, `paedodontist`, `prosthodontist`, `subtitler`, `unani`), 410 tidak; token OOV tetap direpresentasikan melalui subkata. [Audit vektor](outputs/analisis_utama/2026-10-14_23-08-25/audit/audit_vektor.csv) dan [ringkasan tiga populasi OOV](outputs/analisis_utama/2026-10-14_23-08-25/audit/audit_oov_ringkasan.csv) memperlihatkan kondisi aktual tanpa menjalankan pengujian baru.

Seluruh 37 PNG mempunyai metadata 300 dpi. Keterbacaan 20 visual utama diperiksa melalui contact sheet dan gambar individual yang padat; halaman pertama dan terakhir heatmap lengkap diperiksa langsung. Ukuran dan SHA-256 seluruh 100 berkas dalam daftar output cocok dengan manifest; manifest sendiri tidak masuk daftar checksum tersebut.

Tidak ada kendala eksekusi yang tersisa pada run ini. Proposal revisi asli belum tersedia, dan ekstraksi klasifikasi tidak dianggap salinan resmi KBJI tanpa audit sumber tambahan. Model tunggal, atribut operasional, agregasi tanpa urutan/konteks, serta batas asosiasi intrinsik tetap berlaku. Semua gambar memakai data baru; [caption aktual](outputs/analisis_utama/2026-10-14_23-08-25/audit/caption_figures.csv) dan [jawaban RM](outputs/analisis_utama/2026-10-14_23-08-25/audit/jawaban_rm.json) tersedia pada run.
