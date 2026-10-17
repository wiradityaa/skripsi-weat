# Kamus data dan output aktif

Satu run baru tersimpan di `outputs/analisis_utama/<run_id>/`. Tabel di `tables/` berisi hasil penuh; `figures/` memuat visualisasi; `audit/` memuat jejak validasi dan metadata. Schema halaman ini diselaraskan dengan keluaran notebook aktif, bukan tabel eksperimen historis.

## Konvensi pembacaan

- CSV berformat UTF-8, header pada baris pertama, tanpa indeks DataFrame tambahan. Angka tidak dibulatkan sebelum ekspor.
- Kode jabatan dan hierarki adalah **string**. `2111.10` dan `2162.00` harus tetap demikian. Membaca sebagai float dapat merusak identitas dan tidak dapat diperbaiki melalui tebakan.
- Boolean diekspor sebagai `True`/`False`; parse secara eksplisit dan tolak nilai lain ketika membaca ulang. String `"False"` tidak boleh langsung dikonversi dengan `bool()` karena menghasilkan `True`.
- `occupation_code` adalah kunci jabatan. `pair_id` adalah kunci pasangan. Join memakai kunci, bukan posisi baris atau nama jabatan.
- Nilai kosong hanya diperbolehkan pada metadata opsional atau statistik yang memang tidak terdefinisi dan dijelaskan, bukan sebagai pengganti skor yang gagal dihitung.
- Tampilan skor boleh dibulatkan; nilai analisis dan ekspor penuh dipertahankan. Semua cosine dan statistik lanjutan memakai `float64`.

Contoh pembacaan ulang identitas:

```python
hasil = pd.read_csv(
    run_dir / "tables" / "hasil_scweat.csv",
    dtype={"occupation_code": "string", "submajor_group_code": "string"},
    encoding="utf-8",
)
```

Kolom berupa daftar (`tokens`, `oov_tokens`, `gender_leakage_terms`, `max_change_pairs`) disimpan sebagai **JSON** Unicode. Baca menggunakan `json.loads()`, bukan `eval()`. `audit/schema_output.csv` mencatat kolom aktual, tipe dalam memori, serta encoding daftar pada tabel yang diekspor sebelum QC akhir. Schema `validasi.csv` dan `durasi.csv`, yang dibentuk pada sel terakhir, dijelaskan terpisah di bawah.

## Input yang dipertahankan

| Lokasi | Kolom utama | Fungsi |
| --- | --- | --- |
| `data/kbji_profesional.csv` | `occupation_code`, `occupation_name`, kode/nama hierarki, halaman sumber, flag seleksi | Sumber 501 baris dan nama jabatan asli; `occupation_name` disalin sebagai `occupation_name_raw` |
| `data/validasi/kbji_profesional_sumber.csv` | `occupation_code`, `occupation_name_raw`, hierarki, halaman, flag seleksi | Pemeriksaan silang nama/kode/seleksi mentah; `occupation_text` lama tidak menjadi representasi aktif |
| `data/istilah_gender.csv` | `pair_id`, `male_term`, `female_term`, `lexical_category`, `register`, `validation_status`, `include_baseline`, `notes` | Sumber delapan pasangan dan metadata; ID angka lama dicatat, ID hasil aktif berbasis istilah |
| `data/validasi/istilah_gender_historis.csv` | Pasangan dan flag `include_core`, `include_extended` | Hanya audit perbedaan lexical historis; subset historis tidak menjadi pilihan analisis aktif |
| `data/kbji_2014.pdf` | Dokumen sumber, halaman PDF 161–327 pada jejak Profesional | Sumber klasifikasi dan nama, bukan tabel hasil eksperimen |
| `models/cc.id.300.bin` | Biner fastText pretrained, 300 dimensi | Satu-satunya model pengukuran aktif |

Flag sumber mencakup `is_valid_code`, `is_duplicate_code`, `is_residual`, `contains_gender_attribute`, `hierarchy_complete`, `hierarchy_prefix_match`, `code_name_conflict`, `duplicate_occupation_name`, `name_nonempty`, dan `include_analysis`. Pipeline membaca keputusan ini lalu memverifikasi keputusan dengan data aktual. Duplikasi nama tidak otomatis berarti duplikasi identitas.

## Identitas dan representasi jabatan

Kolom berikut digunakan pada data analisis dan dibawa ke tabel individual ketika diperlukan.

