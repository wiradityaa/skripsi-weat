# Metodologi implementasi aktif

Acuan penelitian adalah [`acuan_proposal.md`](acuan_proposal.md). Semua perhitungan aktif berada di [`notebooks/analisis_scweat_kbji.ipynb`](../notebooks/analisis_scweat_kbji.ipynb); pipeline utama tidak mengimpor kode arsip atau membaca embedding, cosine, skor, p-value, maupun ringkasan eksperimen historis. Diagnostik penjelas tambahan membaca hasil run utama lengkap yang terverifikasi. Bagian ini memisahkan metode yang ditetapkan acuan, keputusan teknis pelaksanaannya, dan diagnostik tambahan yang diotorisasi 18 Oktober 2026.

## Identitas, sumber, dan seleksi

CSV sumber adalah [`data/kbji_profesional.csv`](../data/kbji_profesional.csv), dengan 501 baris dan nama asli pada `occupation_name` yang disalin ke `occupation_name_raw`. Jejak sumber menunjuk [`data/kbji_2014.pdf`](../data/kbji_2014.pdf), halaman PDF 161–327. Nama, kode, dan seleksi dibandingkan dengan kolom mentah [`data/validasi/kbji_profesional_sumber.csv`](../data/validasi/kbji_profesional_sumber.csv); teks preprocessing lama tidak digunakan sebagai representasi target aktif.

CSV dibaca dengan kolom identitas sebagai string sejak awal. `occupation_code` mengikuti bentuk `dddd.dd`, misalnya `2111.10` dan `2162.00`; konversi melalui float dilarang. Nama asli dan halaman sumber dipertahankan. Nama sama dengan kode berbeda tidak digabungkan.

Boolean dibaca secara eksplisit dari nilai `True`/`False`, dengan penolakan nilai lain atau nilai hilang bila kolom wajib. Keputusan `include_analysis` diperiksa bersama kategori residual, kebocoran atribut, kelengkapan hierarki, validitas kode, dan nama tidak kosong. Kebocoran atribut diperiksa melalui kecocokan token utuh setelah normalisasi, sehingga substring tidak menghasilkan pengecualian keliru.

Kategori residual diidentifikasi melalui akhiran kode `.99` atau nama yang cocok dengan `lainnya|ybdi|ytdl` setelah normalisasi aktif. Kode `2353.00` dan `2359.00` merupakan residual meskipun bukan `.99`; aturan berbasis kode saja tidak cukup. Kebocoran token atribut menghasilkan pengecualian `2635.09`. Keputusan dihitung ulang lalu dibandingkan dengan flag sumber.

Jejak seleksi pada input dibandingkan dengan acuan 501 − 85 − 1 = 415 dan distribusi 21–26 sebanyak 138, 74, 42, 29, 26, dan 106. Pemeriksaan ini memverifikasi data, tidak membentuk atau menghapus baris agar jumlah cocok. Pengukuran utama memerlukan 415 kode unik dan enam kelompok sesuai acuan. `audit_preparasi.csv` menyimpan keputusan dan alasan serta perubahan teks pada seluruh baris sumber yang tersedia.

## Normalisasi dan tokenisasi

Ketentuan acuan adalah menghapus tanda petik yang tidak diperlukan, merapikan karakter dan spasi, menyeragamkan kapitalisasi, memisahkan token berdasarkan spasi, dan mempertahankan tanda hubung. Urutan operasional aktif adalah:

1. Simpan nama sumber tanpa perubahan sebagai nama asli.
2. Normalisasi Unicode **NFKC**.
3. Seragamkan kapitalisasi menjadi huruf kecil.
4. Hapus tanda petik lurus/keriting, tunggal/ganda: `"`, `'`, `“`, `”`, `„`, `‟`, `‘`, `’`.
5. Ganti karakter tidak tercetak kategori Unicode `Cc` dan `Cf` dengan spasi.
6. Ganti pembatas kurung `(` dan `)`, slash `/`, koma `,`, dan titik `.` dengan spasi; pertahankan semua kata. Tanda hubung dan punctuation lain tetap dipertahankan.
7. Rapatkan seluruh whitespace menjadi satu spasi, trim, lalu tokenisasi berdasarkan spasi.

