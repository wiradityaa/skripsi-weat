# Catatan Versi dan Progress Eksplorasi

Dokumen ini mencatat tujuan, perubahan, hasil validasi, keterbatasan, dan keputusan pada setiap versi. Notebook sintetis diperlakukan sebagai sarana memvalidasi alur kode, bukan sebagai bukti empiris mengenai bias pada Bahasa Indonesia.

## Keputusan metodologis utama

Audit terhadap Caliskan, Bryson, dan Narayanan (2017) menunjukkan bahwa tiga prosedur perlu dibedakan.

1. **WEAT asli** membandingkan dua kelompok target terhadap dua kelompok atribut. Uji permutasinya membagi ulang gabungan kelompok target.
2. **WEFAT** menghitung skor asosiasi terstandarisasi untuk setiap target, lalu menguji hubungannya dengan data faktual eksternal, misalnya persentase perempuan pada suatu pekerjaan.
3. **SC-WEAT** dijelaskan secara eksplisit oleh Caliskan et al. (2022) sebagai asosiasi satu target terhadap dua set atribut dengan sedikitnya delapan stimulus per set. Formulanya sama dengan skor individual yang muncul dalam konteks WEFAT pada Caliskan et al. (2017).
4. **Notebook V1** menghitung formula SC-WEAT per pekerjaan, tetapi uji permutasi dua sisi per pekerjaan dan koreksi FDR merupakan keputusan analisis tambahan. Oleh karena itu, V1 tidak boleh disebut sebagai reproduksi eksperimen Caliskan.

Istilah **SC-WEAT effect size** dapat digunakan untuk skor per pekerjaan dengan menyitasi Caliskan et al. (2022). Hipotesis nol, skema permutasi, sisi pengujian, standar deviasi, dan koreksi pengujian berganda tetap harus dijelaskan dan disitasi secara terpisah karena detail tersebut tidak seluruhnya ditetapkan oleh formula SC-WEAT.

## V1: Alur awal dengan data sintetis

**Artefak utama:** `demo_v1_bias_gender_pekerjaan_scweat.ipynb`

### Tujuan

- Menyusun alur analisis dari data pekerjaan, representasi vektor, skor asosiasi, uji permutasi, koreksi FDR, sensitivitas istilah gender, hingga visualisasi.
- Memastikan bentuk keluaran mudah dipelajari sebelum memakai embedding nyata.

### Hasil pemeriksaan ulang

- Seluruh sel kode dapat dieksekusi secara linear dan seluruh pemeriksaan akhir lulus.
- Data memuat 72 pekerjaan dalam 9 kelompok.
- Rata-rata skor terstandarisasi adalah sekitar -0,241 dan median sekitar -0,404.
- Sebanyak 26 pekerjaan bertanda positif dan 46 bertanda negatif.
- Sebanyak 15 dari 72 pekerjaan lolos koreksi Benjamini-Hochberg pada taraf 0,05.
- Korelasi peringkat pada uji sensitivitas istilah gender tetap tinggi, dengan nilai minimum sekitar 0,972, tetapi terdapat perubahan tanda pada beberapa pekerjaan.

### Interpretasi yang diperbolehkan

Hasil tersebut hanya menunjukkan bahwa pipeline mampu menghasilkan, menguji, mengoreksi, dan membandingkan skor yang ditanamkan pada data sintetis. Hasil tersebut tidak menunjukkan pola bias pada Bahasa Indonesia, pekerjaan di Indonesia, atau model AI nyata.

### Masalah yang ditemukan

- Vektor token bersama dibuat ketika token pertama kali muncul. Akibatnya, vektor dapat mewarisi sinyal pekerjaan pertama dan hasil berpotensi bergantung pada urutan data.
- Hubungan antara sinyal laten yang sengaja ditanamkan dan skor yang dipulihkan hanya sekitar 0,317 untuk Pearson dan 0,320 untuk Spearman. Ini terlalu rendah untuk menyebut simulasi telah memvalidasi kemampuan pemulihan sinyal dengan kuat.
- Nama metode belum cukup tepat karena formula per pekerjaan lebih dekat ke skor individual WEFAT, sedangkan hipotesis nol dan uji permutasinya merupakan perluasan tersendiri.
- Sebanyak 2.000 permutasi Monte Carlo digunakan meskipun pembagian tepat dari 16 atribut menjadi dua kelompok berukuran 8 hanya berjumlah 12.870 dan masih dapat dihitung secara eksak.
- Representasi frasa dengan rata-rata token bukan prosedur yang digunakan Caliskan untuk pekerjaan multi-kata. Dalam eksperimen WEFAT mereka, pekerjaan multi-kata disederhanakan menjadi satu istilah yang lebih umum atau dikeluarkan jika penyederhanaan tidak mungkin.