| Kolom | Tipe | Arti / rentang |
| --- | --- | --- |
| `occupation_code` | string | Kode unik bentuk `dddd.dd`; kunci identitas jabatan |
| `occupation_name_raw` | string | Nama asli tanpa perubahan |
| `occupation_text` | string | Nama analisis hasil preprocessing aktif |
| `tokens` | daftar terserialisasi | Token terurut hasil pemisahan whitespace; tanda hubung dipertahankan |
| `token_count` | integer | Jumlah token, minimal satu untuk target final |
| `raw_token_count` | integer | Jumlah token nama asli bila hanya dipisah whitespace, sebelum preprocessing |
| `major_group_code`, `major_group_name` | string | Golongan Pokok `2`, Profesional |
| `submajor_group_code`, `submajor_group_name` | string | Subgolongan Pokok 21–26 dan namanya |
| `group_code`, `group_name` | string | Golongan tiga digit dan nama; jejak hierarki, bukan unit uji RM 2 |
| `subgroup_code`, `subgroup_name` | string | Subgolongan empat digit dan nama |
| `source_document` | string | Nama dokumen sumber |
| `source_page_pdf` | integer | Nomor halaman PDF, penomoran satu-based |
| `source_page_printed` | integer | Nomor halaman tercetak bila tersedia |
| `include_analysis` | boolean | Keputusan seleksi aktif |
| `exclusion_reason` | string / kosong | Alasan pengecualian; kosong pada entri yang masuk analisis |
| `oov_tokens` | JSON array string | Daftar token jabatan yang tidak ada dalam vocabulary model |
| `oov_token_count` | integer | Jumlah token OOV dalam jabatan, termasuk kemunculan berulang bila ada |
| `has_oov` | boolean | `oov_token_count > 0` |

`data_analisis.csv` berisi 415 kode unik. `audit_preparasi.csv` mencakup seluruh baris sumber yang tersedia, sehingga dapat menelusuri 85 residual dan satu kebocoran atribut serta perubahan teks. Nama asli, teks baru, token, keputusan, dan alasan tetap terhubung melalui kode.

## Lexical set dan cosine

`lexicon_gender.csv` mempunyai delapan baris dengan kunci `pair_id`: `laki_laki_perempuan`, `pria_wanita`, `ayah_ibu`, `suami_istri`, `putra_putri`, `pemuda_pemudi`, `cowok_cewek`, `jejaka_gadis`. `pair_id_source` mempertahankan ID sumber `1`–`8` sebagai string. `male_term`, `female_term`, `lexical_category`, `register`, `validation_status`, `notes`, dan `selection_basis` adalah string; `include_baseline` boolean. Keseluruhan harus berisi 16 istilah unik. Metadata kategori/register/validasi/notes berasal dari input, bukan bukti validasi linguistik baru.

`cosine_atribut.csv` berbentuk wide: `occupation_code` dan 16 kolom float64 `cosine_<istilah>`, tepatnya `cosine_laki-laki`, `cosine_pria`, `cosine_ayah`, `cosine_suami`, `cosine_putra`, `cosine_pemuda`, `cosine_cowok`, `cosine_jejaka`, `cosine_perempuan`, `cosine_wanita`, `cosine_ibu`, `cosine_istri`, `cosine_putri`, `cosine_pemudi`, `cosine_cewek`, `cosine_gadis`. Cosine tanpa satuan berada pada `[-1,1]` dalam toleransi. Mapping istilah/pasangan dibaca dari lexical set; matriks numerik berukuran **415 × 16**.

## Hasil utama SC-WEAT

`hasil_scweat.csv` mempunyai satu baris per `occupation_code`, sebanyak 415 baris.

| Kolom statistik | Tipe | Arti / satuan |
| --- | --- | --- |
| `mean_cosine_male` | float64 | Mean delapan cosine atribut laki-laki |
| `mean_cosine_female` | float64 | Mean delapan cosine atribut perempuan |
| `association_difference` | float64 | Mean laki-laki dikurangi mean perempuan |
| `std_cosine_all` | float64 | SD 16 cosine jabatan yang sama dengan `ddof=0`; harus > 0 |
| `effect_size` | float64 | `association_difference / std_cosine_all`, tanpa satuan |
| `T_observed` | float64 | Jumlah cosine laki-laki dikurangi perempuan; setara `8 × association_difference` |
| `extreme_count` | integer | Jumlah pembagian yang setidaknya sama ekstrem, mencakup pengamatan dan komplemen; dibagi 12.870 menjadi p-value |
| `p_raw` | float64 | P-value eksak dua sisi; `(0,1]`, kejadian ekstrem / 12.870 |
| `p_adjusted` | float64 | Adjusted p-value BH untuk keluarga global 415 pengujian; `[0,1]` |
| `reject_fdr` | boolean | Penolakan setelah BH global pada `q=0.05` |
| `reject_raw` | boolean | Informasi tambahan `p_raw <= 0.05`; bukan keputusan utama |
| `direction` | kategori string | Hanya `positif`, `negatif`, atau `nol`, dari skor penuh |
| `rank_baseline` | float64 | Peringkat effect size bertanda menurun, average ties, minimum satu |

