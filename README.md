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
3. `test1_v3_processing_data_kbji.ipynb` mengulang checkpoint pemrosesan KBJI dengan identifier `test_v3_`.
4. `test2_v3_pipeline_scweat_model_comparison.ipynb` membandingkan `cc.id.300.bin` dengan `wiki.id.bin` menggunakan data, lexical set, representasi, formula, dan exact permutation yang sama. Notebook ini menambahkan perbandingan skor berpasangan, agreement, uji sign-flip, Wilcoxon, dan cluster bootstrap.
5. `test2_v4_indobert_lpbs_pipeline.ipynb` mengukur perubahan probabilitas istilah gender dalam konteks nama jabatan menggunakan IndoBERT dan Log-Probability Bias Score (LPBS).

Notebook dan output `test_v1` dipertahankan sebagai riwayat. `test_v2` dan `test_v3` menggunakan prefix masing-masing sehingga tidak menimpa artefak versi sebelumnya.

## Data dan kelompok

Ekstraksi aktual menghasilkan 501 Jabatan mentah, 6 Subgolongan Pokok, 27 Golongan, 92 Subgolongan, 85 kategori residual, 1 lexical leakage, dan 415 Jabatan analisis. Analisis utama memakai enam Subgolongan Pokok 21–26. Golongan 211–265 hanya analisis tambahan eksploratif dan tidak diberi bobot pembahasan yang sama.

Nama asli disimpan pada `occupation_name_raw`. `occupation_text` adalah representasi baseline; `occupation_text_without_parenthetical` hanya digunakan sebagai pembanding. Normalisasi deterministik tidak memakai stemming atau stopword removal agresif.

Lexical set berisi delapan pasangan operasional: `laki-laki–perempuan`, `pria–wanita`, `ayah–ibu`, `suami–istri`, `putra–putri`, `pemuda–pemudi`, `cowok–cewek`, dan `jejaka–gadis`. CSV mencatat kategori, register, status validasi, dan catatan; pasangan ini belum dianggap tervalidasi linguistik final.

## Model dan metode

Model baseline adalah `models/cc.id.300.bin`, pretrained fastText Bahasa Indonesia dari Common Crawl dan Wikipedia, dimensi 300. Sumber resmi model: https://fasttext.cc/docs/en/crawl-vectors.html. Untuk `test_v3`, model pembanding yang ditetapkan sebelum eksekusi adalah `models/wiki.id.bin`, pretrained fastText Bahasa Indonesia dari Wikipedia, dimensi 300, sumber resmi https://fasttext.cc/docs/en/pretrained-vectors.html. File model lokal tidak dimasukkan ke Git karena ukurannya sangat besar.

Perbandingan v3 bukan uji arsitektur murni. Kedua model sama-sama fastText, tetapi berbeda pada sumber korpus dan objective pelatihan. Oleh karena itu, interpretasi v3 dibatasi sebagai sensitivitas asosiasi terhadap model/korpus fastText yang dibandingkan.

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

## Rancangan `test_v3`

`test1_v3_processing_data_kbji.ipynb` mempertahankan proses ekstraksi KBJI v2 dan menulis checkpoint data dengan awalan `test_v3_`. `test2_v3_pipeline_scweat_model_comparison.ipynb` menjalankan pipeline yang sama pada dua model statis yang ditetapkan a priori.

Outcome utama v3 bukan jumlah p-value signifikan. Notebook menyimpan effect size dan p-value exact per Jabatan × model, FDR global untuk seluruh pengujian tersebut, perbedaan skor `wiki - cc`, kesamaan arah, korelasi Spearman/Kendall, median absolute difference, Wilcoxon, paired sign-flip permutation, dan cluster bootstrap berbasis `group_code`.

Model tidak boleh dipilih atau dihapus berdasarkan hasil. Jika kedua model konsisten, temuan utamanya adalah robustness terhadap sumber korpus; jika berbeda, temuan utamanya adalah sensitivitas pengukuran terhadap model/korpus. Keduanya adalah hasil yang sah. Signifikansi tidak dapat dijanjikan sebelum notebook dijalankan dengan kedua file model.

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

Output v3 terbentuk setelah notebook model dijalankan dengan kedua model:

- `outputs/test_v3_hasil_scweat_per_model.csv`
- `outputs/test_v3_hasil_permutasi_per_model.csv`
- `outputs/test_v3_ringkasan_model.csv`
- `outputs/test_v3_inferensi_perbedaan_model.csv`
- `outputs/test_v3_kesepakatan_model.csv`
- `outputs/test_v3_perbandingan_model_per_jabatan.csv`
- `outputs/test_v3_ringkasan_subgolongan_per_model.csv`
- `outputs/test_v3_sensitivitas_pasangan_per_model.csv`
- `outputs/test_v3_ringkasan_sensitivitas_pasangan_per_model.csv`
- `outputs/test_v3_audit_model.csv`
- `outputs/test_v3_audit_istilah_gender.csv`
- `outputs/test_v3_audit_target_oov.csv`
- `outputs/test_v3_validasi_pipeline.csv`

## Menjalankan