### Status

**Dibekukan sebagai dokumentasi pengembangan.** V1 tidak digunakan sebagai sumber temuan skripsi.

## V2: Validasi simulasi yang dikoreksi

**Artefak utama:** `demo_v2_validasi_simulasi_scweat.ipynb`

**Status:** selesai dan lolos audit eksekusi linear.

### Tujuan

- Menghilangkan ketergantungan hasil pada urutan pekerjaan.
- Memisahkan sinyal gender dari pusat semantik kelompok pekerjaan.
- Menggunakan token bersama yang benar-benar konsisten.
- Memastikan skor mampu memulihkan sinyal laten dengan hubungan yang kuat.
- Mengganti pendekatan 2.000 permutasi dengan enumerasi eksak 12.870 pembagian atribut.

### Kriteria lulus

- Hasil tidak berubah secara substantif ketika baris pekerjaan diacak.
- Korelasi antara sinyal laten dan skor pulihan tinggi serta arahnya benar.
- Tidak ada vektor hilang, nilai bukan angka, atau penyebut nol.
- Orientasi skor dijelaskan secara eksplisit.
- Seluruh pemeriksaan kualitas lulus setelah kode dijalankan dari awal sampai akhir.

### Hasil

- Sebanyak 30 sel kode berhasil dijalankan dan 22 pemeriksaan akhir lulus.
- Korelasi Pearson antara sinyal laten dan SC-WEAT adalah 0,9162.
- Korelasi Spearman adalah 0,9795.
- Kesesuaian arah sinyal adalah 100%.
- Perbedaan maksimum vektor dan skor setelah baris pekerjaan diacak adalah 0.
- Uji permutasi menghitung seluruh 12.870 pembagian atribut.
- Korelasi sensitivitas leave-one-pair-out terendah adalah 0,9864 dan tidak ada perubahan tanda.
- Sebanyak 68 dari 72 pekerjaan lolos FDR. Angka ini merupakan konsekuensi ruang sintetis dengan sinyal yang sengaja ditanamkan dan bukan temuan empiris mengenai pekerjaan.

### Putusan

V2 berhasil memperbaiki kelemahan generator V1 dan menunjukkan bahwa implementasi dapat memulihkan sinyal buatan secara kuat. V2 tetap hanya validasi internal. Keberhasilannya tidak menjamin hasil pada embedding nyata akan kuat, stabil, atau signifikan.

## V3: Kontrol positif dan reproduksi parsial Caliskan

### Tujuan

Memvalidasi implementasi WEAT formal menggunakan stimulus asli berbahasa Inggris sebelum menerapkannya pada Bahasa Indonesia.

### Eksperimen yang disarankan

1. **WEAT 1, bunga-serangga dan menyenangkan-tidak menyenangkan**, sebagai kontrol positif teknis. Paper melaporkan `d = 1,50` pada GloVe dan `d = 1,54` pada Word2Vec, keduanya dengan `p < 10^-7`.
2. **WEAT 6, nama laki-laki-perempuan dan karier-keluarga**, sebagai kontrol yang paling dekat dengan tema gender dan pekerjaan. Paper melaporkan `d = 1,81; p = 10^-3` pada GloVe dan `d = 1,89; p = 10^-4` pada Word2Vec.

### Batas klaim

Jika embedding yang digunakan tidak sama persis dengan model dalam paper, hasil diberi label **reproduksi parsial** atau **functional validation**, bukan reproduksi eksak. Daftar stimulus dapat digunakan sebagai rancangan eksperimen dengan sitasi, tetapi gambar dan narasi hasil harus dibuat sendiri.

## V4: Validasi skor pekerjaan melalui WEFAT

