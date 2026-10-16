# Acuan implementasi proposal revisi

Dokumen ini merangkum spesifikasi proposal yang disampaikan melalui `PROMPT_CODEX_SKRIPSI_SCWEAT_START_FRESH.md` dan diotorisasi pengguna pada 14 Oktober 2026. PDF proposal revisi yang disetujui dosen belum tersedia di repository. Dokumen ini merupakan acuan implementasi dari prompt tersebut, bukan salinan proposal asli atau verifikasi terhadap naskah proposal.

## Judul dan batas penelitian

**Analisis Asosiasi Gender dan Sensitivitas Skor SC-WEAT terhadap Pemilihan Pasangan Istilah Gender pada Klasifikasi Baku Jabatan Indonesia (KBJI) 2014 Menggunakan fastText Bahasa Indonesia.**

Unit pengukuran dan pengujian adalah 415 nama jabatan berkode unik dalam Golongan Pokok 2 Profesional KBJI 2014. Enam Subgolongan Pokok menjadi unit ringkasan deskriptif. Identitas unit adalah `occupation_code`; nama yang sama dengan kode berbeda tetap menjadi unit berbeda.

Hasil mengukur asosiasi intrinsik dalam ruang embedding fastText dan sensitivitas terhadap pilihan atribut. Hasil tidak merupakan bukti langsung diskriminasi, komposisi gender tenaga kerja, atau kelayakan gender suatu pekerjaan. Klasifikasi hasil ekstraksi tidak diklaim sebagai salinan resmi KBJI tanpa pemeriksaan sumber tambahan.

## Rumusan masalah dan pemetaan deliverable

| RM | Pertanyaan | Bagian notebook utama | Tabel dan visual utama |
| --- | --- | --- | --- |
| RM 1 | Bagaimana arah, besar effect size, dan signifikansi asosiasi gender pada 415 jabatan? | Pengukuran SC-WEAT, permutasi eksak, BH-FDR, dan interpretasi RM 1 | `hasil_scweat.csv`, `cosine_atribut.csv`; histogram, jabatan ekstrem, scatter cosine, kurva ambang BH, dan scatter effect size–adjusted p-value |
| RM 2 | Bagaimana perbandingan deskriptif distribusi effect size pada enam Subgolongan Pokok? | Bagian RM 2 tersendiri setelah hasil individual | `ringkasan_subgolongan.csv`; boxplot dengan observasi, mean–median, proporsi arah, ECDF, dan jumlah/proporsi penolakan BH global |
| RM 3 | Bagaimana sensitivitas skor, arah, dan peringkat ketika setiap pasangan gender dikeluarkan bergantian? | Leave-one-pair-out dan interpretasi RM 3 | `lopo_detail.csv`, `lopo_ringkasan_jabatan.csv`, `lopo_ringkasan_pasangan.csv`; scatter delapan kondisi, distribusi delta, heatmap, pembalikan arah, Spearman, dan perubahan maksimum |

Titik masuk tunggal adalah [`notebooks/analisis_scweat_kbji.ipynb`](../notebooks/analisis_scweat_kbji.ipynb). Seluruh tabel, gambar, dan audit baru disimpan di `outputs/analisis_utama/<run_id>/`. Eksperimen historis berada di [`archive/2026-10-14_eksperimen_lama/`](../archive/2026-10-14_eksperimen_lama/README.md).

## Cakupan yang harus diaudit terhadap input aktual

Alur seleksi yang ditetapkan dalam acuan adalah **501 entri awal − 85 kategori residual − 1 entri kebocoran atribut = 415 jabatan**. Angka ini merupakan checkpoint audit; implementasi harus menemukan keputusan seleksi pada input aktual dan menghentikan pengukuran bila identitas atau distribusi final tidak sesuai. Data tidak boleh diubah sekadar agar cocok dengan angka acuan.

| Kode | Subgolongan Pokok | Jumlah menurut acuan |
| --- | --- | ---: |
| 21 | Ahli Ilmu Pengetahuan dan Teknik | 138 |
| 22 | Profesional Kesehatan | 74 |
| 23 | Profesional Pendidikan | 42 |
| 24 | Profesional Bisnis dan Administrasi | 29 |
| 25 | Profesional Teknologi Informasi dan Komunikasi | 26 |
| 26 | Profesional Hukum, Sosial, dan Budaya | 106 |
| Total | | 415 |

Acuan juga mencatat 42 nama satu token dan 373 nama multikata. Komposisi aktual dihitung ulang setelah preprocessing aktif; setiap selisih harus dilaporkan beserta penyebabnya. Jika input hanya memuat 415 baris final, audit harus menyatakan keterbatasan penelusuran seleksi awal tanpa membuat baris pengecualian palsu.

## Atribut gender tetap dan urutan kanonis