Pasang dependensi dari `requirements.txt`, letakkan `cc.id.300.bin` dan `wiki.id.bin` di `models/`, lalu jalankan notebook v3 secara berurutan dari root repository. Model tersedia dari halaman resmi fastText; gunakan URL dan lisensi yang didokumentasikan di halaman tersebut. Gunakan Python 3.11–3.13 pada Windows atau lingkungan Linux yang kompatibel dengan `fasttext-community`.

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

## test2_v4: IndoBERT dan LPBS

Analisis ini dibuat untuk nama jabatan profesional KBJI 2014 yang didominasi frasa multi-kata. Model utama adalah `indobenchmark/indobert-base-p1`; LPBS membandingkan perubahan log-probability istilah gender pada kalimat yang memuat jabatan dengan baseline template yang sama. Ini melengkapi fastText/SC-WEAT sebelumnya, yang memakai rata-rata vektor token. IndoBERT dipakai sebagai masked language model dalam konteks kalimat, bukan sebagai sentence-embedding model dan bukan dengan WEAT pada `[CLS]`.

Notebook: `test2_v4_indobert_lpbs_pipeline.ipynb`. Data dibaca langsung dari `data/test_v3_kbji_profesional_analysis.csv`, `data/test_v3_kbji_profesional_all.csv`, dan `data/test_v3_istilah_gender.csv`; file input tidak ditimpa. Unit utama adalah Subgolongan Pokok 21–26, dengan kode tiga digit `group_code` sebagai cluster bootstrap. Kondisi utama `pria–wanita` dan sensitivitas `laki-laki–perempuan` ditetapkan sebelum hasil; pasangan kedua memakai pseudo-log-likelihood karena `laki-laki` menjadi tiga subtoken, sedangkan tiga istilah lainnya satu token.

Pipeline mencakup audit data, tokenisasi, empat template dan empat baseline, scoring LPBS batched dengan `torch.no_grad()`, agregasi median lintas-template, cluster bootstrap 5.000 replikasi, Kruskal–Wallis dan epsilon-squared eksploratif, sensitivitas template, sensitivitas istilah gender, perbandingan baseline fastText, visualisasi langsung, dan QC final. Dependency yang digunakan sudah tersedia pada environment repository: pandas, numpy, scipy, statsmodels, matplotlib, seaborn, torch, transformers, tqdm, dan Jupyter. Jalankan dari root repository dengan `python -m jupyter nbconvert --to notebook --execute --inplace test2_v4_indobert_lpbs_pipeline.ipynb` atau buka notebook secara interaktif setelah model Hugging Face dapat diakses.

Output aktual berada di `outputs/`: `test_v4_lpbs_scores_long.csv`, `test_v4_lpbs_occupation_summary.csv`, `test_v4_lpbs_group_summary.csv`, `test_v4_lpbs_template_stability.csv`, `test_v4_lpbs_lexical_sensitivity.csv`, `test_v4_lpbs_tokenization_audit.csv`, `test_v4_lpbs_statistical_tests.csv`, `test_v4_lpbs_fasttext_comparison.csv`, `test_v4_lpbs_fasttext_group_comparison.csv`, `test_v4_lpbs_quality_control.csv`, dan `test_v4_run_metadata.json`.

Hasil eksekusi aktual: 415 jabatan analisis dan 501 nama pada CSV lengkap. Median LPBS per kelompok adalah 21 = -0,0834, 22 = 0,0711, 23 = -0,1704, 24 = 0,1094, 25 = 0,0419, dan 26 = 0,1053. Interval bootstrap masing-masing adalah [-0,1390; 0,0116], [0,0124; 0,1174], [-0,2502; -0,0804], [0,0733; 0,1721], [0,0229; 0,0573], dan [-0,0570; 0,1780]. Kruskal–Wallis eksploratif menghasilkan H = 125,0193, p = 2,71 × 10⁻²⁵, dan epsilon-squared = 0,2934; hasil ini harus dibaca hati-hati karena struktur hierarkis KBJI.

Stabilitas template tidak seragam: Spearman antartemplate berkisar 0,2404–0,9200, agreement tanda 0,3108–0,8410, dan median agreement tingkat jabatan 0,7500. Sensitivitas istilah menghasilkan Spearman 0,2468, median absolute difference 0,1636, agreement 0,6024, dan 165 sign flip. Perbandingan fastText tersedia untuk 401 jabatan, dengan Spearman -0,0890, agreement 0,4788, dan 209 sign flip. Angka tersebut menunjukkan sensitivitas pengukuran terhadap konteks, istilah, dan model; LPBS dan SC-WEAT tidak dibandingkan berdasarkan magnitudo.

Status eksekusi: selesai; seluruh 18 cell kode berhasil dijalankan, semua pemeriksaan pada `test_v4_lpbs_quality_control.csv` berstatus True, dan CSV input tetap memiliki SHA-256 yang sama sebelum dan sesudah. Metadata aktual mencatat revision model `c2cd0b51ddce6580eb35263b39b0a1e5fb0a39e2` dan durasi 163,02 detik pada CPU. Keterbatasan utama adalah satu model IndoBERT, template buatan, lexical set operasional yang belum tervalidasi final, dependensi hierarki jabatan, serta batas antara asosiasi intrinsic dan bias downstream. Referensi tambahan: Wilie et al. (2020) IndoNLU, Kurita et al. (2019) LPBS, Husse dan Spitz (2022), serta Goldfarb-Tarrant et al. (2021).