### Tujuan

- Menggunakan 50 pekerjaan dan dua set atribut gender dari Caliskan.
- Menghubungkan skor embedding setiap pekerjaan dengan data faktual persentase perempuan dalam pekerjaan.
- Membandingkan korelasi dan regresi dengan hasil paper.

### Ketergantungan data

PDF tidak memuat seluruh nilai persentase pekerjaan. Data harus diperoleh dari arsip penelitian atau sumber BLS yang sesuai sebelum versi ini dapat disebut reproduksi. Tanpa data tersebut, versi ini tidak boleh mengarang nilai pembanding.

## V5: Pilot pekerjaan Bahasa Indonesia

### Tujuan

- Menggunakan daftar pekerjaan nyata dari KBJI 2014.
- Menggunakan satu embedding statis Bahasa Indonesia sebagai model utama.
- Mengaudit cakupan kosakata, frekuensi, ambiguitas, dan penanganan pekerjaan multi-kata.
- Menghasilkan skor asosiasi intrinsik yang dapat ditelusuri, tanpa mengklaim diskriminasi atau kondisi pasar kerja Indonesia.

### Kandidat model

- fastText Bahasa Indonesia Common Crawl dan Wikipedia sebagai kandidat utama.
- fastText Bahasa Indonesia Wikipedia sebagai replikasi jika versi dan sumber resminya dapat dikunci.
- Model tambahan hanya menjadi testbed replikasi. Hasilnya tidak digunakan untuk menyimpulkan pengaruh arsitektur.

File model penuh tidak dimasukkan ke repository. Bila model sangat besar, vektor yang diperlukan diekstrak sekali untuk daftar stimulus yang telah dibekukan, disertai metadata sumber, versi, checksum, dimensi, dan prosedur ekstraksi.

Rakivnenko et al. (2024) dapat digunakan sebagai penelitian pembanding karena mereka menghitung selisih asosiasi kosinus pekerjaan terhadap pasangan `woman-man` dan `girl-boy` pada sembilan model embedding. Paper tersebut mendukung pentingnya sensitivitas terhadap pilihan istilah gender dan perbedaan antarmodel. Namun, metodenya bukan WEAT atau SC-WEAT, tidak melaporkan uji permutasi per pekerjaan, dan uraian perhitungannya terbatas. Karena itu, paper tersebut tidak dijadikan sumber utama formula atau inferensi statistik.

## V6: Robustness Bahasa Indonesia

Versi ini baru dijalankan setelah V3 sampai V5 lolos.

- Menguji sensitivitas terhadap pasangan istilah gender dengan leave-one-pair-out.
- Membandingkan kategori istilah gender yang telah divalidasi.
- Membandingkan penyederhanaan kepala kata, rata-rata token, dan vektor frasa bila tersedia untuk pekerjaan multi-kata.
- Mengukur stabilitas tanda, besar skor, dan peringkat pekerjaan.
- Mereplikasi analisis pada model kedua.

## Titik keputusan

Topik masih layak dilanjutkan apabila: implementasi WEAT formal lolos kontrol positif; skor pekerjaan dapat divalidasi; cakupan istilah KBJI pada model Indonesia memadai; dan kesimpulan tetap stabil pada variasi leksikal yang wajar. Jika syarat tersebut gagal, penelitian harus dipersempit atau metodenya diganti sebelum penulisan hasil.

## Batas interpretasi

- Skor embedding merupakan ukuran asosiasi intrinsik, bukan bukti langsung diskriminasi dalam aplikasi nyata.
- Signifikansi statistik tidak sama dengan besarnya dampak sosial.
- Tidak signifikan tidak berarti embedding bebas dari seluruh asosiasi gender.
- Hasil satu model tidak mewakili seluruh model AI.
- Terjemahan stimulus Inggris tidak otomatis valid untuk Bahasa Indonesia.

## Referensi metodologis utama

Caliskan, A., Bryson, J. J., & Narayanan, A. (2017). Semantics derived automatically from language corpora contain human-like biases. *Science, 356*(6334), 183-186. https://doi.org/10.1126/science.aal4230

