# Analisis Asosiasi Gender pada Jabatan KBJI 2014 Menggunakan SC-WEAT

Repository ini mengimplementasikan analisis asosiasi relatif gender-pekerjaan pada representasi embedding Bahasa Indonesia. Fokus implementasi adalah Golongan Pokok 2, Profesional, dari PDF KBJI 2014 yang tersedia lokal. Targetnya adalah nama Jabatan berkode enam digit; hasil dibaca sebagai asosiasi intrinsik dalam ruang embedding.

## Pertanyaan analisis

1. Bagaimana arah dan effect size SC-WEAT pada setiap nama Jabatan?
2. Bagaimana pola tersebut tersebar pada Subgolongan Pokok dan Golongan?
3. Seberapa stabil peringkat dan arah hasil ketika satu pasangan istilah gender dikeluarkan?

Skor positif berarti relatif lebih dekat ke istilah laki-laki, skor negatif berarti relatif lebih dekat ke istilah perempuan, dan nilai mendekati nol menunjukkan perbedaan asosiasi relatif yang kecil dalam konfigurasi pengujian. Tanda skor bukan label normatif dan bukan label pekerjaan untuk gender tertentu.

## Notebook dan data

Urutan utama:

1. test1_v1_processing_data_kbji.ipynb membaca data/KBJI 2014.pdf, mengekstrak hierarki Golongan Pokok 2 sampai sebelum Golongan Pokok 3, menormalisasi nama secara konservatif, dan menyimpan data lengkap serta data analisis.
2. test2_v1_pipeline_scweat_fasttext.ipynb memuat data/test_kbji_profesional_analysis.csv, membangun representasi frasa dari token fastText, menghitung SC-WEAT, exact permutation, FDR, leave-one-pair-out, dan ringkasan kelompok.

Data lengkap data/test_kbji_profesional_all.csv mempertahankan baris residual, audit validitas, lexical leakage, dan alasan eksklusi. Data/test_kbji_profesional_analysis.csv hanya memuat baris dengan include_analysis == True.

Hierarki KBJI disimpan pada kolom major_group_code, major_group_name, submajor_group_code, submajor_group_name, group_code, group_name, subgroup_code, subgroup_name, occupation_code, occupation_name_raw, dan occupation_text. PDF yang digunakan adalah berkas lokal KBJI 2014; dataset keluaran bukan dataset yang diklaim berasal dari referensi metodologis.

## Model fastText

Notebook 2 menggunakan cc.id.300.bin, model binary pretrained Bahasa Indonesia dari fastText yang dilatih pada Common Crawl dan Wikipedia, dimensi 300. Eksekusi tervalidasi menggunakan fasttext-community 0.11.7 yang diimpor sebagai fasttext. Sumber resmi dan URL unduhan:

https://fasttext.cc/docs/en/crawl-vectors

Unduh dan ekstrak cc.id.300.bin.gz menjadi models/cc.id.300.bin. Model yang digunakan pada eksekusi berukuran 7,236,808,873 byte dengan SHA-256 46E6E4C3975C2386F76C608C896CB36FA9C90F9D7F8212988C7EEB5FF41D235F. File model dan arsip terkompresi tidak dimasukkan ke Git. Notebook memeriksa keberadaan, ukuran minimum, dan dimensi model; tidak ada random vector atau model pengganti.

Nama jabatan multi-kata ditokenisasi, setiap token diberi vektor fastText, lalu dirata-ratakan dan dinormalisasi L2. Status in-vocabulary dan subword OOV dicatat pada outputs/test_v1_audit_embedding.csv. Pendekatan rata-rata token adalah keputusan operasional dan memiliki keterbatasan semantik.

## Metode

SC-WEAT menghitung rata-rata cosine nama Jabatan terhadap delapan istilah laki-laki dan delapan istilah perempuan, selisih rata-rata, dan effect size dengan sample standard deviation (ddof=1). Exact two-sided permutation mengevaluasi seluruh C(16, 8) = 12.870 pembagian atribut. P-value dikoreksi dengan Benjamini-Hochberg FDR.

Lexical set awal terdiri dari pasangan laki-laki–perempuan, pria–wanita, ayah–ibu, suami–istri, putra–putri, pemuda–pemudi, cowok–cewek, dan jejaka–gadis. Daftar ini bersifat operasional dan masih memerlukan validasi akademik/linguistik.

Leave-one-pair-out mengeluarkan satu pasangan pada setiap iterasi dan membandingkan korelasi, perubahan absolut, kesesuaian arah, serta perubahan tanda. Analisis ini adalah quality control terhadap pilihan leksikal, bukan bukti ketergantungan sosial nyata.

## Output

- outputs/test_v1_hasil_scweat.csv
- outputs/test_v1_hasil_permutasi.csv
- outputs/test_v1_hasil_sensitivitas.csv
- outputs/test_v1_ringkasan_subgolongan_pokok.csv
- outputs/test_v1_ringkasan_golongan.csv
- outputs/test_v1_audit_embedding.csv
- outputs/test_v1_validasi_pipeline.csv

Visualisasi ditampilkan langsung di notebook dan tidak disimpan sebagai file gambar.

## Menjalankan

Gunakan Python yang kompatibel dengan paket fastText, sebaiknya Python 3.11–3.13 pada Windows atau lingkungan Linux dengan compiler C++:

python -m venv .venv
pip install -r requirements.txt

Letakkan PDF di data/KBJI 2014.pdf, letakkan model di models/cc.id.300.bin, lalu jalankan Notebook 1 dan Notebook 2 dari root repository. Model lebih besar dari file biasa dan tidak dikelola Git.

## Batas interpretasi

SC-WEAT mengukur asosiasi intrinsik dalam satu ruang embedding. Hasil tidak membuktikan komposisi gender tenaga kerja, diskriminasi perekrutan, kesesuaian pekerjaan untuk gender tertentu, atau fairness aplikasi downstream. Satu model fastText dan satu lexical set tidak mewakili seluruh model, bahasa, masyarakat, atau konteks penggunaan. Status residual, lexical leakage, dan kualitas embedding OOV perlu ditinjau sebelum menarik kesimpulan akademik.

## Referensi konseptual

- Caliskan, Bryson, dan Narayanan (2017), Semantics derived automatically from language corpora contain human-like biases, https://doi.org/10.1126/science.aal4230
- Caliskan et al. (2022), Gender bias in word embeddings: A comprehensive analysis of frequency, syntax, and semantics, https://doi.org/10.1145/3514094.3534162
- Rakivnenko et al. (2024), Bias in text embedding models, https://arxiv.org/abs/2406.12138
- fastText, Word vectors for 157 languages, https://fasttext.cc/docs/en/crawl-vectors
- BPS dan Kementerian Ketenagakerjaan, KBJI 2014, sebagai sumber klasifikasi pada berkas PDF lokal dan dasar struktur hierarki tahap ini.
