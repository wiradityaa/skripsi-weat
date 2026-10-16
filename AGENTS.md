# Panduan pembangunan aktif `skripsi-weat`

## Acuan dan arah penelitian

Mulai 14 Oktober 2026, spesifikasi aktif adalah prompt `PROMPT_CODEX_SKRIPSI_SCWEAT_START_FRESH.md` yang diotorisasi pengguna, diringkas pada `docs/acuan_proposal.md`. Proposal revisi asli belum tersedia di repository; jangan mengklaim dokumen acuan sebagai salinan proposal asli. Pedoman lama berada di `archive/2026-10-14_eksperimen_lama/reports/AGENTS_legacy.md` dan tidak menjadi instruksi aktif.

Judul: **Analisis Asosiasi Gender dan Sensitivitas Skor SC-WEAT terhadap Pemilihan Pasangan Istilah Gender pada Klasifikasi Baku Jabatan Indonesia (KBJI) 2014 Menggunakan fastText Bahasa Indonesia.**

Titik masuk tunggal adalah `notebooks/analisis_scweat_kbji.ipynb`. Semua kode pengukuran aktif berada dalam notebook linear, dengan Markdown berbahasa Indonesia, fungsi kecil seperlunya, parameter eksplisit, dan keluaran aktual. Jangan mengganti deliverable menjadi package, aplikasi, class, pipeline object, atau kumpulan notebook eksperimen.

Tiga analisis utama:

1. RM 1: arah, effect size, dan signifikansi per 415 jabatan Profesional KBJI 2014.
2. RM 2: perbandingan deskriptif enam Subgolongan Pokok, bagian utama tersendiri.
3. RM 3: sensitivitas terhadap pengeluaran setiap pasangan atribut melalui leave-one-pair-out, deskriptif tanpa inferensi tambahan.

Model tunggal adalah pretrained `models/cc.id.300.bin`, 300 dimensi, tanpa pelatihan ulang. Perbandingan model, `wiki.id.bin`, IndoBERT/LPBS/w-SEAT, dan uji beda inferensial antarsubgolongan berada di luar pipeline aktif. Jangan memilih metode/model/hasil agar temuan lebih menarik.

## Perlindungan repository dan hasil historis

- Pastikan working directory berada di root repository sebelum perubahan.
- Periksa status Git dan isi folder sebelum penghapusan atau pemindahan besar. Lindungi perubahan lokal pengguna.
- Periksa target absolut secara eksplisit; jangan menghapus parent directory atau `.git`.
- Notebook/output/laporan eksperimen lama berada di `archive/2026-10-14_eksperimen_lama/` dan dipertahankan sebagai sejarah. Jangan menghapus, menimpa, atau memakai hasilnya sebagai hasil skripsi aktif.
- Input aktif memakai nama deskriptif tanpa awalan eksperimen sesuai instruksi pengguna. Data analisis berada di `data/`, pemeriksaan silang di `data/validasi/`, dan data yang tidak dipakai diarsipkan sebagai `data_tidak_relevan`. Renaming tidak boleh mengubah isi data; catat pemetaan path dan hash. Nama model resmi dipertahankan.
- Pipeline aktif tidak mengimpor kode arsip atau membaca embedding/cosine/skor/p-value/ringkasan lama. Kode lama boleh dibaca untuk memahami data; pengukuran aktif ditulis dan dijalankan baru.
- Arsipkan berkas tracked melalui `git mv` bila memungkinkan dan catat manifest lokasi/jenis/alasan/SHA-256. Jangan membuat commit tanpa instruksi pengguna.

## Data, identitas, dan lexical set

Sumber utama adalah `data/kbji_profesional.csv`; `occupation_name` menjadi `occupation_name_raw`. Periksa silang nama/kode/seleksi mentah dengan `data/validasi/kbji_profesional_sumber.csv`; jangan memakai teks preprocessing lama sebagai target. Sumber PDF adalah `data/kbji_2014.pdf` dengan jejak halaman.

Baca semua kode sebagai string sejak awal sampai ekspor. Bentuk `2111.10` dan `2162.00` harus dipertahankan; konversi melalui float atau perbaikan identitas berdasarkan tebakan dilarang. Kunci jabatan adalah `occupation_code`; nama sama berkode berbeda tidak digabungkan. Parse boolean eksplisit, termasuk string `True`/`False`.