Caliskan, A., Ajay, P. P., Charlesworth, T., Wolfe, R., & Banaji, M. R. (2022). Gender bias in word embeddings: A comprehensive analysis of frequency, syntax, and semantics. *Proceedings of the 2022 AAAI/ACM Conference on AI, Ethics, and Society*, 156-170. https://doi.org/10.1145/3514094.3534162

Rakivnenko, V., Maslej, N., Cervi, J., & Zhukov, V. (2024). Bias in text embedding models. arXiv:2406.12138.

## Implementasi terbaru: pemrosesan KBJI 2014

Tanggal pengerjaan: 2026-09-07.

Implementasi baru memindahkan sumber target dari data pekerjaan yang dibentuk secara programatis ke PDF KBJI 2014 yang tersedia di data/KBJI 2014.pdf. Notebook test1_v1_processing_data_kbji.ipynb mengekstrak bagian isi Golongan Pokok 2 Profesional sampai sebelum Golongan Pokok 3, mempertahankan hierarki resmi pada tingkat Golongan Pokok, Subgolongan Pokok, Golongan, Subgolongan, dan Jabatan.

Eksekusi Notebook 1 menghasilkan 501 Jabatan mentah, 6 Subgolongan Pokok, 27 Golongan, dan 92 Subgolongan. Sebanyak 415 Jabatan masuk data analisis. Eksklusi terdiri dari 85 kategori residual dan 1 lexical leakage atribut gender. Tidak ada kode Jabatan duplikat dan tidak ada kode tidak valid pada hasil ekstraksi.

Kolom occupation_name_raw mempertahankan nama dari PDF. Kolom occupation_text dinormalisasi secara konservatif dengan lowercase, perapian spasi, penghapusan penanda (Umum), serta normalisasi tanda baca tanpa stemming atau penghapusan stopword agresif. File data lengkap dan data analisis telah terbentuk di folder data dengan awalan test_.

Notebook kedua, test2_v1_pipeline_scweat_fasttext.ipynb, telah disiapkan untuk model binary pretrained fastText Bahasa Indonesia cc.id.300.bin dari Common Crawl dan Wikipedia. Metode yang digunakan adalah agregasi rata-rata token untuk nama Jabatan multi-kata, normalisasi L2, SC-WEAT, exact permutation C(16,8)=12.870, FDR Benjamini-Hochberg, serta leave-one-pair-out.

Status saat pembaruan ini: pemrosesan KBJI lulus eksekusi dan validasi. Pipeline fastText belum dinyatakan selesai sebelum package fastText dan model binary berhasil dimuat pada kernel bersih serta seluruh CSV output dan quality control terbentuk.

Kendala: lingkungan Python yang tersedia adalah Python 3.14 pada Windows; package fasttext dari PyPI membutuhkan kompilasi Microsoft Visual C++ dan belum dapat dipasang pada lingkungan ini. Arsip model resmi sedang dipersiapkan secara lokal dan tidak akan dimasukkan ke Git.

Hal yang masih memerlukan validasi akademik: kelayakan lexical set Bahasa Indonesia, keputusan normalisasi frasa, dampak token OOV, dan batas inferensi dari asosiasi intrinsik embedding ke konteks sosial.

## Hasil eksekusi pipeline fastText

Notebook 2 berhasil dijalankan dari awal sampai akhir menggunakan Python 3.13.15, fasttext-community 0.11.7, dan model resmi cc.id.300.bin berukuran 7.236.808.873 byte dengan dimensi 300. SHA-256 model adalah 46E6E4C3975C2386F76C608C896CB36FA9C90F9D7F8212988C7EEB5FF41D235F. Model berasal dari URL Common Crawl + Wikipedia fastText dan disimpan di luar Git.

Data analisis berisi 415 Jabatan. Agregasi embedding menghasilkan 1.152 token in-vocabulary dan 5 token subword OOV. Seluruh vektor finite dan bukan zero vector. Exact permutation menjalankan 12.870 partisi untuk setiap Jabatan; 0 hasil signifikan pada p-value mentah dan 0 setelah FDR Benjamini-Hochberg.

Ringkasan awal SC-WEAT pada satu model: mean effect size -0,1594, median -0,1605, minimum -0,9062, maksimum 0,8768; 303 Jabatan bertanda negatif dan 112 bertanda positif. Angka ini hanya menunjukkan pola asosiasi relatif intrinsik pada ruang embedding yang diuji.