| `pair_id` | Istilah laki-laki | Istilah perempuan |
| --- | --- | --- |
| `laki_laki_perempuan` | laki-laki | perempuan |
| `pria_wanita` | pria | wanita |
| `ayah_ibu` | ayah | ibu |
| `suami_istri` | suami | istri |
| `putra_putri` | putra | putri |
| `pemuda_pemudi` | pemuda | pemudi |
| `cowok_cewek` | cowok | cewek |
| `jejaka_gadis` | jejaka | gadis |

Lexical set dibaca dari [`data/istilah_gender.csv`](../data/istilah_gender.csv), diperiksa terhadap pasangan tetap di atas, kemudian diberi `pair_id` kanonis berdasarkan istilah untuk hasil aktif. Penomoran input historis dan metadata validasi dipertahankan sebagai jejak sumber; input tidak ditimpa. Pasangan adalah atribut operasional sesuai acuan, bukan lexical set yang telah tervalidasi linguistik final.

## Metode yang ditetapkan dalam acuan

- Satu model pretrained `models/cc.id.300.bin`, fastText Bahasa Indonesia 300 dimensi, tanpa pelatihan ulang. Model berbasis Common Crawl dan Wikipedia dengan CBOW berbobot posisi menurut spesifikasi model.
- Vektor target satu token diambil langsung. Target multikata menggunakan rata-rata aritmetika vektor token mentah dengan koefisien sama, kemudian normalisasi L2; vektor atribut dinormalisasi sebelum cosine.
- SC-WEAT menggunakan selisih rata-rata cosine laki-laki dan perempuan, dibagi simpangan baku seluruh 16 cosine **untuk jabatan yang sama**, dengan `ddof=0`.
- Permutasi dua sisi menguji statistik jumlah dengan seluruh `C(16,8) = 12.870` pembagian atribut berlabel. Seluruh 16 label dapat dipertukarkan; pengacakan tidak dibatasi di dalam pasangan. Pengamatan dan komplemennya termasuk enumerasi, tanpa koreksi tambah satu.
- Benjamini–Hochberg diterapkan sekali pada seluruh 415 p-value, dengan `q=0.05`. RM 2 menggunakan keputusan dari keluarga global ini.
- RM 2 hanya deskriptif: jumlah, mean, median, simpangan baku `ddof=0`, minimum–maksimum, proporsi arah, dan jumlah/proporsi hasil signifikan setelah BH global.
- RM 3 mengeluarkan satu pasangan sekaligus, menghitung ulang skor dari 14 atribut dengan `ddof=0`, lalu mengukur perubahan skor, arah, dan peringkat. Tidak ada p-value atau FDR tambahan pada LOPO.
- Positif berarti relatif lebih dekat ke atribut laki-laki; negatif relatif lebih dekat ke atribut perempuan; nol menunjukkan perbedaan relatif nol menurut aturan numerik yang didokumentasikan.

Nomor persamaan yang tersedia pada acuan: L2 (2.2)–(2.3), cosine (2.4), SC-WEAT (2.8), statistik permutasi (2.9), p-value (2.10), BH (2.11), dan Spearman (2.12). Persamaan WEAT (2.5)–(2.7) merupakan latar, bukan pengganti pengukuran satu target ini. Nomor tersebut ditautkan sesuai keterangan prompt; naskah persamaan proposal asli belum diperiksa.

## Keputusan teknis implementasi

Normalisasi Unicode NFKC, lowercase, perapian whitespace, kebijakan punctuation, toleransi floating point, struktur output, `run_id`, aturan ranking, dan penyimpanan gambar merupakan keputusan operasional yang didokumentasikan dalam [`metodologi.md`](metodologi.md). Secara khusus, kata di dalam kurung termasuk `umum` dipertahankan dan `/` menjadi spasi. Prompt menyatakan dua perlakuan ini belum dirinci dalam proposal asli.

## Perubahan arah yang berlaku mulai 14 Oktober 2026

Objek sensitivitas adalah pemilihan pasangan istilah gender. Fokus aktif berpindah ke satu notebook dan satu model, `ddof=0`, tiga RM yang lengkap, serta visualisasi statis tersimpan. Notebook lama dan angka historis dipertahankan sebagai arsip. Perbandingan model, `wiki.id.bin`, IndoBERT, LPBS, w-SEAT, variasi preprocessing sebagai analisis tambahan, dan uji beda inferensial antarsubgolongan berada di luar pipeline aktif.

Implementasi dan eksekusi aktual selesai pada run [`2026-10-14_23-08-25`](../outputs/analisis_utama/2026-10-14_23-08-25/), dengan seluruh 23 cell kode dijalankan dan 16 QC PASS. Angka hasil dan jawaban RM dicatat di [`README.md`](../README.md) dan [`catatan_pengerjaan.md`](catatan_pengerjaan.md). Acuan jumlah pada halaman ini tetap berfungsi sebagai spesifikasi audit, bukan pengganti hasil eksekusi.
