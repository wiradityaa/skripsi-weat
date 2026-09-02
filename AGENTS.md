# Panduan pembangunan proyek `skripsi-weat`

Dokumen ini menjadi acuan saat membangun notebook penelitian berikutnya. Arah proyek saat ini adalah **Analisis Bias Gender pada Representasi Pekerjaan Bahasa Indonesia Menggunakan Single-Category Word Embedding Association Test (SC-WEAT)**.

## Aturan perubahan proyek

- Pastikan working directory benar-benar `skripsi-weat` sebelum mengubah file.
- Sebelum penghapusan besar, tampilkan isi folder dan status Git.
- Hapus implementasi lama di dalam project jika diminta, tetapi jangan menghapus folder parent atau `.git`.
- Pastikan target penghapusan sudah diperiksa secara eksplisit; jangan memakai perintah destruktif pada path yang belum dipastikan.
- Setelah penghapusan, tampilkan kembali struktur folder sebelum membangun ulang.
- Jangan mempertahankan arah lama Gender-Career WEAT, atribut career-family, atau sensitivity analysis sebagai fokus utama.

## Struktur target

```text
skripsi-weat/
├── demo_v1_bias_gender_pekerjaan_scweat.ipynb
├── data/
│   ├── demo_v1_data_pekerjaan.csv
│   └── demo_v1_istilah_gender.csv
├── outputs/
│   ├── demo_v1_hasil_scweat.csv
│   ├── demo_v1_ringkasan_kelompok.csv
│   ├── demo_v1_hasil_permutasi.csv
│   └── demo_v1_hasil_sensitivitas.csv
├── README.md
├── requirements.txt
└── .gitignore
```

- Seluruh proses utama harus berada dalam satu notebook.
- Jangan membuat banyak file Python, package, class, pipeline object, config manager, factory, atau abstraksi serupa.
- Visualisasi tidak disimpan sebagai PNG, JPG, SVG, atau PDF. Tampilkan langsung di notebook.
- Tabel boleh disimpan sebagai CSV, tetapi juga wajib ditampilkan di notebook.
- Gunakan identifier `demo_v1_` secara konsisten pada notebook, data, dan output.

## Filosofi dan struktur notebook

Notebook harus linear dan mudah dibaca dari atas ke bawah, dengan urutan:

1. judul, tujuan, dan batas interpretasi;
2. import dan konfigurasi;
3. pembuatan data pekerjaan;
4. eksplorasi data pekerjaan;
5. pembuatan istilah gender;
6. pembentukan representasi embedding;
7. visualisasi ruang embedding;
8. perhitungan similarity;
9. perhitungan SC-WEAT;
10. permutation test dan FDR;
11. analisis pekerjaan;
12. ringkasan kelompok pekerjaan;
13. analisis sensitivitas sebagai quality control;
14. ringkasan hasil;
15. validasi akhir.

Setiap bagian harus memiliki judul Markdown, penjelasan singkat, kode, output yang terlihat, dan interpretasi singkat jika diperlukan.

Jangan menyembunyikan alur utama dalam satu fungsi pipeline. Hindari banyak `def`; maksimal sekitar tiga fungsi utama, hanya untuk proses yang benar-benar berulang. Fungsi yang diizinkan terutama `get_phrase_vector` dan `calculate_sc_weat`. Jangan menggunakan class.

## Bahasa dan interpretasi

- Fokus utama adalah asosiasi gender-pekerjaan pada ruang embedding.
- Gunakan orientasi: skor positif = lebih dekat ke atribut laki-laki; skor negatif = lebih dekat ke atribut perempuan; mendekati nol = tidak ada perbedaan asosiasi yang kuat pada konfigurasi pengujian.
- Gunakan istilah **asosiasi relatif dalam ruang embedding**.
- Jangan menulis “pekerjaan laki-laki”, “pekerjaan perempuan”, “model mendiskriminasi”, atau kesimpulan normatif.
- Tegaskan bahwa SC-WEAT mengukur asosiasi intrinsik dan tidak membuktikan diskriminasi pada aplikasi nyata.
- Jangan mengklaim hasil sebagai fakta tentang masyarakat Indonesia, pasar kerja, BPS, KBJI, atau FastText.
- Jangan menggunakan istilah “smoke test”, “toy example”, “dummy”, atau keterangan berulang bahwa notebook adalah demo. Prefix filename sudah menjadi identifier tahap pengembangan.

## Data pekerjaan

Bentuk sekitar 60–90 pekerjaan berbahasa Indonesia dengan kolom:

```text
occupation_id
occupation_name
major_group_code
major_group_name
token_count
```

Sebarkan pekerjaan ke kelompok yang menyerupai klasifikasi pekerjaan:

1. manajer;
2. profesional;
3. teknisi dan asisten profesional;
4. tenaga tata usaha;
5. tenaga usaha jasa dan penjualan;
6. pekerja terampil pertanian dan perikanan;
7. pekerja pengolahan dan kerajinan;
8. operator dan perakit mesin;
9. pekerjaan elementer.

Kode pekerjaan harus realistis dan konsisten, misalnya `101`, `102`, dan seterusnya. Kelompok tersebut tidak boleh diklaim sebagai salinan resmi KBJI.

Data dibentuk secara programatis. Jangan membuat sumber atau sitasi palsu dan jangan menyatakan data berasal dari BPS/KBJI.

Simpan ke `data/demo_v1_data_pekerjaan.csv`. Setelah membuatnya, tampilkan `head(10)`, shape, dtypes, nilai kosong, duplikasi, jumlah per kelompok, dan distribusi pekerjaan satu kata versus multi-kata. Tampilkan juga tabel contoh tiap kelompok serta visualisasi jumlah pekerjaan per kelompok dan distribusi token.

## Data istilah gender

Gunakan DataFrame dengan kolom:

```text
term_id
term
gender_group
lexical_category
pair_id
```

Kandidat operasional yang realistis:

```text
laki-laki – perempuan
pria – wanita
ayah – ibu
suami – istri
putra – putri
pemuda – pemudi
cowok – cewek
jejaka – gadis
```

Kategori leksikal dapat mencakup identitas umum, kekerabatan, perkawinan, usia/generasi, dan informal. Simpan ke `data/demo_v1_istilah_gender.csv`, tampilkan seluruh tabel, jumlah per kelompok/kategori, pasangan tidak lengkap, dan duplikasi. Jelaskan bahwa kategori berbeda dapat membawa konteks semantik tambahan.

## Representasi embedding

Bangun representasi berdimensi 300 untuk seluruh token pekerjaan, istilah gender, dan nama pekerjaan. Gunakan operasi vektor nyata, bukan mengarang skor akhir.

Representasi harus memiliki satu ruang embedding, arah laten gender, pusat laki-laki, pusat perempuan, pusat semantik kelompok pekerjaan, variasi token/pekerjaan, dan campuran pekerjaan dengan asosiasi relatif positif, negatif, serta mendekati nol. Jangan membuat semua pekerjaan mengarah ke satu kelompok.

Untuk pekerjaan multi-kata, gunakan rata-rata vektor token penyusunnya melalui satu fungsi pendek `get_phrase_vector(text, token_vectors)`. Tampilkan jumlah token unik, dimensi 300, lima elemen pertama beberapa vektor, norma, pemeriksaan NaN/infinity, dan shape matriks embedding pekerjaan.

PCA hanya untuk visualisasi, bukan ukuran utama bias. Tampilkan centroid gender, beberapa pekerjaan, dan seluruh pekerjaan berdasarkan kelompok.

## Similarity dan SC-WEAT

Hitung untuk setiap pekerjaan rata-rata cosine terhadap seluruh istilah laki-laki, rata-rata cosine terhadap seluruh istilah perempuan, selisih similarity, dan similarity terhadap setiap istilah gender. Tampilkan tabel, pekerjaan dengan similarity tertinggi pada masing-masing kelompok, scatter plot dengan garis diagonal, dan heatmap pekerjaan terpilih.

Gunakan satu fungsi utama `calculate_sc_weat(occupation_vector, male_vectors, female_vectors)` yang mengembalikan mean male similarity, mean female similarity, raw association difference, dan effect size. Gunakan sample atau population standard deviation secara konsisten dan dokumentasikan pilihannya. Tangani standar deviasi nol dengan aman.

Hasil utama minimal memuat:

```text
occupation_id
occupation_name
major_group_code
major_group_name
mean_male_similarity
mean_female_similarity
association_difference
scweat_effect_size
association_direction
```

Jangan memberi label “bias kuat/sedang/tidak bias” memakai ambang arbitrer. Gunakan angka dan arah asosiasi.

## Permutation test dan multiple testing

Gunakan `N_PERMUTATIONS = 2000` dan two-sided p-value untuk setiap pekerjaan. Gabungkan atribut gender, acak pembagian dengan ukuran sama, lalu bandingkan statistik permutasi dengan statistik observasi.

Gunakan Benjamini–Hochberg melalui `multipletests(..., method="fdr_bh")`. Simpan `p_value`, `adjusted_p_value`, `significant_raw`, dan `significant_fdr`.

Tampilkan jumlah signifikan sebelum/sesudah FDR, tabel adjusted p-value terkecil, distribusi p-value, dan scatter effect size terhadap `-log10(adjusted_p_value)`.

Simpan hasil ke:

- `outputs/demo_v1_hasil_scweat.csv`
- `outputs/demo_v1_hasil_permutasi.csv`