Leave-one-pair-out menghasilkan 8 iterasi. Korelasi Spearman terendah 0,9283, mean absolute change terbesar 0,2551, dan perubahan tanda terbanyak 149 pada satu kondisi. Hasil ini dipakai sebagai quality control sensitivitas lexical set, bukan sebagai kesimpulan sosial.

Seluruh 10 pemeriksaan quality control Notebook 2 berstatus PASS, termasuk dimensi, finite vector, cosine valid, exact partitions, p-value/FDR, invariansi urutan istilah, jumlah iterasi, dan kesamaan jumlah output dengan data analisis. Seluruh tujuh output CSV test_v1_ terbentuk.

Keterbatasan yang tersisa: model sangat besar, lexical set masih operasional, 5 token OOV memerlukan kehati-hatian, representasi frasa memakai rata-rata token, PDF merupakan edisi KBJI 2014, dan hasil satu model tidak mewakili seluruh embedding atau konteks penggunaan.

## Implementasi `test_v2`: revisi menyeluruh dan hasil aktual

Tanggal pengerjaan: 2026-09-08.

`test1_v2_processing_data_kbji.ipynb` mempertahankan parser PDF yang telah diperiksa pada v1, tetapi menambahkan audit `has_parenthetical_text`, `has_english_or_foreign_term`, `contains_slash`, `representation_notes`, dan `occupation_text_without_parenthetical`. Notebook ini juga membuat `data/test_v2_istilah_gender.csv` sebagai satu-satunya sumber lexical set untuk pipeline berikutnya. Eksekusi menghasilkan 501 Jabatan mentah, 6 Subgolongan Pokok, 27 Golongan, 92 Subgolongan, 85 residual, 1 lexical leakage, dan 415 Jabatan analisis. Quality control hierarchy, prefix mapping, konflik kode-nama, duplikasi teks, nama kosong, dan read-back CSV lulus.

`test2_v2_pipeline_scweat_fasttext.ipynb` membaca data dan lexical set v2, lalu menghitung provenance model langsung. Eksekusi menggunakan Python 3.13.15, `fasttext-community` 0.11.7, model `cc.id.300.bin` berukuran 7.236.808.873 byte, dimensi 300, dan SHA-256 `46E6E4C3975C2386F76C608C896CB36FA9C90F9D7F8212988C7EEB5FF41D235F`. Vocabulary model yang terbaca adalah 2.000.000 token.

SC-WEAT menghasilkan 415 baris utama: mean effect size -0,1594, median -0,1605, minimum -0,9062, maksimum 0,8768, dengan 303 arah negatif dan 112 positif. Exact permutation menjalankan 12.870 partisi untuk setiap Jabatan. Tidak ada hasil signifikan pada p-value mentah maupun setelah FDR.

Bug sign agreement diperbaiki dengan menyimpan 3.320 baris long-format Jabatan × 8 pasangan pada `outputs/test_v2_hasil_sensitivitas_per_jabatan.csv`. Ringkasan per Jabatan menghitung proporsi sign agreement, median dan maksimum absolute change, rentang LOO effect, serta jumlah perubahan tanda. Ringkasan kelompok mengambil median dari nilai Jabatan dalam kelompok; tidak ada scalar global yang disalin. Korelasi Spearman LOO terendah adalah 0,9283, mean absolute change terbesar 0,2551, dan perubahan tanda terbanyak 149.

Audit kontribusi pasangan mempertahankan seluruh delapan pasangan dan memberi penanda khusus untuk `pemuda–pemudi` serta `jejaka–gadis`. Sensitivitas representasi mencakup 43 Jabatan dengan parenthetical, menghasilkan korelasi Spearman 0,9878, mean absolute change 0,0893, dan 5 perubahan tanda. Audit OOV menemukan 5 token OOV pada Jabatan; ringkasan memisahkan 415 Jabatan, 410 tanpa OOV, dan 5 dengan OOV.

Audit model menemukan 16/16 istilah gender memiliki vocabulary ID dan vektor valid, 155 baris nearest-neighbor untuk istilah gender, contoh Jabatan satu kata, dan token penting frasa multi-kata. Mean coherence istilah laki-laki adalah 0,4311, perempuan 0,4875, dan mean cosine antarpasangan 0,8175. Coverage vocabulary tidak dipakai sebagai satu-satunya dasar validitas semantik.

