# Panduan pembangunan proyek `skripsi-weat`

## Arah implementasi aktif

Proyek ini menganalisis asosiasi intrinsik gender–jabatan berbahasa Indonesia menggunakan SC-WEAT pada ruang embedding fastText. Sumber jabatan adalah PDF KBJI 2014 yang tersedia di repository; ruang lingkup aktif hanya Golongan Pokok 2 Profesional dan target utama adalah Jabatan berkode enam digit. Klasifikasi hasil tidak boleh diklaim sebagai salinan resmi KBJI tanpa pemeriksaan sumber tambahan.

Implementasi aktif terdiri atas dua notebook linear:

1. `test1_v2_processing_data_kbji.ipynb` untuk ekstraksi dan audit data KBJI serta pembuatan satu CSV lexical set.
2. `test2_v2_pipeline_scweat_fasttext.ipynb` untuk provenance model, embedding, SC-WEAT, permutation exact, FDR, analisis sensitivitas, dan quality control.

File `test_v1` adalah riwayat dan tidak boleh dihapus atau ditimpa. Data sintetis dan arah lama hanya dicatat sebagai sejarah di `VERSI_DAN_PROGRESS.md`, bukan sebagai fokus aktif.

## Aturan perubahan

- Pastikan working directory adalah repository `skripsi-weat` sebelum mengubah file.
- Sebelum penghapusan besar, tampilkan isi folder dan status Git.
- Periksa target penghapusan secara eksplisit; jangan gunakan perintah destruktif pada path yang belum dipastikan.
- Jangan menghapus parent directory atau `.git`.
- Simpan tabel sebagai CSV dan tampilkan juga di notebook. Visualisasi hanya ditampilkan langsung, tanpa `plt.savefig()`.
- Jangan membuat package, class, pipeline object, config manager, factory, atau banyak file Python. Notebook harus linear dan maksimal sekitar tiga fungsi pendek bila benar-benar diperlukan.
- Gunakan `RANDOM_SEED = 42`, library ilmiah yang sudah ada, dan output dengan identifier `test_v2_` untuk implementasi v2.

## Data dan lexical set

- Pertahankan checkpoint ekstraksi: 501 Jabatan mentah, 6 Subgolongan Pokok, 27 Golongan, 92 Subgolongan, 85 kategori residual, 1 lexical leakage, dan 415 Jabatan analisis. Angka tersebut adalah checkpoint, bukan hardcode pemaksa hasil.
- Analisis utama memakai Subgolongan Pokok 21–26. Golongan 211–265 hanya analisis tambahan eksploratif.
- `data/test_v2_istilah_gender.csv` adalah satu-satunya sumber lexical set. Notebook pipeline wajib membaca CSV ini dan tidak boleh mendefinisikan ulang pasangan gender.
- Kolom lexical set minimal: `pair_id`, `male_term`, `female_term`, `lexical_category`, `register`, `validation_status`, `include_baseline`, `notes`.
- Delapan pasangan adalah baseline operasional dan belum boleh disebut tervalidasi final.

## Model dan metode

- Model utama adalah `models/cc.id.300.bin` dari fastText Bahasa Indonesia.
- Provenance model dihitung langsung dalam notebook: nama, ukuran, dimensi, SHA-256 streaming, versi Python, versi `fasttext-community`, URL sumber resmi, dan vocabulary jika tersedia.
- Gunakan agregasi rata-rata token untuk `occupation_text` sebagai baseline serta `occupation_text_without_parenthetical` sebagai pembanding.
- SC-WEAT memakai delapan istilah laki-laki, delapan istilah perempuan, cosine, association difference, effect size dengan `ddof=1`, exact two-sided permutation dengan 12.870 partisi, dan Benjamini–Hochberg FDR.
- Positif berarti lebih dekat relatif ke atribut laki-laki; negatif berarti lebih dekat relatif ke atribut perempuan. Gunakan hanya label `positif`, `negatif`, atau `nol`.
- Jangan mengganti model atau memilih hasil hanya karena menghasilkan temuan lebih menarik.
- Hasil adalah asosiasi intrinsik pada embedding, bukan bukti diskriminasi, komposisi gender tenaga kerja, atau fakta tentang masyarakat.

## Sensitivitas dan quality control wajib

- Leave-one-pair-out harus disimpan long-format pada tingkat Jabatan dan diringkas per pasangan. Ukuran group-level harus dihitung dari nilai tiap Jabatan, bukan scalar global yang disalin ke semua kelompok.
- Audit kontribusi seluruh pasangan, terutama `pemuda–pemudi` dan `jejaka–gadis`, tetapi jangan menghapus pasangan otomatis.
- Audit semantik mencakup vocabulary status, nearest neighbors istilah gender/jabatan/token penting, coherence kelompok gender, cosine antarpasangan, dan kontribusi pasangan.
- Audit OOV harus membedakan seluruh jabatan, tanpa token OOV, dan jabatan yang memiliki token OOV. fastText subword tidak boleh langsung diperlakukan sebagai kesalahan.
- Invariansi istilah harus membandingkan langsung berdasarkan `occupation_code` tanpa mengurutkan skor. Invariansi pekerjaan harus mengacak baris lalu merge kembali berdasarkan kode.
- Seluruh hasil utama dan tambahan harus dapat ditelusuri sampai tingkat Jabatan.

## Validasi dan dokumentasi

- Seluruh cell kedua notebook harus dapat dijalankan dari awal sampai akhir tanpa error dan output harus disimpan di notebook.
- Seluruh angka di `README.md` dan `VERSI_DAN_PROGRESS.md` harus berasal dari eksekusi notebook aktual.
- Output v2 minimal menggunakan nama `test_v2_` sesuai prompt revisi.
- Sel terakhir notebook pipeline wajib berisi assertion dan tabel status quality control.
- Dokumentasi harus memisahkan analisis utama Subgolongan Pokok dari analisis tambahan Golongan.
