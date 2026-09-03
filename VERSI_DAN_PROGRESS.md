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