Notebook v2 menghasilkan 14 CSV berawalan `test_v2_`, termasuk hasil utama, permutation, LOO per Jabatan, ringkasan pasangan, kontribusi lexical pair, sensitivitas frasa/OOV, ringkasan kelompok, audit embedding/model/istilah, nearest neighbors, dan validasi. Quality control akhir berisi 15 pemeriksaan dan seluruhnya berstatus PASS. Keputusan operasional: fastText layak sebagai model utama untuk baseline ini, dengan kebutuhan tinjauan linguistik lexical set, inspeksi nearest neighbors, satu-model limitation, dan keterbatasan korpus/usia model.

Keterbatasan v2: lexical set belum tervalidasi final, parenthetical dan foreign-term audit memakai aturan deterministik operasional, fastText subword membuat OOV tidak identik dengan vektor gagal, hasil tidak membuktikan diskriminasi atau komposisi gender tenaga kerja, dan model pembanding belum dijalankan.

## Rancangan `test_v3`: perbandingan model fastText

Tanggal pembaruan: 2026-09-08.

V3 menambahkan model pembanding nyata tanpa mengubah data, lexical set, representasi Jabatan, formula SC-WEAT single-target, atau exact permutation dari baseline. `cc.id.300.bin` ditetapkan sebagai baseline dan `wiki.id.bin` ditetapkan sebagai pembanding sebelum hasil dijalankan. Keduanya adalah fastText Bahasa Indonesia berdimensi 300, tetapi `cc.id.300.bin` berasal dari Common Crawl + Wikipedia dengan konfigurasi CBOW position-weighted, sedangkan `wiki.id.bin` berasal dari Wikipedia dengan konfigurasi skip-gram. Karena itu, v3 menguji sensitivitas terhadap model/korpus fastText yang dibandingkan, bukan pengaruh arsitektur secara murni.

Notebook baru:

- `test1_v3_processing_data_kbji.ipynb` mempertahankan proses ekstraksi dan audit PDF, lalu menggunakan awalan output `test_v3_`.
- `test2_v3_pipeline_scweat_model_comparison.ipynb` memuat dua model, menyimpan provenance, menghitung skor tingkat Jabatan, exact permutation 12.870 partisi, FDR global untuk seluruh pengujian Jabatan × model, ringkasan enam Subgolongan Pokok, leave-one-pair-out, dan audit OOV.

Analisis komparatif v3 menambahkan:

- `model_difference = d_wiki - d_cc` untuk setiap Jabatan;
- kesamaan arah, Spearman rho, dan Kendall tau;
- mean/median absolute model difference;
- Wilcoxon signed-rank;
- paired sign-flip permutation;
- cluster bootstrap 5.000 replikasi dengan `group_code` sebagai unit pengelompokan.

Status: notebook dan checkpoint data v3 telah disiapkan, tetapi belum dieksekusi penuh karena kedua file model biner tidak tersedia di lingkungan pembangunan ini. Tidak ada hasil signifikansi v3 yang boleh ditulis ke dokumentasi sebelum notebook dijalankan dengan kedua model dan seluruh CSV output terbentuk.

Keputusan metodologis: model tidak boleh dipilih atau dikeluarkan setelah melihat p-value. Jika kedua model konsisten, hasilnya mendukung robustness terhadap perubahan korpus; jika berbeda, hasilnya menunjukkan sensitivitas pengukuran terhadap model/korpus. Keduanya merupakan temuan yang sah.

## Implementasi `test2_v4`: IndoBERT LPBS

Tanggal pengerjaan: 2026-09-08.

Tujuan versi ini adalah memakai model masked language yang lebih sesuai untuk nama jabatan profesional multi-kata. Notebook `test2_v4_indobert_lpbs_pipeline.ipynb` membaca kembali `data/test_v3_kbji_profesional_analysis.csv`, `data/test_v3_kbji_profesional_all.csv`, dan `data/test_v3_istilah_gender.csv` tanpa ekstraksi PDF baru dan tanpa menimpa CSV. Model aktual yang digunakan adalah `indobenchmark/indobert-base-p1` dari Hugging Face, revision `c2cd0b51ddce6580eb35263b39b0a1e5fb0a39e2`, vocabulary 30.521, mask token `[MASK]` dengan ID 4, dan device CPU. Eksekusi memakai Python 3.14.0, PyTorch 2.12.1+cpu, Transformers 4.57.6, dan seed 42.