Checkpoint seleksi adalah 501 entri awal, 85 residual, satu kebocoran atribut, 415 jabatan final. Distribusi Subgolongan Pokok 21–26 adalah 138, 74, 42, 29, 26, dan 106. Acuan 42 satu token dan 373 multikata dibandingkan dengan hasil preprocessing aktual; jangan memaksa data agar cocok.

Residual dihitung ulang melalui akhiran kode `.99` atau nama `lainnya|ybdi|ytdl`; residual non-`.99` `2353.00` dan `2359.00` harus terdeteksi. Kebocoran atribut menggunakan kecocokan token utuh, bukan substring; `2635.09` dikecualikan pada input saat ini. Selidiki perbedaan identitas/jumlah sebelum pengukuran utama. Simpan keputusan, alasan, nama asli/analisis, token, hierarki, dan halaman sumber.

Lexical set dibaca dari `data/istilah_gender.csv` tanpa menimpa input. Validasi delapan pasangan dan 16 istilah unik, lalu gunakan urutan/ID kanonis:

| pair_id | male_term | female_term |
| --- | --- | --- |
| laki_laki_perempuan | laki-laki | perempuan |
| pria_wanita | pria | wanita |
| ayah_ibu | ayah | ibu |
| suami_istri | suami | istri |
| putra_putri | putra | putri |
| pemuda_pemudi | pemuda | pemudi |
| cowok_cewek | cowok | cewek |
| jejaka_gadis | jejaka | gadis |

Metadata input kategori/register/validation_status/include_baseline/notes dipertahankan. Pasangan merupakan atribut operasional sesuai acuan; jangan menyebutnya tervalidasi linguistik final.

## Preprocessing, embedding, dan numerik

Konfigurasi terpusat memuat path relatif, kebijakan teks, `RANDOM_SEED = 42`, `ddof=0`, FDR `0.05`, `atol=1e-12`, `rtol=1e-12`, `zero_atol=0.0` untuk arah nol literal, urutan kelompok/pasangan, palette, dan parameter visual. Gunakan `pathlib` dengan root yang bekerja dari root maupun `notebooks/`; hindari path absolut pengembang.

Normalisasi: Unicode NFKC, lowercase, hapus tanda petik lurus/keriting tunggal/ganda, ubah karakter tidak tercetak Unicode `Cc`/`Cf` menjadi spasi, ubah delimiter `(`, `)`, `/`, `,`, `.` menjadi spasi, rapatkan whitespace, lalu split spasi. Kata di dalam kurung termasuk `umum` dipertahankan. Hyphen di dalam kata dan punctuation lain dipertahankan. Kebijakan punctuation berlaku pada teks, bukan kode. Tidak ada stemming, stopword removal, lemmatization, terjemahan, sinonim otomatis, atau pembobotan kata inti. Nyatakan kebijakan tambahan ini sebagai keputusan teknis, bukan klaim isi proposal.

Gunakan API fastText `.bin` yang mendukung `get_word_vector()` dan subword OOV. Konversi vektor native ke `float64` sebelum agregasi. Target multikata memakai rata-rata aritmetika vektor token mentah, koefisien sama, kemudian L2; jangan normalisasi token sebelum rata-rata. Atribut dinormalisasi L2 sebelum cosine. Cosine berupa dot product vektor satuan dan diverifikasi dengan bentuk umum. Periksa dimensi 300, finite, norma bukan nol/satu, dan vocabulary/OOV. OOV tidak dibuang otomatis. Vektor tidak valid atau penyebut nol tidak diganti nol atau dihilangkan diam-diam.

## Metode tetap dan interpretasi

SC-WEAT per jabatan: mean delapan cosine laki-laki dikurangi mean delapan cosine perempuan, dibagi SD **16 cosine jabatan yang sama**, `ddof=0`. Statistik permutasi adalah jumlah laki-laki dikurangi perempuan, setara delapan kali selisih mean.