Tabel hasil juga membawa `occupation_name_raw`, `occupation_text`, `submajor_group_code`, dan `submajor_group_name`. Positif berarti relatif lebih dekat ke kelompok laki-laki, negatif relatif ke perempuan. `zero_atol=0.0`: label nol hanya skor tepat nol sebelum pembulatan. Besar effect size tidak sama dengan signifikansi; nol tidak berarti netralitas sosial.

## Ringkasan deskriptif subgolongan

`ringkasan_subgolongan.csv` mempunyai enam baris unik berdasarkan `submajor_group_code`, urutan 21–26. Statistik kelompok dihitung dari hasil individual yang bergabung melalui kode.

| Kolom | Tipe | Arti |
| --- | --- | --- |
| `n_occupations` | integer | Denominator semua proporsi kelompok; jumlah keseluruhan 415 |
| `mean_effect_size`, `median_effect_size` | float64 | Mean dan median skor individual |
| `std_effect_size` | float64 | SD antarjabatan kelompok, `ddof=0` |
| `min_effect_size`, `max_effect_size` | float64 | Minimum dan maksimum skor individual |
| `n_positif`, `n_negatif`, `n_nol` | integer | Hitungan label arah individual |
| `proportion_positif`, `proportion_negatif`, `proportion_nol` | float64 `[0,1]` | Hitungan / `n_occupations`; jumlah tiga proporsi satu |
| `n_significant_fdr` | integer | Hitungan `reject_fdr=True` dari pengujian global |
| `proportion_significant_fdr` | float64 `[0,1]` | `n_significant_fdr / n_occupations` |

SD kelompok bukan `std_cosine_all` penyebut SC-WEAT. Tabel ini tidak memuat p-value uji perbedaan antarkelompok.

## Detail dan ringkasan leave-one-pair-out

`lopo_detail.csv` mempunyai **3.320 baris** dengan kunci gabungan `(occupation_code, pair_id)`. Tiap kondisi mengeluarkan dua istilah pasangan, menyisakan tujuh laki-laki dan tujuh perempuan. Tidak ada p-value atau keputusan signifikansi baru pada LOPO.

| Ukuran detail | Tipe | Arti |
| --- | --- | --- |
| `effect_size_baseline` | float64 | Skor dari seluruh 16 atribut |
| `effect_size_lopo` | float64 | Skor ulang dari 14 atribut, penyebut dihitung ulang `ddof=0` |
| `removed_male_term`, `removed_female_term` | string | Dua istilah yang dikeluarkan berdasarkan `pair_id` |
| `n_attributes` | integer | Selalu 14 untuk tiap kondisi aktif |
| `std_cosine_lopo` | float64 > 0 | SD 14 cosine yang tersisa, `ddof=0` |
| `delta` | float64 | Skor LOPO dikurangi baseline |
| `abs_delta` | float64 ≥ 0 | Nilai absolut delta |
| `direction_baseline`, `direction_lopo` | kategori string | Label arah baseline dan kondisi baru |
| `direction_retained` | boolean | Dua label sama, termasuk nol→nol |
| `sign_flip` | boolean | Hanya positif→negatif atau negatif→positif |
| `zero_transition` | boolean | Perubahan arah yang melibatkan nol; nol→nol bukan transisi |
| `direction_transition` | string | Label `arah_baseline→arah_lopo` untuk penelusuran |
| `rank_baseline`, `rank_lopo` | float64 | Ranking bertanda menurun dengan average ties |
| `rank_delta` | float64 | Peringkat LOPO dikurangi baseline; positif berarti posisi menurun |

`lopo_ringkasan_jabatan.csv` mempunyai 415 baris unik, membawa kode/nama/kelompok serta `effect_size` dan `direction` baseline. Kolom ringkasannya:

