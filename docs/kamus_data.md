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