## Analisis pekerjaan dan kelompok

Urutkan pekerjaan berdasarkan effect size. Tampilkan horizontal bar chart sepuluh asosiasi relatif laki-laki tertinggi, sepuluh asosiasi relatif perempuan tertinggi, histogram/KDE effect size dengan garis nol, tabel nilai terdekat ke nol, dan tabel hasil lengkap.

Ringkas per `major_group_name` dengan jumlah, mean, median, standar deviasi, minimum, maksimum, proporsi positif/negatif, dan jumlah signifikan FDR. Simpan ke `outputs/demo_v1_ringkasan_kelompok.csv`.

Tampilkan boxplot + strip/swarm plot, median effect size, heatmap rata-rata similarity per kelompok, dan proporsi arah asosiasi. Interpretasikan hanya sebagai pola ruang embedding.

## Sensitivitas sebagai quality control

Sensitivity analysis adalah bagian sekunder. Gunakan leave-one-pair-out: keluarkan satu pasangan gender, hitung ulang skor seluruh pekerjaan, lalu bandingkan dengan baseline.

Untuk setiap pasangan yang dikeluarkan, simpan korelasi Spearman peringkat pekerjaan, mean absolute change, proporsi arah tetap, dan jumlah perubahan tanda. Simpan ke `outputs/demo_v1_hasil_sensitivitas.csv`.

Tampilkan tabel dan visualisasi korelasi, mean absolute change, jumlah perubahan tanda, serta heatmap perubahan effect size untuk pekerjaan terpilih. Jangan menyimpulkan ketergantungan sosial nyata dari analisis ini.

## Gaya kode dan visualisasi

- Gunakan `RANDOM_SEED = 42` dan `rng = np.random.default_rng(RANDOM_SEED)`.
- Gunakan library umum: NumPy, pandas, SciPy, scikit-learn, statsmodels, Matplotlib, seaborn, IPython display.
- Tampilkan DataFrame setelah transformasi penting dengan `display()` atau `.head()`.
- Tampilkan shape, dtypes, nilai kosong, duplikasi, dan statistik deskriptif.
- Hindari komentar panjang; gunakan Markdown untuk penjelasan konsep.
- Jangan gunakan emoji atau output yang ditulis manual.
- Jangan gunakan `plt.savefig()`; semua plot harus terlihat langsung dan ditutup dengan `plt.show()` serta `tight_layout()`.
- Label panjang harus terbaca dan visualisasi tidak boleh terpotong.

Visualisasi minimum mencakup data kelompok/token, PCA, similarity, heatmap, bar chart asosiasi, distribusi effect size, p-value/FDR, analisis kelompok, dan sensitivity analysis.

## Validasi akhir

Sel terakhir wajib memakai `assert` dan menampilkan tabel status. Periksa occupation ID duplikat, nama kosong, pasangan gender lengkap, ukuran atribut gender sama, dimensi vektor 300, NaN/infinity, cosine di rentang valid, standar deviasi nol yang tertangani, p-value/adjusted p-value di rentang 0–1, seluruh CSV tersedia, dan jumlah baris hasil sama dengan jumlah pekerjaan.

## README dan dependensi

README harus menjelaskan fokus SC-WEAT, pertanyaan analisis, struktur, cara menjalankan notebook, output, orientasi skor, status data yang dibentuk secara programatis, batas interpretasi, dan referensi konseptual:

1. Caliskan, Bryson, dan Narayanan (2017) untuk WEAT;
2. Rakivnenko et al. (2024) untuk asosiasi gender-pekerjaan pada text embedding;
3. dokumentasi FastText;
4. BPS KBJI 2014 sebagai rencana sumber klasifikasi tahap berikutnya.

Jangan menyatakan dataset notebook berasal dari referensi tersebut.

`requirements.txt` minimal memuat:

```text
numpy
pandas
scipy
scikit-learn
statsmodels
matplotlib
seaborn
jupyter
nbconvert
```

`.gitignore` hanya perlu mengabaikan checkpoint/cache umum seperti `.ipynb_checkpoints/`, `__pycache__/`, `*.pyc`, dan `.DS_Store`. Jangan mengabaikan notebook, CSV data, atau CSV hasil.

## Kriteria selesai

Notebook harus dijalankan dari awal sampai akhir tanpa cell error. Seluruh tabel dan visualisasi harus muncul, seluruh CSV harus terbentuk dengan prefix yang benar, output mudah diikuti, dan angka pada pembahasan harus berasal dari eksekusi kode. Laporan akhir menyebut file lama yang dihapus, struktur baru, jumlah pekerjaan/kelompok/visualisasi, CSV, status eksekusi, status validasi, dan keterbatasan—tanpa mengklaim hasil sebagai penelitian empiris final.