| Kolom | Tipe | Arti |
| --- | --- | --- |
| `n_conditions` | integer | Delapan kondisi per jabatan |
| `n_direction_retained` | integer 0–8 | Banyak kondisi dengan arah sama |
| `proportion_direction_retained` | float64 `[0,1]` | `n_direction_retained / 8` |
| `n_sign_flips`, `n_zero_transitions` | integer 0–8 | Banyak pembalikan positif–negatif dan transisi nol |
| `min_effect_size_lopo`, `max_effect_size_lopo` | float64 | Rentang delapan skor ulang |
| `max_abs_delta` | float64 ≥ 0 | Perubahan absolut maksimum dari delapan kondisi |
| `max_change_pairs` | JSON array string | Semua ID pasangan terikat maksimum dengan `atol=rtol=1e-12`, urutan kanonis |

Tidak ditambahkan label stabil/tidak stabil.

`lopo_ringkasan_pasangan.csv` mempunyai delapan baris unik sesuai urutan kanonis.

| Kolom | Tipe | Arti |
| --- | --- | --- |
| `pair_id`, `pair_label` | string | Identitas dan label pasangan yang dikeluarkan |
| `n_occupations` | integer | 415 jabatan pada tiap kondisi |
| `mean_abs_delta`, `median_abs_delta`, `max_abs_delta` | float64 ≥ 0 | Mean, median, maksimum perubahan absolut |
| `n_sign_flips`, `proportion_sign_flips` | integer / float64 `[0,1]` | Jumlah pembalikan dan proporsi terhadap 415 |
| `n_zero_transitions` | integer | Jumlah perubahan arah melibatkan nol |
| `spearman_rho` | float64 `[-1,1]` / kosong | Korelasi ranking baseline–LOPO pada seluruh 415 jabatan |
| `spearman_note` | string | Penggunaan deskriptif tanpa p-value, atau penjelasan skor konstan |

`kontribusi_pasangan_per_jabatan.csv` mempunyai 415 baris, kode/nama/kelompok, dan delapan kolom bernama `pair_id` kanonis. Nilainya adalah cosine laki-laki dikurangi cosine perempuan untuk pasangan tersebut. Rata-rata delapan kontribusi sama dengan `association_difference`; audit seluruh pasangan tidak menghapus pasangan otomatis.

## Audit vektor, validasi, dan manifest

Audit vektor dipisahkan agar jenis unit jelas:

| Berkas | Kolom dan tipe | Arti |
| --- | --- | --- |
| `audit_vektor.csv` | Identitas/nama/teks/kelompok jabatan; `token_count`, `oov_tokens` JSON, `oov_token_count`, `has_oov`; `dimension` integer, `raw_norm`, `unit_norm` float64, `finite` boolean | 415 target; norma sebelum L2 dan setelahnya, dimensi 300, validitas vektor |
| `audit_token.csv` | `token` string, `in_vocabulary` boolean, `dimension` integer, `raw_norm` float64, `finite`, `is_gender_attribute` boolean | Seluruh token unik target/atribut, status vocabulary dan norma mentah |
| `audit_atribut.csv` | `attribute`, `gender`, `pair_id` string; `in_vocabulary`, `finite` boolean; `raw_norm`, `unit_norm` float64 | 16 atribut; `gender` teknis `male`/`female` menunjukkan kelompok input, bukan label arah skor |

OOV merupakan status vocabulary dan tidak otomatis merupakan kesalahan fastText.

`audit_preparasi.csv` membawa `occupation_code`, `occupation_name_raw`, `occupation_text`, `tokens`, `raw_token_count`, `token_count`, `submajor_group_code`, `source_document`, `source_page_pdf`, `source_page_printed`, `exclusion_reason`, dan `gender_leakage_terms` JSON. Kolom boolean aktif adalah `text_changed`, `token_count_changed`, `include_analysis_source`, `include_analysis`, `residual_recomputed`, dan `gender_leakage_recomputed`. `include_analysis_source` merupakan keputusan input; `include_analysis` keputusan yang dihitung ulang dan harus cocok. `audit_perubahan_teks.csv` memakai schema sama, difilter ketika teks atau jumlah token berubah.

Audit tambahan:

| Berkas | Schema | Fungsi |
| --- | --- | --- |
| `audit_input.csv` | `role`, `path`, `sha256` string; `size_bytes` integer | Identitas setiap berkas input termasuk model dan lexical historis khusus audit |
| `audit_lexicon_historis.csv` | `pair_id`, `pair_id_source`, `male_term`, `female_term`, `pair_id_v5`, `include_core`, `include_extended`, `note` | Mapping pasangan aktif ke lexical v5; flag historis string, tidak digunakan untuk seleksi aktif |
| `audit_token_composition.csv` | `kategori` string; `acuan_proposal`, `aktual`, `selisih` integer | Perbandingan komposisi satu token/multikata; selisih = aktual − acuan |
| `audit_oov_ringkasan.csv` | `population` string, `n_occupations` integer, `mean_effect_size`, `median_effect_size`, `std_effect_size_ddof0` float64, `n_positive`, `n_negative`, `n_zero`, `n_significant_global_fdr` integer, `note` string | Tiga populasi: seluruh, tanpa token OOV, memiliki token OOV; deskriptif, FDR tetap global |
| `validasi_referensi.csv` | `occupation_code` string; `T_reference`, `std_reference_ddof0`, `effect_reference`, `p_reference` float64; `extreme_count_reference` integer | Hasil implementasi referensi skalar pada sampel aktual |
| `validasi_invariansi.csv` | `scenario` string; `n_occupations`, `n_lopo_combinations` integer; `max_effect_error` float64; `status` boolean | Hitung ulang setelah acak baris/atribut dan perbandingan berdasarkan kode |
| `caption_figures.csv` | `name`, `caption`, `png`, `svg` string | Nama/path dan interpretasi gambar aktual |
| `schema_output.csv` | `file`, `column`, `dtype_in_memory`, `csv_list_encoding` string | Nama kolom ekspor sebelum QC akhir; encoding daftar `JSON` atau kosong |
| `durasi.csv` | `stage` string; `seconds` float64 | Durasi tahap, satuan detik |

`validasi.csv` mempunyai `check` string, `observed` nilai aktual heterogen terserialisasi CSV, `expected` nilai/aturan harapan, `status` boolean, `notes` string, dan `status_label` `PASS`/`FAIL`. Status lulus berasal dari pemeriksaan aktual: cakupan, vektor, cosine, `ddof`, enumerator, simetri, referensi p-value, pertukaran label, hitung ulang setelah pengacakan, ringkasan, LOPO, dan round-trip identitas. Kegagalan dicatat dengan identitas terdampak.

`manifest_run.json` merupakan metadata satu run. Field utamanya: `run_id`, `status`, `start_time`, `end_time`, `config`, `python`, `platform`, `dependencies`, `git_commit`, `git_status_at_start`, `notebook_source_sha256`, `notebook_path`, `inputs`, `model`, `timings_seconds`, `counts`, `results_summary`, `quality_control_passed`, `rm_answers`, dan `outputs`. `inputs`/`outputs` memuat path/ukuran/SHA-256; model memuat dimensi/vocabulary/sumber resmi. Fingerprint notebook memakai tipe dan source setiap cell, tanpa output, sehingga penyimpanan hasil eksekusi tidak mengubah identitas kode sumber. Waktu memakai ISO 8601 zona `+07:00`; durasi detik. `jawaban_rm.json` memuat jawaban aktual masing-masing RM. Manifest tidak mengubah hasil historis.

## Relasi hasil dan gambar

Alur relasi adalah sumber → `audit_preparasi.csv` → `data_analisis.csv` → cosine → `hasil_scweat.csv` → ringkasan kelompok dan LOPO. Tabel jabatan berelasi melalui `occupation_code`; tabel pasangan melalui `pair_id`; detail LOPO melalui kedua kunci. Setiap ringkasan harus dapat ditelusuri kembali ke baris individual.

Gambar RM 1 memakai hasil individual; gambar RM 2 memakai enam kelompok dan titik individual; gambar RM 3 memakai detail/ringkasan LOPO. Heatmap subset notebook menampilkan jabatan menurut kriteria perubahan maksimum yang dinyatakan caption; heatmap lengkap tersedia sebagai keluaran terpisah. Warna delta menunjukkan arah **perubahan skor**, bukan langsung arah asosiasi akhir.

## Konvensi diagnostik penjelas D1–D5

Keluaran diagnostik memakai folder terpisah `outputs/diagnostik/<diagnostic_run_id>/`, dengan `parent_run_id` menunjuk run utama lengkap. Sumbernya adalah sembilan CSV `tables/`: `hasil_scweat.csv`, `cosine_atribut.csv`, `lexicon_gender.csv`, `data_analisis.csv`, `ringkasan_subgolongan.csv`, `lopo_detail.csv`, `lopo_ringkasan_jabatan.csv`, `lopo_ringkasan_pasangan.csv`, dan `kontribusi_pasangan_per_jabatan.csv` dari parent yang sama. Hash input dan manifest sumber diperiksa; hasil run utama tidak ditimpa.