Metode utama adalah Log-Probability Bias Score. Kondisi utama `pria–wanita` ditetapkan a priori dan keduanya menjadi satu token IndoBERT. Sensitivitas `laki-laki–perempuan` juga dihitung a priori; `laki-laki` menjadi tiga subtoken, sehingga diproses dengan pseudo-log-likelihood per subtoken. Empat template jabatan dan empat baseline template yang sama digunakan. Skor utama per jabatan adalah median lintas-template.

Data aktual terdiri dari 501 nama pada CSV lengkap dan 415 jabatan analisis pada enam kelompok 21–26. Output long-format berisi 3.320 baris, yaitu 415 × 4 template × 2 kondisi gender. Cluster bootstrap memakai `group_code` tiga digit dengan 5.000 repetitions dan seed tetap.

Hasil kelompok aktual: median LPBS kode 21 = -0,0834 (CI95% -0,1390 sampai 0,0116), kode 22 = 0,0711 (0,0124 sampai 0,1174), kode 23 = -0,1704 (-0,2502 sampai -0,0804), kode 24 = 0,1094 (0,0733 sampai 0,1721), kode 25 = 0,0419 (0,0229 sampai 0,0573), dan kode 26 = 0,1053 (-0,0570 sampai 0,1780). Kruskal–Wallis eksploratif menghasilkan H 125,0193, p 2,71 × 10⁻²⁵, dan epsilon-squared 0,2934. Ini bukan bukti bahwa profesi di masyarakat diperuntukkan bagi gender tertentu.

Sensitivitas template menunjukkan Spearman 0,2404–0,9200, agreement tanda 0,3108–0,8410, dan median agreement tingkat jabatan 0,7500. Sensitivitas istilah menghasilkan Spearman 0,2468, median absolute difference 0,1636, agreement 0,6024, dan 165 perubahan tanda. Perbandingan dengan fastText/SC-WEAT tersedia untuk 401 jabatan: Spearman -0,0890, agreement 0,4788, dan 209 perubahan tanda. Skala LPBS dan SC-WEAT berbeda, sehingga perbandingan dibatasi pada arah, peringkat, dan agreement.

Output yang dihasilkan: `outputs/test_v4_lpbs_scores_long.csv`, `test_v4_lpbs_occupation_summary.csv`, `test_v4_lpbs_group_summary.csv`, `test_v4_lpbs_template_stability.csv`, `test_v4_lpbs_lexical_sensitivity.csv`, `test_v4_lpbs_tokenization_audit.csv`, `test_v4_lpbs_statistical_tests.csv`, `test_v4_lpbs_fasttext_comparison.csv`, `test_v4_lpbs_fasttext_group_comparison.csv`, `test_v4_lpbs_quality_control.csv`, dan `test_v4_run_metadata.json`.

Quality control lulus: seluruh cell kode berhasil, jumlah baris sesuai, empat template tersedia untuk setiap jabatan, semua probabilitas berada pada rentang 0–1, tidak ada NaN/tak hingga pada skor numerik, tidak ada observasi duplikat, baseline tiap template tersedia, dan SHA-256 tiga CSV input tidak berubah. Durasi eksekusi final 163,02 detik pada CPU. Tidak ada masalah inferensi yang tersisa; peringatan Jupyter hanya terkait environment Windows dan metadata ID cell.

Keputusan metodologis: tidak ada template atau istilah yang dihapus berdasarkan p-value atau besar asosiasi. LPBS diperlakukan sebagai asosiasi model dalam konteks kalimat. Keterbatasan meliputi satu model, dua kondisi gender operasional, template buatan, struktur hierarkis KBJI, dan perbedaan intrinsic dengan downstream. Tahap berikutnya yang disarankan adalah validasi linguistik lexical set dan replikasi pada model kontekstual Indonesia kedua dengan protokol yang sama.
