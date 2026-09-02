# Analisis Bias Gender pada Representasi Pekerjaan Bahasa Indonesia Menggunakan SC-WEAT

Proyek ini membangun pipeline pembelajaran untuk mengukur asosiasi relatif antara nama pekerjaan berbahasa Indonesia dan dua kelompok istilah gender menggunakan **Single-Category Word Embedding Association Test (SC-WEAT)**.

## Fokus analisis

Notebook menjawab empat pertanyaan:

1. Bagaimana arah dan besaran asosiasi gender pada setiap pekerjaan?
2. Pekerjaan apa yang memiliki asosiasi relatif paling dekat dengan masing-masing kelompok gender?
3. Bagaimana distribusi asosiasi pada kelompok pekerjaan?
4. Apakah hasil relatif konsisten ketika sebagian pasangan istilah gender dikeluarkan?

Skor positif berarti representasi pekerjaan lebih dekat ke atribut laki-laki, skor negatif berarti lebih dekat ke atribut perempuan, dan skor mendekati nol berarti tidak ada perbedaan asosiasi yang kuat dalam konfigurasi pengujian.

## Struktur

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

## Menjalankan notebook

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Buka `demo_v1_bias_gender_pekerjaan_scweat.ipynb` dan jalankan semua sel dari atas ke bawah. Data pada versi ini dibentuk secara programatis untuk pembelajaran pipeline; dataset tidak diklaim berasal dari BPS, KBJI, FastText, atau sumber empiris lain.

## Isi pipeline

Notebook membuat data pekerjaan yang tersebar pada sembilan kelompok klasifikasi-semu, data istilah gender berpasangan, representasi embedding berdimensi 300, similarity pekerjaan-gender, effect size SC-WEAT individual, permutation test two-sided, koreksi Benjamini–Hochberg, ringkasan kelompok, dan leave-one-pair-out sebagai quality control.

PCA hanya digunakan untuk visualisasi. Semua grafik ditampilkan langsung di notebook dan tidak disimpan sebagai file gambar. Tabel hasil disimpan sebagai CSV dan juga ditampilkan di notebook.

## Referensi konseptual

- Caliskan, Bryson, dan Narayanan (2017), *Semantics derived automatically from language corpora contain human-like biases*.
- Rakivnenko et al. (2024), untuk analisis asosiasi gender terhadap pekerjaan pada text embedding.
- Dokumentasi FastText, sebagai rujukan format static word embedding.
- BPS KBJI 2014, sebagai rencana sumber klasifikasi pekerjaan pada tahap penelitian berikutnya.

Dataset pada notebook ini tidak menyatakan diri sebagai data dari referensi tersebut.

## Batas interpretasi

SC-WEAT mengukur asosiasi intrinsik dalam ruang embedding, bukan diskriminasi pada aplikasi nyata. Perbedaan kelompok hanya dibaca sebagai pola representasi pada embedding yang dianalisis. Hasil versi ini bukan temuan empiris mengenai masyarakat Indonesia, pasar kerja Indonesia, atau performa FastText tertentu.
