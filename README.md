# Analisis Asosiasi Gender–Jabatan dengan SC-WEAT

Repository ini memuat implementasi bertahap analisis asosiasi intrinsik gender–jabatan berbahasa Indonesia menggunakan embedding fastText. Sumber target terbaru adalah PDF KBJI 2014 yang tersedia di repository, dengan ruang lingkup Golongan Pokok 2 Profesional.

## Pertanyaan analisis

1. Bagaimana arah association difference dan effect size SC-WEAT pada setiap Jabatan enam digit?
2. Seberapa stabil hasil terhadap penghapusan satu pasangan lexical set gender?
3. Seberapa besar perubahan hasil ketika parenthetical pada nama Jabatan dihilangkan atau ketika Jabatan dengan token OOV dipisahkan?

Hasil hanya menyatakan pola asosiasi relatif dalam ruang embedding. Positif berarti lebih dekat relatif ke atribut laki-laki, negatif berarti lebih dekat relatif ke atribut perempuan. Association difference mentah dan effect size harus dibaca bersama. Hasil ini bukan bukti diskriminasi, komposisi gender tenaga kerja, atau fakta tentang masyarakat.

## Notebook aktif

1. `test1_v2_processing_data_kbji.ipynb` mengekstrak dan mengaudit PDF, membuat data lengkap/analisis, serta menulis satu-satunya sumber lexical set `data/test_v2_istilah_gender.csv`.
2. `test2_v2_pipeline_scweat_fasttext.ipynb` membaca data tersebut, menghitung provenance model, embedding frasa, SC-WEAT, exact permutation, FDR, leave-one-pair-out, audit semantik, sensitivitas frasa, dan sensitivitas OOV.

Notebook dan output `test_v1` dipertahankan sebagai riwayat. `test_v2` tidak menimpa artefak tersebut.

## Data dan kelompok

Ekstraksi aktual menghasilkan 501 Jabatan mentah, 6 Subgolongan Pokok, 27 Golongan, 92 Subgolongan, 85 kategori residual, 1 lexical leakage, dan 415 Jabatan analisis. Analisis utama memakai enam Subgolongan Pokok 21–26. Golongan 211–265 hanya analisis tambahan eksploratif dan tidak diberi bobot pembahasan yang sama.

Nama asli disimpan pada `occupation_name_raw`. `occupation_text` adalah representasi baseline; `occupation_text_without_parenthetical` hanya digunakan sebagai pembanding. Normalisasi deterministik tidak memakai stemming atau stopword removal agresif.

Lexical set berisi delapan pasangan operasional: `laki-laki–perempuan`, `pria–wanita`, `ayah–ibu`, `suami–istri`, `putra–putri`, `pemuda–pemudi`, `cowok–cewek`, dan `jejaka–gadis`. CSV mencatat kategori, register, status validasi, dan catatan; pasangan ini belum dianggap tervalidasi linguistik final.

## Model dan metode

Model utama adalah `models/cc.id.300.bin`, pretrained fastText Bahasa Indonesia dari Common Crawl dan Wikipedia, dimensi 300. Sumber resmi model: https://fasttext.cc/docs/en/crawl-vectors. File model lokal tidak dimasukkan ke Git karena ukurannya sangat besar.

Nama Jabatan multi-kata direpresentasikan dengan rata-rata vektor token lalu normalisasi L2. fastText tetap dapat menghasilkan subword vector untuk token OOV; karena itu OOV diaudit, bukan otomatis dibuang.

SC-WEAT memakai delapan istilah per atribut, simpangan baku sampel (`ddof=1`), exact two-sided permutation seluruh C(16, 8) = 12.870 pembagian label, dan koreksi Benjamini–Hochberg FDR. Permutation test adalah keputusan operasional untuk menguji pengacakan label atribut pada satu target Jabatan.

## Hasil `test_v2` dari eksekusi aktual