Konvensi string, boolean, UTF-8, nilai penuh, dan join berdasarkan kode tetap berlaku. Kunci detail pasangan adalah `(occupation_code, pair_id)`; kunci ringkasan kelompok–pasangan adalah `(submajor_group_code, pair_id)`. Selisih cosine per pasangan adalah komponen pembilang, sementara effect size tetap menggunakan SD gabungan 16/14 cosine. Delta berarti kondisi LOPO dikurangi baseline. Proporsi pembalikan kelompok memakai jumlah anggota kelompok sebagai denominator.

Istilah **pernah membalik arah** berarti setidaknya satu pembalikan positif–negatif dari delapan kondisi, tanpa cutoff besar skor. Jumlah jabatan unik berbeda dari jumlah kejadian jabatan–pasangan. Transisi yang melibatkan nol tetap terpisah, dan nol literal mengikuti konfigurasi sumber. D5 menyimpan rekonstruksi pengujian baseline; adjusted p-value tetap berasal dari BH global parent, tanpa pengujian LOPO atau koreksi khusus contoh.

### Identitas dan tipe yang dipakai bersama

`occupation_code`, `submajor_group_code`, `pair_id`, nama jabatan/kelompok, label, dan istilah merupakan string. `permutation_id` merupakan **integer indeks partisi mulai nol**, berbeda dari kode jabatan string. Hitungan adalah integer; boolean memakai `True`/`False`; seluruh cosine, skor, delta, proporsi, dan komponen numerik lanjutan adalah float64 dalam memori. Cosine berada pada `[-1,1]`, proporsi pada `[0,1]`, SD positif, dan perubahan absolut tidak negatif. Nilai skor/delta tidak dibulatkan sebelum ekspor.

Identitas jabatan lengkap yang dibawa beberapa tabel adalah `occupation_code`, `occupation_name_raw`, `submajor_group_code`, `submajor_group_name`. Identitas pasangan adalah `pair_id` dan `pair_label`. Schema CSV aktual dicatat pada `audit/schema_diagnostik.csv`; tabel di bawah menerangkan arti kolomnya.

### D1 — tabel selisih cosine pasangan

| Berkas / kunci | Kolom | Arti |
| --- | --- | --- |
| `diag_selisih_pasangan_detail.csv`; `(occupation_code, pair_id)` | Identitas jabatan/pasangan, `male_term`, `female_term`, `cosine_male`, `cosine_female`, `cosine_difference`, `difference_direction` | 3.320 baris; `cosine_difference = cosine_male − cosine_female`. Arah hanya positif/negatif/nol dari selisih penuh, bukan arah effect size pasangan tunggal |
| `diag_selisih_pasangan_ringkasan.csv`; `pair_id` | `pair_label`, `n_occupations`, `mean_cosine_difference`, `median_cosine_difference`, `std_cosine_difference`, `min_cosine_difference`, `max_cosine_difference`, `n_positif`, `n_negatif`, `n_nol`, `proportion_positif`, `proportion_negatif`, `proportion_nol` | Delapan ringkasan, SD `ddof=0`; proporsi memakai 415 jabatan per pasangan |
| `diag_pasangan_berlawanan_contoh.csv`; `occupation_code` | `occupation_name_raw`, `submajor_group_code`, `positive_pair_id`, `negative_pair_id`, `positive_pair_difference`, `negative_pair_difference`, `total_absolute_difference` | Pasangan dipilih berdasarkan mean selisih terbesar/terkecil. Simpan **seluruh** jabatan dengan selisih pasangan pertama positif dan kedua negatif; nama berkas `contoh` tidak berarti hanya cuplikan. Jumlah absolut kedua selisih dipakai untuk urutan tampilan, lalu kode sebagai tie-break |

Tabel terakhir tidak memberi kategori validitas. Notebook menampilkan sepuluh baris teratas menurut `total_absolute_difference` menurun, sedangkan CSV mempertahankan seluruh subset.

### D2 — tabel pembalikan dan besar skor awal