Pemilihan NFKC, penghapusan petik, karakter tidak tercetak, serta daftar delimiter adalah **keputusan teknis**. `Ahli Fisika (Umum)` menjadi `ahli fisika umum`; `Pertamanan/Lansekap` menjadi `pertamanan lansekap`; `laki-laki` tetap satu token. Titik hanya diganti dalam **teks jabatan**, bukan dalam kode identitas. Tidak digunakan stemming, lemmatization, stopword removal, terjemahan, sinonim otomatis, atau pembobotan kata inti.

Kebijakan punctuation lengkap dicatat pada metadata run. Tabel audit perubahan menunjukkan nama sebelum/sesudah, token, jumlah token, dan jenis perubahan. Audit preparasi aktual dengan aturan ini menghasilkan 42 jabatan satu token dan 373 multikata, sesuai acuan; notebook menghitung ulang pemeriksaan ini saat dijalankan.

## Model, representasi target, dan cosine

Model tunggal adalah biner pretrained fastText Bahasa Indonesia `cc.id.300.bin` dengan 300 dimensi, tanpa pelatihan ulang. Sumber resmi: [fastText — word vectors Common Crawl](https://fasttext.cc/docs/en/crawl-vectors.html). Model berbasis Common Crawl dan Wikipedia dan menggunakan CBOW berbobot posisi. Path model relatif terhadap root repository.

Untuk token `t`, API `get_word_vector(t)` menghasilkan vektor native yang dikonversi ke `float64` sebelum agregasi. Pada target dengan `k` token:

\[
v_w=\frac{1}{k}\sum_{i=1}^{k}v_{t_i},\qquad \hat v_w=\frac{v_w}{\lVert v_w\rVert_2}.
\]

Vektor token **tidak dinormalisasi sebelum rata-rata**. Untuk satu token rumus tersebut setara dengan mengambil vektor token langsung. Vektor atribut juga dinormalisasi L2. Tahap ini mengimplementasikan normalisasi pada Persamaan (2.2)–(2.3) dalam acuan.

Semua vektor harus berdimensi 300, finite, dan mempunyai norma bukan nol. Norma vektor satuan diperiksa ulang. Token OOV tetap digunakan karena model biner fastText dapat membentuk vektor melalui subkata. Status vocabulary, norma, dan jumlah token OOV disimpan; OOV tidak otomatis berarti vektor salah. Jika ada vektor atau penyebut tidak valid, identitasnya ditampilkan dan eksekusi dihentikan sampai penyebabnya ditangani.

Cosine pada Persamaan (2.4) dihitung sebagai hasil kali titik vektor satuan:

\[
c(w,a)=\hat v_w^\top\hat v_a.
\]

Hasil diverifikasi terhadap bentuk umum cosine pada beberapa jabatan aktual dan diperiksa berada di `[-1,1]` dalam toleransi numerik. Koefisien rata-rata yang sama tidak menjamin pengaruh geometris sama: panjang dan arah vektor mentah dapat berbeda. Representasi rata-rata tidak menangkap urutan kata atau konteks penggunaan.

## RM 1: SC-WEAT individual

Kelompok `M` dan `F` masing-masing berisi delapan istilah sesuai urutan kanonis pada acuan. Untuk satu jabatan:

\[
\mu_M=\frac18\sum_{m\in M}c(w,m),\qquad
\mu_F=\frac18\sum_{f\in F}c(w,f),\qquad
s(w)=\mu_M-\mu_F.
\]

\[
\sigma_w=\sqrt{\frac1{16}\sum_{a\in M\cup F}(c(w,a)-\bar c_w)^2},\qquad
d(w)=\frac{s(w)}{\sigma_w}.
\]

SC-WEAT pada Persamaan (2.8) menggunakan simpangan baku **populasi** (`ddof=0`) dari 16 cosine untuk jabatan tersebut. Penyebut tidak dihitung dari 415 skor atau dari simpangan baku dua kelompok yang dipool. Penyebut nol menghasilkan skor tidak terdefinisi dan tidak boleh diisi nol.

Seluruh cosine, rata-rata, penyebut, dan skor dihitung serta diekspor penuh sebagai `float64`. Casting mengurangi pembulatan pada perhitungan lanjutan tetapi tidak menambah presisi informasi asli model. Pembulatan hanya untuk tampilan.

## Permutasi eksak dua sisi

Statistik pada Persamaan (2.9) adalah:

\[
T_{obs}(w)=\sum_{m\in M}c(w,m)-\sum_{f\in F}c(w,f)=8s(w).
\]

Enumerator membentuk seluruh pilihan delapan dari 16 indeks atribut dan komplemennya. Terdapat `C(16,8)=12.870` pembagian **berlabel**; pengamatan dan komplemen masuk enumerasi. Model, target, dan cosine tetap; seluruh label atribut dipertukarkan bebas. Pasangan gender tidak membatasi permutasi.

Untuk setiap pembagian, hitung statistik jumlah dengan operasi yang konsisten dengan pengamatan. Persamaan (2.10) diimplementasikan sebagai:

\[
p_{raw}(w)=\frac{\#\{b:|T_b(w)|\ge |T_{obs}(w)|\}}{12870}.
\]

Kejadian sama ekstrem dihitung. Enumerasi lengkap tidak memakai Monte Carlo, uji satu sisi, atau koreksi tambah satu. Keputusan teknis toleransi adalah `atol=1e-12` dan `rtol=1e-12`: statistik yang lebih ekstrem atau sama dalam toleransi termasuk hitungan. Toleransi tidak diubah menurut hasil signifikansi. Statistik komplemen harus berlawanan tanda dan mempunyai nilai absolut sama dalam toleransi.

P-value harus pada `(0,1]`, mencakup pengamatan, dan sama dengan jumlah kejadian ekstrem dibagi 12.870. Implementasi referensi sederhana menghitung ulang beberapa jabatan aktual secara terpisah dari versi vektorisasi.

## BH-FDR global dan arah skor

Benjamini–Hochberg pada Persamaan (2.11) diterapkan **sekali** pada 415 `p_raw`, `q=0.05`. Adjusted p-value dan keputusan `reject_fdr` disimpan. Grafik BH membandingkan p-value terurut dengan ambang `i/415 × 0.05`; keputusan kelompok selalu mengambil hasil koreksi global.

Status `p_raw <= 0.05` dapat disimpan sebagai informasi tambahan. Keputusan utama mengikuti BH global. Besar effect size dan signifikansi merupakan dua hasil yang berbeda; hasil tidak signifikan tidak membuktikan tidak ada asosiasi.

Arah ditentukan dari skor belum dibulatkan dan hanya memakai label `positif`, `negatif`, atau `nol`. Konfigurasi aktif `zero_atol=0.0`: skor > 0 positif, < 0 negatif, tepat 0 nol. Toleransi kesetaraan `atol=rtol=1e-12` tidak digunakan untuk membuat ambang kategori arah. Positif berarti relatif ke `M`; negatif relatif ke `F`. Nol bukan bukti netralitas sosial.

## RM 2: perbandingan deskriptif enam kelompok

Urutan kelompok adalah kode 21–26. Statistik dihitung dari setiap skor individual dalam kelompok: jumlah, mean, median, simpangan baku `ddof=0`, minimum, maksimum, jumlah/proporsi setiap arah, serta jumlah/proporsi `reject_fdr` global. Proporsi memakai jumlah jabatan dalam kelompok sebagai denominator.

Simpangan baku kelompok menggambarkan penyebaran **antarjabatan**, sedangkan penyebut SC-WEAT menggambarkan penyebaran **16 cosine satu jabatan**. Keduanya tidak dapat dipertukarkan. Jumlah anggota kelompok harus kembali ke 415; proporsi positif + negatif + nol harus satu dalam tiap kelompok.

Interpretasi menyandingkan pemusatan, penyebaran, rentang, proporsi, dan signifikansi individual sambil memperhatikan ukuran kelompok. Tidak dijalankan ANOVA, Kruskal–Wallis, uji pasangan, atau p-value perbandingan kelompok. Tidak ada klaim beda signifikan antarsubgolongan.

## RM 3: leave-one-pair-out deskriptif

Pada kondisi `j`, satu istilah laki-laki dan pasangan perempuannya dikeluarkan. Model, vektor target, preprocessing, dan cosine berasal dari eksekusi baru yang sama. Kedua mean dihitung ulang dari tujuh istilah, dan penyebut dihitung ulang dari 14 cosine dengan `ddof=0`.

\[
\Delta_{w,j}=d_{w,j}^{LOPO}-d_w,\qquad
|\Delta_{w,j}|=\text{perubahan absolut}.
\]

Kondisi dipasangkan dengan baseline melalui `occupation_code`. Tidak dihitung p-value, BH-FDR, atau status signifikansi LOPO. Detail berisi 415 × 8 = 3.320 kombinasi unik.

Arah dipertahankan bila label baseline dan LOPO sama. Pembalikan arah hanya transisi positif→negatif atau negatif→positif. Transisi yang melibatkan nol dicatat terpisah. Ranking memakai skor **bertanda** menurun, skor terbesar peringkat satu, dan rata-rata untuk ties; perubahan peringkat adalah peringkat LOPO dikurangi baseline.

Ringkasan jabatan menghitung jumlah dan proporsi dari delapan kondisi yang mempertahankan arah, jumlah pembalikan, minimum/maksimum skor LOPO, perubahan absolut maksimum, dan seluruh pasangan bila nilai maksimum terikat menurut `atol=rtol=1e-12`. Ringkasan pasangan berisi mean/median/maksimum perubahan absolut, jumlah/proporsi pembalikan, transisi nol, dan Spearman pada seluruh 415 jabatan.

Spearman pada Persamaan (2.12) digunakan secara deskriptif berdasarkan peringkat dengan average ties, tanpa melaporkan p-value. Skor konstan menyebabkan korelasi tidak terdefinisi dan dilaporkan demikian. Tidak dibuat kategori stabil/tidak stabil atau ambang sensitivitas baru. Korelasi tinggi tetap dapat hadir bersama pembalikan arah individual.

## Validasi, ekspor, dan provenance

Notebook menghitung ulang hasil setelah baris diacak dan setelah atribut beserta label/mapping pasangan diacak. Skor, p-value, FDR, dan sensitivitas dicocokkan berdasarkan kode, bukan posisi atau skor terurut. Pertukaran kelompok `M`/`F` harus membalik skor dan mempertahankan p-value dua sisi. LOPO diperiksa ulang untuk pasangan yang benar, jumlah 14 atribut, delta, dan ranking.

Ekspor CSV memakai UTF-8 tanpa indeks. Round-trip impor membaca identitas sebagai string dan memastikan kode dengan nol sesudah titik tidak berubah. Gambar disimpan sebagai PNG minimal 300 dpi serta SVG ketika sesuai; warna arah konsisten (`positif` `#D55E00`, `negatif` `#7B61A8`, `nol` `#7A7A7A`). Jitter memakai seed 42 hanya untuk tampilan. Heatmap delta divergen dan simetris terhadap nol; warnanya menunjukkan perubahan skor.

`manifest_run.json` menyimpan waktu, parameter, SHA-256 streaming input/lexical/model, identitas sumber, ukuran/dimensi model, versi Python/paket, Git/working tree, daftar output, durasi, dan status. `run_id` berdasarkan waktu menghasilkan folder baru; output lama tidak ditimpa. Sel terakhir notebook menampilkan tabel quality control dan assertion yang dihitung dari eksekusi aktual. Kegagalan dilaporkan dengan identitas, bukan diganti `PASS`.

Schema lengkap dan cara membaca ulang tersedia di [`kamus_data.md`](kamus_data.md); status dan keputusan aktual di [`catatan_pengerjaan.md`](catatan_pengerjaan.md).

## Diagnostik penjelas tambahan D1–D5

Diagnostik ini mengikuti prompt lanjutan `PROMPT_VSCODE_DIAGNOSTIK_SCWEAT.md`. Tujuannya menjelaskan hasil RM 1–RM 3 kepada pembaca dan penguji, tanpa mengubah metode utama atau menambah pengujian inferensial. Seluruh input perhitungan berupa sembilan CSV `tables/` dari run utama lengkap; model dan embedding tidak perlu dimuat ulang. Identitas jabatan dan mapping pasangan dicocokkan melalui `occupation_code` dan `pair_id`, bukan posisi baris. Konfigurasi numerik, urutan kelompok, dan warna arah dibaca dari manifest sumber; urutan pasangan kanonis divalidasi terhadap lexical set tersimpan.

### D1 — selisih cosine per pasangan

Untuk tiap jabatan dan pasangan, hitung `g = cosine_laki_laki − cosine_perempuan` dari cosine tersimpan dan mapping lexical kanonis. Hasil 3.320 kombinasi dibandingkan dengan tabel kontribusi sumber; mean delapan `g` untuk jabatan yang sama harus cocok dengan `association_difference` baseline.

Ringkasan delapan pasangan menghitung n, mean, median, SD populasi `ddof=0`, minimum, maksimum, serta proporsi selisih positif/negatif/nol dari nilai penuh. Boxplot dan dot plot mean–median menunjukkan perbedaan arah antarpasangan. `g` disebut **selisih cosine per pasangan**, karena merupakan komponen pembilang, bukan effect size SC-WEAT pasangan tunggal. Perbedaan ini tidak menjelaskan frekuensi kata, usia, register, kekerabatan, atau sebab semantik tanpa analisis pendukung.

### D2 — besar skor awal dan pembalikan arah

Baseline digabungkan dengan ringkasan sensitivitas berdasarkan kode. Ukuran yang dibaca bersama adalah `abs(effect_size_baseline)`, jumlah pembalikan dari delapan kondisi, indikator pernah membalik, dan perubahan absolut maksimum. Dua kelompok faktual adalah **pernah membalik arah** dan **tidak pernah membalik arah**; kelompok dibentuk berdasarkan kejadian yang benar-benar teramati, tanpa cutoff besar skor.

Ringkasan dua kelompok menyajikan n serta mean, median, dan rentang besar skor awal/perubahan maksimum. Distribusi jumlah pembalikan 0–8 memakai jumlah **jabatan unik**; jumlah **kejadian jabatan–pasangan** dihitung terpisah. Scatter menggunakan seed tetap untuk jitter tampilan, tanpa mengubah nilai analisis. Hubungan antara skor awal kecil dan pembalikan dilaporkan deskriptif; tidak dijalankan uji signifikansi, ambang netralitas, atau kategori valid/tidak valid.

### D3 — pembilang, penyebut, dan skor

Dari 16 cosine baseline, hitung ulang mean laki-laki, mean perempuan, selisih `D`, SD, dan `D / SD`. Setiap kondisi LOPO mengeluarkan dua istilah pasangan yang benar, lalu menghitung ulang kedua mean dari tujuh istilah serta SD 14 cosine dengan `ddof=0`. Komponen baseline dan seluruh 3.320 skor LOPO diperiksa terhadap hasil sumber dalam toleransi numerik yang sama.

Perubahan disimpan sebagai LOPO dikurangi baseline untuk `D`, SD, dan effect size. Identitas aljabar `D_LOPO = (8 × D_baseline − g_pasangan_dikeluarkan) / 7` diperiksa untuk seluruh kondisi; ini penjelas perhitungan tanpa nomor persamaan proposal baru. Selama SD positif, **arah effect size ditentukan tanda `D`**: SD mengubah besar skor, tetapi tidak dapat membalik tandanya. Perubahan pembilang dan penyebut dibaca bersama, tanpa persentase pembagian pengaruh kausal atau persentase perubahan skor sebagai indikator utama dekat nol.

Kasus ilustratif yang ditetapkan prompt adalah `2654.03 Sutradara Film` dan `2622.02 Juru Katalog`. Semua delapan kondisi keduanya tersedia, dengan perhatian pada pengeluaran `jejaka–gadis` dan `pemuda–pemudi`. Pemilihan kasus didasarkan pada hasil sebelumnya dan tidak mewakili seluruh jabatan; tabel seluruh jabatan tetap menjadi dasar penelusuran.

### D4 — sensitivitas pada enam kelompok

Detail direkap menjadi 48 kombinasi unik Subgolongan Pokok 21–26 × delapan pasangan. Tiap kombinasi memuat jumlah anggota, mean/median baseline dan LOPO, mean delta, mean perubahan absolut, jumlah pembalikan, dan proporsinya. Denominator pembalikan adalah jumlah jabatan **subgolongan tersebut**, bukan 415 atau jumlah kejadian keseluruhan.

Heatmap mean delta memakai skala divergen simetris terhadap nol; warna menunjukkan arah perubahan skor. Heatmap proporsi pembalikan memakai skala berurutan. Perbandingan lintas kelompok tetap deskriptif, tanpa BH per kelompok atau uji LOPO/beda antarkelompok.

### D5 — rekonstruksi distribusi permutasi baseline

Tiga contoh dipilih berdasarkan besar skor baseline: tertinggi, terdekat dengan median besar skor, dan terendah. Ties dipecahkan menurut kode, dan contoh harus berbeda. Seluruh 12.870 statistik jumlah direkonstruksi dari 16 cosine masing-masing contoh; tidak diperlukan model baru.

Histogram menyertakan statistik pengamatan, batas dua sisi `±abs(T_observed)`, dan kejadian ekstrem dengan aturan ties/toleransi sumber. Jumlah ekstrem dibagi 12.870 harus cocok dengan p-value mentah sumber. Adjusted p-value diambil dari keluarga BH global 415 pengujian yang sudah ada, bukan dihitung ulang khusus tiga contoh. D5 merupakan visualisasi ulang pengujian utama, bukan pengujian tambahan; p-value bukan probabilitas hipotesis nol benar.

### Lingkup eksekusi dan provenance diagnostik

Bagian **11. Diagnostik penjelas hasil** berada setelah 45 cell utama pada notebook yang sama, dengan 23 cell tambahan termasuk 13 cell kode, semuanya bertag `diagnostics`. Bootstrap mengimpor paket dan memuat CSV secara eksplisit dari pointer run utama yang manifestnya `complete` serta QC lulus. Parser float `round_trip` dan kode string mempertahankan presisi/identitas sumber. Sembilan checksum tabel dibandingkan dengan daftar output manifest sebelum perhitungan; pointer, manifest, dan sembilan tabel diperiksa kembali pada akhir.

Eksekusi dari kernel baru tidak mengandalkan variabel bagian utama. Output 45 cell utama dipertahankan dari run sebelumnya. Bila seluruh notebook dijalankan, bagian utama selesai lebih dahulu dan diagnostik membaca hasil lengkap yang baru dibentuk. Manifest diagnostik mencatat fingerprint source seluruh notebook, source bagian utama, dan fingerprint notebook parent secara terpisah; output cell tidak masuk fingerprint source. Snapshot `audit/notebook_source.json` menyimpan pasangan tipe/source seluruh 68 cell dengan SHA-256 yang sama dengan fingerprint notebook pada manifest.

Run diagnostik memiliki direktori terpisah `outputs/diagnostik/<diagnostic_run_id>/{tables,figures,audit}`. `manifest_diagnostik.json` mencatat parent run, path/hash sumber, source notebook, konfigurasi, versi, waktu, status, dan `execution_scope=diagnostics_only_from_saved_csv`; `model_loaded=false`. Pointer diagnostik adalah `outputs/diagnostik/LATEST_DIAGNOSTIC_RUN.txt`, terpisah dari `LATEST_RUN.txt` utama yang tidak ditimpa. CSV berformat UTF-8 tanpa indeks dan tanpa pembulatan analisis; gambar PNG minimal 300 dpi dan SVG disimpan beserta caption aktual. Bagian diagnostik berakhir dengan pemeriksaan konsistensi dan tabel QC yang dihitung dari run tersebut.

Hasil tetap merupakan asosiasi intrinsik dalam fastText dan pilihan atribut. Tidak signifikan tidak membuktikan semua asosiasi tidak ada. Pembalikan arah tidak otomatis membuat hasil tidak valid, dan pasangan dengan perubahan besar tidak otomatis salah atau perlu dihapus. Diagnostik tambahan ini tidak diberi nomor persamaan proposal baru.