- 415 Jabatan dianalisis; 303 effect size negatif dan 112 positif.
- Mean effect size -0,1594, median -0,1605, minimum -0,9062, maksimum 0,8768.
- Tidak ada hasil signifikan pada p-value mentah maupun setelah FDR.
- Leave-one-pair-out menghasilkan 3.320 baris Jabatan × 8 pasangan. Korelasi Spearman terendah 0,9283, mean absolute change terbesar 0,2551, dan perubahan tanda terbanyak 149.
- Sensitivitas parenthetical mencakup 43 Jabatan: korelasi Spearman 0,9878 pada perbandingan seluruh representasi, mean absolute change 0,0893, dan 5 perubahan tanda.
- Audit vocabulary menemukan 16/16 istilah gender masuk vocabulary dan 5 token OOV pada Jabatan; seluruh vektor valid.
- Audit coherence: mean cosine antarterm laki-laki 0,4311 dan perempuan 0,4875; mean cosine antarpasangan 0,8175. Angka ini adalah audit deskriptif, bukan validasi semantik final.
- Keputusan operasional: fastText layak sebagai model utama untuk baseline ini, dengan keterbatasan satu model, korpus/usia model, lexical set operasional, dan kebutuhan inspeksi nearest neighbors.

Hasil `test_v1` sebelumnya juga tidak menghasilkan signifikansi; `test_v2` ditujukan untuk menelusuri sumber ketidakstabilan, bukan untuk mencari hasil yang lebih kuat.

## Output

Data:

- `data/test_v2_kbji_profesional_all.csv`
- `data/test_v2_kbji_profesional_analysis.csv`
- `data/test_v2_istilah_gender.csv`

Hasil dan audit:

- `outputs/test_v2_hasil_scweat.csv`
- `outputs/test_v2_hasil_permutasi.csv`
- `outputs/test_v2_hasil_sensitivitas_per_jabatan.csv`
- `outputs/test_v2_ringkasan_sensitivitas_pasangan.csv`
- `outputs/test_v2_kontribusi_pasangan_gender.csv`
- `outputs/test_v2_sensitivitas_representasi_frasa.csv`
- `outputs/test_v2_sensitivitas_oov.csv`
- `outputs/test_v2_ringkasan_subgolongan_pokok.csv`
- `outputs/test_v2_ringkasan_golongan.csv`
- `outputs/test_v2_audit_embedding.csv`
- `outputs/test_v2_audit_model.csv`
- `outputs/test_v2_audit_istilah_gender.csv`
- `outputs/test_v2_nearest_neighbors.csv`
- `outputs/test_v2_validasi_pipeline.csv`

## Menjalankan

Pasang dependensi dari `requirements.txt`, letakkan `cc.id.300.bin` di `models/`, lalu jalankan notebook secara berurutan dari root repository. Gunakan Python 3.11–3.13 pada Windows atau lingkungan Linux yang kompatibel dengan `fasttext-community`.

```powershell
pip install -r requirements.txt
jupyter notebook
```

Notebook terakhir setiap pipeline harus menyimpan output dan tabel status validasi. Visualisasi tampil langsung dan tidak disimpan sebagai gambar.

## Batas interpretasi

SC-WEAT mengukur asosiasi intrinsik pada satu ruang embedding. Effect size besar tidak otomatis berarti association difference mentah besar atau dampak sosial besar. Tidak signifikan tidak berarti embedding bebas dari seluruh asosiasi gender. Satu model fastText, satu sumber PDF, dan delapan pasangan operasional tidak mewakili seluruh bahasa, model, masyarakat, atau konteks downstream. Validitas lexical set dan penilaian nearest neighbors masih memerlukan tinjauan linguistik/akademik.

## Referensi konseptual

- Caliskan, Bryson, dan Narayanan (2017), *Semantics derived automatically from language corpora contain human-like biases*, Science. https://doi.org/10.1126/science.aal4230
- Caliskan et al. (2022), *Gender bias in word embeddings: A comprehensive analysis of frequency, syntax, and semantics*. https://doi.org/10.1145/3514094.3534162
- Rakivnenko et al. (2024), *Bias in text embedding models*. https://arxiv.org/abs/2406.12138
- fastText, *Word vectors for 157 languages*. https://fasttext.cc/docs/en/crawl-vectors
- BPS, KBJI 2014 sebagai rencana rujukan klasifikasi tahap berikutnya; PDF yang dianalisis di notebook tetap diperlakukan sebagai sumber lokal yang perlu diaudit.