| Berkas / kunci | Kolom | Arti |
| --- | --- | --- |
| `diag_skor_awal_pembalikan.csv`; `occupation_code` | Identitas jabatan, `effect_size_baseline`, `abs_effect_size_baseline`, `n_conditions`, `n_sign_flips`, `n_zero_transitions`, `max_abs_delta`, `ever_sign_flip`, `flip_history` | 415 baris. Delapan kondisi; `ever_sign_flip` boolean `n_sign_flips > 0`; label faktual pernah/tidak pernah membalik. Besar awal adalah absolut skor baseline; maksimum perubahan berasal dari delapan LOPO |
| `diag_ringkasan_kejadian_pembalikan.csv`; `ever_sign_flip` | `flip_history`, `n_occupations`, `mean_abs_baseline`, `median_abs_baseline`, `min_abs_baseline`, `max_abs_baseline`, `mean_max_abs_delta`, `median_max_abs_delta`, `min_max_abs_delta`, `max_max_abs_delta`, `n_occupation_pair_flip_events` | Dua kelompok kejadian. Statistik pertama atas besar skor awal; statistik kedua atas maksimum perubahan per jabatan. Jumlah kejadian adalah jumlah `n_sign_flips`, bukan jumlah jabatan unik |
| `diag_distribusi_jumlah_pembalikan.csv`; `n_sign_flips` | `n_occupations`, `n_occupation_pair_flip_events` | Sembilan baris 0–8, termasuk frekuensi nol; kejadian = `n_sign_flips × n_occupations` |

### D3 — tabel komponen baseline dan LOPO

`diag_komponen_baseline.csv` mempunyai 415 baris unik `occupation_code`, identitas jabatan lengkap, `n_attributes=16`, `mean_cosine_male`, `mean_cosine_female`, `D`, `SD`, `effect_size`, dan `direction`. `D` adalah selisih kedua mean, `SD` dari 16 cosine dengan `ddof=0`, `effect_size=D/SD`.

`diag_komponen_scweat_lopo.csv` mempunyai 3.320 baris dengan kunci `(occupation_code, pair_id)`:

| Kolom | Arti |
| --- | --- |
| Identitas jabatan/pasangan, `removed_male_term`, `removed_female_term` | Jejak jabatan dan dua istilah yang dikeluarkan |
| `n_attributes_baseline`, `n_attributes_lopo` | 16 dan 14 |
| `mean_cosine_male_baseline`, `mean_cosine_female_baseline`, `D_baseline`, `SD_baseline`, `effect_size_baseline` | Dua mean, selisih, SD `ddof=0`, dan skor sebelum pengeluaran |
| `mean_cosine_male_lopo`, `mean_cosine_female_lopo`, `D_lopo`, `SD_lopo`, `effect_size_lopo` | Komponen ulang dari tujuh laki-laki dan tujuh perempuan; SD 14 cosine |
| `removed_pair_cosine_difference` | Selisih cosine pasangan yang dikeluarkan, sama dengan `g` pada D1 |
| `delta_D`, `delta_SD`, `delta_effect_size`, `abs_delta_effect_size` | LOPO dikurangi baseline untuk tiap komponen; kolom terakhir absolut perubahan skor |
| `direction_baseline`, `direction_lopo`, `sign_flip`, `zero_transition` | Arah literal dari skor penuh; dua indikator boolean, transisi nol terpisah |

`diag_studi_kasus_komponen.csv` menyimpan **18 baris**: dua jabatan × baseline dan delapan LOPO. Kuncinya `(occupation_code, pair_id)`; `pair_id="baseline"` merupakan penanda kondisi baseline, bukan pasangan atribut kesembilan. Kolom lain: `occupation_name_raw`, `condition`, `n_attributes`, `mean_cosine_male`, `mean_cosine_female`, `D`, `SD`, `effect_size`, `delta_D`, `delta_SD`, `delta_effect_size`, `direction`, `sign_flip`, `is_focus_condition`. Delta baseline nol; `is_focus_condition` boolean menandai pasangan ilustratif yang ditetapkan prompt. Kasus tidak merupakan sampel representatif; komponen seluruh jabatan tersedia pada tabel detail.

### D4 — tabel kelompok–pasangan

`diag_sensitivitas_subgolongan.csv` mempunyai 48 baris unik `(submajor_group_code, pair_id)`, dengan `submajor_group_name`, `pair_label`, `n_occupations`, `mean_effect_size_baseline`, `median_effect_size_baseline`, `mean_effect_size_lopo`, `median_effect_size_lopo`, `mean_delta`, `mean_abs_delta`, `n_sign_flips`, `proportion_sign_flips`, dan `n_zero_transitions`. Mean/median dihitung dari anggota kombinasi aktual; `proportion_sign_flips = n_sign_flips / n_occupations` kelompok tersebut. Tidak ada BH atau p-value kelompok baru.