Permutasi eksak dua sisi memakai semua `C(16,8)=12870` pembagian berlabel, pengamatan/komplemen termasuk, dan menghitung nilai sama ekstrem dalam toleransi. Permutasi menukar label seluruh atribut, tidak terbatas dalam pasangan. Tidak ada Monte Carlo atau koreksi tambah satu. BH diterapkan satu kali global pada 415 p-value, `q=0.05`; jangan BH per kelompok.

Gunakan hanya `positif`, `negatif`, dan `nol` berdasarkan skor belum dibulatkan dengan kebijakan nol yang dinyatakan. Positif relatif ke laki-laki, negatif relatif ke perempuan. Tidak signifikan bukan bukti tidak ada asosiasi; nol bukan netralitas sosial. Hasil bukan bukti langsung diskriminasi, komposisi tenaga kerja, atau kelayakan gender pekerjaan. Klasifikasi ekstraksi tidak disebut salinan resmi KBJI tanpa audit sumber tambahan.

RM 2: enam kelompok 21–26, jumlah, mean, median, SD **antarjabatan** `ddof=0`, min/max, jumlah/proporsi arah, dan jumlah/proporsi keputusan BH global. Tidak ada uji beda inferensial antarkelompok.

RM 3: keluarkan dua istilah satu pasangan, hitung ulang mean tujuh/tujuh dan SD 14 cosine `ddof=0`, tanpa p-value/FDR/signifikansi tambahan. Pasangkan baseline berdasarkan kode. Simpan 3.320 detail, delta LOPO−baseline, abs_delta, arah, pembalikan positif-negatif, transisi nol terpisah, dan rank bertanda menurun dengan average ties. Ringkasan 415 jabatan dan delapan pasangan mencakup perubahan skor/arah/rank, Spearman deskriptif, dan seluruh pasangan terikat maksimum. Jangan membuat ambang stabil/tidak stabil.

## Output, validasi, dan dokumentasi

Satu run waktu Asia/Jakarta memiliki folder baru `outputs/analisis_utama/<run_id>/{tables,figures,audit}`. CSV UTF-8 tanpa indeks dan tanpa pembulatan analisis; tabel ditampilkan ringkas dalam notebook. Simpan gambar utama PNG minimal 300 dpi serta SVG bila sesuai; visualisasi juga tampil di notebook. Heatmap delta divergen simetris terhadap nol; subset notebook punya kriteria transparan dan keluaran lengkap tersedia. Gunakan mapping warna positif `#D55E00`, negatif `#7B61A8`, nol `#7A7A7A`, label Indonesia, seed jitter 42, dan ruang label memadai.

Setiap RM mempunyai tabel, visualisasi, dan interpretasi dari eksekusi baru. Sel terakhir notebook berisi assertion dan tabel QC. Validasi harus benar-benar menghitung ulang: referensi p-value terpisah, simetri komplemen, pertukaran label, pengacakan baris dan atribut beserta mapping, hasil/FDR/LOPO dicocokkan berdasarkan kode, group consistency, dan round-trip kode string. Jangan mencatat PASS tanpa bukti.

Notebook valid menurut `nbformat`, kernelspec Python 3, dan lulus Restart Kernel and Run All. Simpan output aktual serta durasi tahap mahal, hindari mencetak embedding penuh atau ribuan baris. Manifest mencatat SHA-256 streaming input/lexical/model, konfigurasi, versi Python/paket, Git/working tree, timestamp, output, jumlah data/permutasi, durasi, dan status. Jangan memakai cache eksperimen lama.

README, docs, AGENTS, dan notebook harus selaras dengan run terbaru. Angka hasil harus berasal dari run aktual; bedakan implementasi selesai dari eksekusi selesai. Pin environment sesuai versi aktual dan dokumentasikan instalasi sebelum notebook. Jangan menambahkan referensi ilmiah atau nomor persamaan di luar acuan; nomor yang tersedia: L2 (2.2)–(2.3), cosine (2.4), SC-WEAT (2.8), statistik (2.9), p-value (2.10), BH (2.11), Spearman (2.12).

Jika sumber/model/dependensi/komputasi menghalangi, selesaikan bagian yang bisa dikerjakan dan nyatakan kendala spesifik serta tahap yang belum selesai. Jangan membuat output palsu atau mengganti hasil baru dengan hasil lama.