### D5 — contoh dan distribusi permutasi

| Berkas / kunci | Kolom | Arti |
| --- | --- | --- |
| `diag_permutasi_contoh.csv`; `occupation_code` | `occupation_name_raw`, `effect_size`, `T_observed`, `extreme_count`, `p_raw`, `p_adjusted` | Statistik dari hasil utama untuk tiga kode terpilih |
| Tabel yang sama | `abs_effect_size`, `distance_to_median_abs`, `selection_criterion`, `population_median_abs_effect_size` | Dasar pemilihan maksimum, terdekat median, minimum besar skor; ties menurut kode dan contoh tidak berulang |
| Tabel yang sama | `T_reconstructed`, `n_partitions`, `extreme_count_reconstructed`, `p_raw_reconstructed`, `p_adjusted_global` | Statistik/jumlah ekstrem dihitung ulang, `n_partitions=12870`; p direkonstruksi sebagai jumlah ekstrem/12.870. Adjusted p tetap milik BH global parent |
| `diag_permutasi_distribusi.csv`; `(occupation_code, permutation_id)` | `selection_criterion`, `T_permutation`, `is_extreme` | 38.610 baris = tiga contoh × 12.870 statistik jumlah; `is_extreme` boolean mencakup ties menurut toleransi sumber |
| `audit/diag_partisi_atribut.csv`; `permutation_id` | `first_group_terms`, `second_group_terms` | 12.870 baris mapping indeks ke dua kelompok delapan istilah, disimpan sebagai JSON array string. Relasi ke distribusi melalui `permutation_id` |

### Audit dan manifest diagnostik

| Berkas pada `audit/` | Schema / isi |
| --- | --- |
| `input_diagnostik.csv` | `path`, `size_bytes`, `sha256`, `role`; 11 input: pointer utama, manifest parent, dan sembilan CSV. Path relatif root, ukuran byte, hash string 64 digit heksadesimal |
| `caption_diagnostik.csv` | `name`, `png`, `svg`, `caption`; path gambar relatif folder run dan caption dari angka aktual |
| `schema_diagnostik.csv` | `file`, `column`, `dtype`, `list_encoding`; daftar kolom/tipe dari 15 tabel ekspor sebelum QC akhir. Encoding daftar `JSON`, lainnya kosong |
| `validasi_diagnostik.csv` | `check`, `observed`, `expected`, `status`, `status_label`; boolean hasil pemeriksaan dan label PASS/FAIL |
| `interpretasi_diagnostik.json` | Jawaban deskriptif D1–D5 berdasarkan eksekusi aktual |
| `ringkasan_diagnostik.json` | Nilai ringkasan D1–D5, termasuk kasus dan contoh permutasi |
| `notebook_source.json` | Snapshot JSON pasangan `[cell_type, source]` seluruh 68 cell, tanpa output; SHA-256 cocok dengan `notebook_source_sha256`, sehingga source saat run tetap dapat ditelusuri setelah notebook berubah |
| `manifest_diagnostik.json` | Identitas run/parent, lingkup, input dan output checksum, source notebook, konfigurasi, versi, waktu, status, ukuran hasil, ringkasan, dan QC |

Field manifest utama adalah `diagnostic_run_id`, `parent_run_id`, `status`, `execution_scope`, `model_loaded`, `start_time`, `end_time`, `elapsed_seconds`, `config`, `inputs`, `parent_model`, `parent_manifest_path`, `notebook_path`, `notebook_source_sha256`, `main_cell_source_sha256`, `parent_notebook_source_sha256`, `python`, `platform`, `dependencies`, `git_commit`, `git_status_at_start`, `counts`, `quality_control_passed`, `summary`, `interpretations`, `outputs`, `main_input_hashes_unchanged`, dan `main_latest_pointer_unchanged`. Waktu ISO8601 zona `+07:00`; durasi detik. `parent_model` merupakan provenance model run sumber, bukan klaim pemuatan ulang.

`execution_scope=diagnostics_only_from_saved_csv` menyatakan perhitungan diagnostik memakai hasil tersimpan dan `model_loaded=false`. Semua source cell notebook dicakup fingerprint tanpa output; source bagian utama dan source notebook parent dicatat terpisah. Manifest tidak diikutkan dalam checksum dirinya sendiri. Pointer `outputs/diagnostik/LATEST_DIAGNOSTIC_RUN.txt` terpisah dari pointer run utama.
