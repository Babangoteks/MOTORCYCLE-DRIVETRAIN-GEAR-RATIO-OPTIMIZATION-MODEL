# Optimasi Drivetrain Motor Drag Race 300cc / 12.000 RPM
**Perbandingan System A (6-speed manual) vs System B (4-speed kopling sentrifugal, B1 & B2)**

> **Status data - baca dulu.** Kurva dyno, massa, jari-jari ban, koefisien traksi, dan kondisi lintasan **tidak diberikan**. Karena itu:
> - Rasio gigi dari tabel Anda = **MEASURED**. Tiga kurva torsi ternormalisasi (Skenario A/B/C) = **HYPOTHETICAL**.
> - Massa 165 kg, R_tire 0,29 m, η 0,90, μ 1,0, CdA 0,40 m², T_peak 29,8 Nm, dll. = **ASSUMED placeholder** (semua bisa diubah di `core.py -> Params`).
> - Semua ET / kecepatan / percepatan = **SIMULATED**. **Bukan prediksi performa nyata.** Hanya perbandingan *relatif* antar konfigurasi yang bermakna. Tidak ada nilai HP atau torsi aktual yang diklaim (grafik daya memakai skala ternormalisasi).

---

## 0. Ringkasan eksekutif

1. **Faktor penentu bukan jumlah gigi, melainkan rasio total (Rp x Rg x Rf).** Dengan asumsi dasar, rasio total gigi 1 yang optimum ≈ **19-20** (rentang 15-26 jika torsi puncak ±30%). System B asli (20,3-20,9) sudah dekat optimum; System A asli (12,85) terlalu "tinggi".
2. **System A dengan final drive asli (2,933) lebih lambat 0,47-0,53 s (relatif) dari System B** pada 201 m. Gigi 5 dan 6 tidak pernah terpakai (di 402 m hanya mencapai gigi 3-5; kecepatan puncak gigi 6 @12k = 266 km/h, sedangkan terminal speed simulasi hanya ~130-157 km/h). Agar A mendekati optimum, rasio final **teoretis ≈ 4,1-4,6** (perlu dicek kelayakan mekanis, **tidak ada kombinasi gigi yang diklaim layak**).
3. **System B sudah hampir optimum**: mengubah final drive saja ke ≈ 2,3-2,4 hanya menghemat ≤0,04 s (relatif), kecuali B1 pada mesin Skenario A di 402 m (≈0,10 s).
4. **Enam gigi tidak otomatis lebih baik.** Optimum Level 2 untuk 6-speed dan 4-speed **seri** (ET201 8,933 s vs 8,933 s; ET402 14,135 vs 14,146 s), selisih jauh di bawah resolusi model.
5. **Diskrepansi final ratio B (2,500 vs 34/14 = 2,42857)** = beda 2,94% pada rasio total. Dampak ET kecil (≤0,01 s di 201 m, 0,02 s di 402 m) tetapi mengubah batas kecepatan gigi 4 @12.000 rpm: **169,7 km/h (B1) vs 174,7 km/h (B2)**. Harus dikonfirmasi dengan menghitung gigi sprocket fisik.
6. **GA dan PSO sama andalnya** (5 seed: 8,933 ± 0,001 s). Grid exhaustive (2.700 titik) 0,006-0,008 s lebih buruk karena grid kasar. NSGA-II dipakai untuk trade-off, bukan untuk ET minimum.

---

## 1. Data, asumsi, dan validasi

### 1.1 Validasi rasio dari jumlah gigi
| Item | Dinyatakan | Dari gigi | Status |
|---|---|---|---|
| A primary | 1,627 | 83/51 = 1,6275 | konsisten (pembulatan) |
| A overall @6th | 4,938 | 1,6275 x 1,0345 x 2,9333 = 4,9384 | konsisten |
| A gigi 1-6 | 2,692 / 2,059 / 1,650 / 1,409 / 1,222 / 1,034 | 2,692 / 2,059 / 1,650 / 1,409 / 1,222 / 1,034 | konsisten |
| B primary | 3,350 | 67/20 | konsisten |
| B gigi 1-4 | 2,500 / 1,550 / 1,150 / 0,923 | idem | konsisten |
| **B final** | **2,500** | **34/14 = 2,42857** | **TIDAK konsisten (+2,94%)** |

Model memakai rasio **dari jumlah gigi** (bukan nilai bulat). B dijalankan sebagai **B1 (2,500)** dan **B2 (34/14)**; tidak ada yang dibuang atau dikoreksi diam-diam. (Tebakan yang belum terverifikasi: 2,500 bisa saja 35/14; ini *tidak* dipakai.)

### 1.2 Dampak diskrepansi B1 vs B2
| | Rf | Rtotal gigi 1 | Rtotal gigi 4 | Vmax gigi 4 @12k | ET201 | ET402 |
|---|---|---|---|---|---|---|
| B1 | 2,500 | 20,938 | 7,731 | 169,7 km/h | 8,993 | 14,202 |
| B2 | 2,4286 | 20,339 | 7,510 | 174,7 km/h | 8,983 | 14,182 |

B1 dan B2 hanya berbeda posisi pada **kurva optimum yang sama** (primary + gearbox sama). Optimum teoretis Rf* identik (≈2,32 untuk Skenario B). Pada Skenario C di 402 m, B1 selesai di ~11.110 rpm (92,6% dari limiter) vs B2 ~10.780 rpm: **B2 punya headroom limiter lebih besar**.

### 1.3 Asumsi simulasi (placeholder, semua dapat diubah)
massa 165 kg (motor + rider), R_tire 0,29 m, η 0,90 (sensitivitas 0,85/0,95 di bawah), CdA 0,40 m², ρ 1,15, Crr 0,025, μ 1,0, distribusi beban belakang 60%, tinggi CG 0,60 m, wheelbase 1,30 m, inersia roda 0,9 kg·m², inersia mesin 0,025 kg·m², **T_peak 29,8 Nm (dari BMEP 12,5 bar x 300 cc; hanya skala hitung)**, jeda daya per upshift 0,10 s (sama untuk A dan B).

### 1.4 Skenario mesin (HYPOTHETICAL, T_peak ternormalisasi = 1,0)
| Skenario | Zona analisis | RPM torsi puncak | RPM daya puncak (ternormalisasi) |
|---|---|---|---|
| A Low-Mid | 4.000-8.000 | 6.000 | 7.275 |
| B Balanced | 6.000-10.000 | 8.000 | 8.925 |
| C High-RPM | 8.000-12.000 | 10.000 | 10.525 |

12.000 rpm diperlakukan sebagai limiter, bukan titik daya puncak.

---

## 2. Model (ringkas)
Seluruh rumus bagian 4 spesifikasi diimplementasi: Rp, Rg, Rf, Rtotal(i) = Rp·Rg(i)·Rf, f = RPM/60 (mesin, output gearbox, roda; **hanya frekuensi putar - tidak ada analisis getaran torsional/resonansi**), T_wheel = T_eng·Rtotal·η, V = RPM·2π·R·60/(Rtotal·1000), F = T_wheel/R, a = (F - F_drag - F_roll)/m_eff.

- **m_eff** menyertakan inersia roda dan inersia mesin yang direfleksikan (∝ Rtotal²; besar di gigi 1).
- **Traksi tidak dianggap tak terbatas:** a dibatasi oleh (i) traksi ban dengan transfer beban dan (ii) batas wheelie (beban depan = 0). Dengan placeholder, **batas wheelie ≈ 8,5 m/s² mengikat di gigi rendah** (μ 1,2 memberi hasil sama dengan μ 1,0).
- **Launch:** kopling slip pada RPM launch = batas bawah zona skenario, tanpa model panas/ramp kopling. Kopling sentrifugal B **tidak** dianggap membuat gearbox otomatis; perpindahan gigi tetap manual dengan jeda daya yang sama. Karakter engagement sentrifugal nyata **tidak dimodelkan** (data tidak ada).
- **Kebijakan shift:** naik gigi saat percepatan (terbatasi traksi) di gigi berikutnya pada RPM pasca-shift lebih besar daripada gigi sekarang, atau saat limiter 12.000. Jadi **tidak otomatis shift di 12.000**.
- **Powerband utilization** = % waktu lari 201 m dengan mesin di dalam zona skenario. **Traction utilization** = F_ground / (μ·N_belakang).

---

## 3. Hasil

Tabel lengkap (CSV + Markdown) ada di `tables/` dan Lampiran di bawah. Grafik 1-14 + 1 grafik tambahan ada di `plots/`.

### TABLE A - Spesifikasi drivetrain
| | A (6-spd) | B1 (4-spd) | B2 (4-spd) |
|---|---|---|---|
| Primary | 1,6275 | 3,350 | 3,350 |
| Final | 2,933 | 2,500 | 2,4286 |
| Rtotal per gigi | 12,85 / 9,83 / 7,88 / 6,73 / 5,84 / 4,94 | 20,94 / 12,98 / 9,63 / 7,73 | 20,34 / 12,61 / 9,36 / 7,51 |
| Step antar gigi | 1,31 / 1,25 / 1,17 / 1,15 / 1,18 | 1,61 / 1,35 / 1,25 | 1,61 / 1,35 / 1,25 |
| Span 1/top | 2,60 | 2,71 | 2,71 |
| Vmax top @12k | 265,7 km/h | 169,7 km/h | 174,7 km/h |

Kesimpulan terukur dari angka saja (tidak butuh dyno): B jauh lebih "pendek". Rasio total gigi 1 B 63% lebih besar dari A, gigi teratas B 57% lebih besar.

### TABLE B - Kondisi A: low-mid RPM (mesin Skenario A, 4.000-8.000 rpm = 66,7-133,3 Hz)
| | A | B1 | B2 |
|---|---|---|---|
| Torsi roda gigi 1 @6.000 rpm (Nm, skala placeholder) | 345 | 562 | 546 |
| Percepatan puncak (m/s²) | 5,57 (dibatasi mesin) | 8,50 (dibatasi wheelie) | 8,50 |
| Traction util. rata-rata / puncak | 58 / 69 % | 60 / 90 % | 60 / 90 % |
| Powerband util. | 97,0 % | 85,4 % | 91,5 % |
| Rata-rata RPM drop per shift | 17,8 % | 27,8 % | 27,8 % |
| **ET201 (relatif)** | 9,861 | 9,491 | **9,482** |
| ET402 (relatif) | 15,475 | 15,195 | **15,152** |

**Pemenang low-RPM:** B (launch dan torsi roda lebih kuat; ET lebih baik 0,37-0,38 s). A unggul pada **kerapatan rasio dan powerband util.**, tetapi hanya mencapai 5,57 m/s² di gigi 1 sehingga traksi tidak termanfaatkan (≈58% rata-rata). **Gigi 1 A terlalu panjang**, bukan terlalu pendek. Pada B, gigi 1 tidak "terlalu pendek": utilitas traksi puncak 90% (<100%), pembatasnya adalah wheelie, bukan wheelspin (pada μ=0,8 traksi menjadi pembatas, ET B1 naik ke 9,18 s).

### TABLE C - Kondisi B: high-RPM (mesin Skenario C, 8.000-12.000 rpm = 133,3-200 Hz)
| | A | B1 | B2 |
|---|---|---|---|
| Vmax gigi teratas @12k | 265,7 km/h | 169,7 km/h | 174,7 km/h |
| RPM & gigi saat finish 402 m | 11.304 rpm (188 Hz), **gigi 3** | 11.110 rpm (185 Hz), gigi 4 | 10.778 rpm (180 Hz), gigi 4 |
| Powerband util. (201 m) | 97,9 % | 92,3 % | 93,2 % |
| **ET201 (relatif)** | 9,294 | 8,700 | **8,689** |
| **ET402 (relatif)** | 14,246 | **13,654** | **13,653** |
| V402 | 156,9 | 157,1 | 156,9 km/h |

**Pemenang high-RPM:** B (ET 0,59-0,61 s lebih baik). Mesin tidak pernah mencapai gigi 5-6 pada A. A hanya menang pada "kedekatan ke powerband" (97,9% vs 92-93%) yang **bukan** tujuan akhir. Gearing lebih pendek (rasio total lebih besar) memperbaiki akselerasi, sedangkan kecepatan puncak teoretis gigi teratas tidak terpakai pada 201/402 m. B1/B2 mencapai ~90-93% limiter di garis finish 402 m, jadi **limiter hanya tercapai jika lintasan/daya melampaui asumsi**.

### TABLE D - RPM sebelum/sesudah tiap perpindahan gigi
RPM shift ideal = RPM terendah di mana gaya roda gigi berikutnya ≥ gigi sekarang (silang gaya, drag diabaikan).

| Sistem | Shift | Rasio next/cur | RPM drop | Pasca-shift dari 12.000 | Shift ideal (Skenario B) → RPM sesudah | Shift ideal (Skenario C) → sesudah |
|---|---|---|---|---|---|---|
| A | 1→2 | 0,765 | 23,5 % | 9.176 | 10.150 → 7.762 | 11.900 → 9.100 |
| A | 2→3 | 0,801 | 19,9 % | 9.617 | 9.925 → 7.954 | 11.675 → 9.357 |
| A | 3→4 | 0,854 | 14,6 % | 10.248 | 9.650 → 8.241 | 11.375 → 9.714 |
| A | 4→5 | 0,867 | 13,3 % | 10.409 | 9.575 → 8.305 | 11.300 → 9.801 |
| A | 5→6 | 0,846 | 15,4 % | 10.157 | 9.675 → 8.189 | 11.425 → 9.670 |
| B | 1→2 | 0,620 | **38,0 %** | 7.440 | 11.075 → 6.867 | 12.000 → 7.440 |
| B | 2→3 | 0,742 | 25,8 % | 8.903 | 10.275 → 7.623 | 12.000 → 8.903 |
| B | 3→4 | 0,803 | 19,7 % | 9.632 | 9.925 → 7.967 | 11.675 → 9.371 |

(Skenario A: shift ideal ≈ 7.800-9.075 rpm; lengkap di `tables/D_rpm_drop.csv`.) **Shift tidak selalu di 12.000:** di Skenario B optimum ≈ 9.600-11.100 rpm; hanya Skenario C yang mendekati limiter. Pada B, drop 1→2 sebesar 38% membuat RPM mendarat di 6.867 (Skenario B, masih di dalam zona 6.000-10.000) tetapi 7.440 (Skenario C), **di bawah batas bawah zona 8.000**; itulah kelemahan spacing B.

### TABLE E - Performa algoritma (Level 2, mesin Skenario B, objektif ET 201 m)
| Arsitektur | Algoritma | Rasio total optimal | ET201 | ET402 | Evals | Konvergensi |
|---|---|---|---|---|---|---|
| 6-speed | Exhaustive (grid 2.700) | 19,36 / 13,15 / 10,32 / 8,40 / 6,97 / 5,87 | 8,941 | 14,143 | 2.700 | n/a |
| 6-speed | GA | 19,44 / 13,96 / 10,65 / 8,72 / 6,52 / 6,06 | **8,933** | 14,177 | 4.080 | 20 gen |
| 6-speed | PSO | 19,43 / 13,89 / 10,48 / 8,36 / 6,81 / 6,54 | 8,935 | 14,151 | 3.050 | 16 iter |
| 6-speed | NSGA-II (min-ET) | 19,43 / 14,45 / 11,40 / 9,60 / 8,50 / 6,73 | 8,948 | 14,172 | 4.080 | 80 titik Pareto |
| 4-speed | Exhaustive | 19,65 / 13,57 / 10,77 / 8,84 | 8,939 | 14,290 | 2.700 | n/a |
| 4-speed | GA | 19,50 / 13,98 / 10,73 / 8,68 | **8,933** | 14,253 | 4.080 | 9 gen |
| 4-speed | PSO | 19,43 / 14,02 / 10,67 / 8,62 | **8,933** | 14,244 | 3.050 | 13 iter |
| 4-speed | NSGA-II (min-ET) | 19,49 / 13,88 / 10,54 / 8,71 | 8,934 | 14,261 | 4.080 | 80 titik Pareto |

Keandalan 5 seed acak: GA 8,933 ± 0,000-0,001 s; PSO 8,933 ± 0,000-0,001 s (kedua arsitektur). **Validasi Level 1 (1-D):** GA dan PSO menemukan Rf* dalam 0,004 dari sweep exhaustive (A: 4,42; B: 2,32), ET identik sampai 0,001 s.

**Peringatan penting:** hanya hasil kali Rp·Rg·Rf yang memengaruhi fisika, sehingga banyak kombinasi (Rp, Rf, gigi) **secara fisik identik**. Contoh: optimum 6-speed GA Rp=2,23/Rf=3,00 dan PSO Rp=2,08/Rf=4,20 memberi rasio total hampir sama. Gigi yang tak terpakai di 201 m (gigi 5-6) bebas berubah. Selisih ET <0,01 s berada di bawah resolusi model (dt = 5 ms, kebijakan shift sederhana) dan **tidak boleh dibaca sebagai "algoritma X menang"**.

**NSGA-II (Gambar 13):** objektif ET201 dan terminal speed 402 m **tidak saling berlawanan** pada rentang realistis. Keduanya terbaik di sekitar rasio gigi-1 ≈ 19,4 (titik "knee" ≈ titik ET minimum, ET 8,948-8,954). Trade-off nyata muncul pada Obj4 (RPM drop) dan Obj3 (akselerasi dalam powerband). Objektif tidak digabung menjadi skor tunggal; "knee" didefinisikan sebagai jarak min-max ternormalisasi terkecil terhadap titik ideal pada (ET201, V402) dengan bobot sama.

- Obj3 = ∫ a·dt di dalam zona / ET (m/s²). Obj4 = rata-rata % RPM drop + penalti pendaratan di bawah batas zona.

### Level 1 - Hanya final drive (primary + gearbox asli)
Target rasio teoretis (tidak ada daftar sprocket, jadi **tidak ada kombinasi gigi yang diklaim layak**):

| Engine | Dasar | Rf asli | Rf* teoretis (201 m / 402 m) | Hemat ET (201 / 402) |
|---|---|---|---|---|
| Skenario A | A | 2,933 | 4,40 / 4,08 | 0,335 / 0,307 s |
| Skenario B | A | 2,933 | 4,42 / 4,42 | 0,491 / 0,502 s |
| Skenario C | A | 2,933 | 4,58 / 4,58 | 0,694 / 0,717 s |
| Skenario A | B1 / B2 | 2,500 / 2,4286 | 2,32 / 2,28 | 0,022 / 0,102 s (B1); 0,013 / 0,055 s (B2) |
| Skenario B | B1 / B2 | 2,500 / 2,4286 | 2,32 / 2,32 | 0,029 / 0,039 s (B1); 0,017 / 0,016 s (B2) |
| Skenario C | B1 / B2 | 2,500 / 2,4286 | 2,40 / 2,48 | 0,014 / 0,003 s (B1); 0,003 / 0,001 s (B2) |

Kurva ET vs Rf sangat datar di dekat optimum (misalnya B: Rf 2,24-2,48 hanya beda ~0,03 s; grafik 15). Rf* untuk A ≥ 4,3 sudah melebihi kisaran sprocket yang umum; bila dibatasi ≤ 4,3, A masih menghemat ~0,48 s di Skenario B (ET 8,975 vs 9,456), tetapi kelayakan rantai/ground-clearance/ukuran sprocket **harus diperiksa dengan data nyata**.

### Sensitivitas efisiensi drivetrain (η)
ET201 Skenario B: η = 0,85 → A 9,706 / B1 9,150 / B2 9,147; η = 0,90 → 9,461 / 8,993 / 8,983; η = 0,95 → 9,236 / 8,848 / 8,840. Efisiensi menggeser semua sistem bersama-sama (≈ ±0,15-0,25 s per ±0,05); **urutan A lebih lambat dari B tidak berubah.**

### Ketahanan terhadap asumsi placeholder (Skenario B, 201 m)
A selalu lebih lambat dari B1 dengan rasio asli pada semua variasi (selisih 0,13 s pada T_peak x1,3 sampai 0,82 s pada T_peak x0,7; massa 150-180 kg: 0,32-0,62 s; R_tire 0,27-0,31 m: 0,31-0,64 s; waktu shift 0,05-0,20 s: 0,41-0,50 s). Rf* untuk A tetap **> 2,933 pada semua variasi (3,4 - 5,0)**. Rasio total gigi-1 optimum berkisar **15-26** (≈19 pada asumsi dasar). Jadi *arah* kesimpulan kuat, tetapi *nilai* optimum bergantung pada data nyata.

### Optimum Level 2 per skenario mesin (teoretis, kontinu)
| Skenario | Jarak | Arsitektur | Rasio total | ET201 | ET402 |
|---|---|---|---|---|---|
| A | 201 | 6-speed | 19,44 / 13,06 / 9,61 / 7,59 / 5,33 / 5,12 | 9,463 | 15,124 |
| A | 201 | 4-speed | 19,44 / 13,06 / 9,59 / 7,59 | 9,463 | 15,144 |
| B | 201 | 6-speed | 19,43 / 13,96 / 10,72 / 8,66 / 6,05 / 5,25 | 8,933 | 14,223 |
| B | 201 | 4-speed | 19,44 / 14,07 / 10,84 / 8,73 | 8,933 | 14,262 |
| C | 201 | 6-speed | 19,92 / 14,91 / 11,83 / 9,71 / 7,18 / 6,07 | 8,586 | 13,596 |
| C | 201 | 4-speed | 19,92 / 14,81 / 11,71 / 9,66 | 8,586 | 13,915 |
| B | 402 | 6-speed | 19,44 / 13,88 / 10,59 / 8,53 / 7,17 / 5,03 | 8,939 | **14,135** |
| B | 402 | 4-speed | 19,44 / 13,19 / 9,65 / 7,51 | 8,948 | **14,146** |

Gigi 5-6 pada 6-speed hasil optimasi tidak terpakai di 201 m, sehingga nilainya tidak ditentukan oleh fisika model; hanya 4 gigi pertama yang "terukur".

### TABLE F - Rekomendasi
| Konfigurasi | Optimum **TEORETIS** (Level 2) | Yang bisa dicapai dengan **hardware B yang ada** (hanya ganti final; target rasio, bukan jumlah gigi) |
|---|---|---|
| **1. Terbaik low-RPM** (mesin tipe A) | rasio total ≈ 19,4 / 13,1 / 9,6 / 7,6 (ET201 9,463) | Rf ≈ 2,28-2,32 (B2 hampir optimum) |
| **2. Terbaik high-RPM** (mesin tipe C) | rasio total ≈ 19,9 / 14,8 / 11,7 / 9,7 (ET201 8,586) | Rf ≈ 2,40-2,48, **B2 (2,4286) sudah di optimum** (selisih 0,001-0,003 s) |
| **3. Terbaik keseluruhan** (mesin tipe B; dyno belum ada) | rasio total ≈ 19,4 / 14,0 / 10,8 / 8,7 (ET201 8,933) | **System B, Rf ≈ 2,32-2,43**; B1/B2 hanya 0,017-0,039 s dari optimum |

Teoretis = optimum kontinu yang **belum** diverifikasi secara mekanis (modul gear, jarak sumbu, packaging, rantai). Praktis = hanya perubahan final drive pada hardware B, itu pun hanya sebagai target rasio.

---

## 4. Jawaban pertanyaan final (bagian 12)

1. **Primary mana yang lebih cocok?** Model **tidak dapat memilih 1,6275 vs 3,350 berdasarkan akselerasi**, karena yang berpengaruh adalah rasio total. Rp 2,0-3,4 memberi ET yang praktis sama bila Rg dan Rf disesuaikan. Argumen yang bisa dipertanggungjawabkan: dengan primary 3,350, rasio total gigi 1 ≈ 19-21 dicapai dengan **Rf ≈ 2,3-2,5**, sedangkan dengan 1,6275 dibutuhkan **Rf ≈ 4,1-4,6**. Rp tinggi menempatkan rasio target pada Rf yang jauh lebih realistis dan membuat kopling basah berputar lebih lambat (≈3.580 rpm vs ≈7.370 rpm pada mesin 12.000 rpm). Itu pertimbangan **mekanis** yang perlu diverifikasi (beban gigi gearbox naik, rantai lebih ringan), bukan hasil model performa.
2. **Arsitektur terbaik di torsi RPM rendah:** System B (ET201 9,48-9,49 vs 9,86 di Skenario A; torsi roda gigi 1 ≈ 1,6x). A unggul di powerband util. (97% vs 85-92%).
3. **Arsitektur terbaik di daya RPM tinggi:** System B (ET201 8,69-8,70 vs 9,29; ET402 13,65 vs 14,25 di Skenario C). A tidak mencapai gigi tertingginya.
4. **RPM recovery pasca-shift:** **System A lebih baik** (drop 13-24%, mendarat 7.760-8.300 rpm di Skenario B) dibanding B (drop 20-38%, 1→2 turun 38%). Namun recovery yang lebih baik tidak berarti ET lebih baik (pemenang ET tetap B).
5. **Final drive terbaik:** untuk B ≈ **2,3-2,4** (teoretis 2,32); B1 dan B2 sama-sama dalam 0,03 s dari optimum, jadi pilihan 2,500 vs 2,4286 harus ditentukan oleh **headroom limiter di 402 m**, bukan oleh ET (B2 lebih aman). Untuk A ≈ **4,1-4,6** (teoretis).
6. **Apakah 6-speed pasti lebih baik?** **Tidak.** Dengan optimasi penuh keduanya seri (ET201 8,933 = 8,933 s; ET402 14,135 vs 14,146 s). Pada 201 m / 402 m gigi 5-6 jarang atau tidak pernah dipakai, dan setiap upshift tambahan menghabiskan jeda daya 0,10 s (asumsi). Keunggulan 6-speed hanya dalam kerapatan rasio dan fleksibilitas bila daya atau jarak lebih tinggi.
7. **Gigi 1 terlalu pendek atau panjang?** Bergantung pada rasio **total**. A (12,85) **terlalu panjang** (percepatan dibatasi mesin 5,6 m/s², wheelie limit 8,5). B (20,3-20,9) **mendekati optimum, sedikit pendek** (+5-8% terhadap ≈19,4), dibatasi wheelie bukan wheelspin. Nilai optimum 15-26 tergantung torsi nyata.
8. **Spacing terbaik:** secara geometri **A lebih rapat dan merata** (step 1,31→1,15). Untuk ET, spacing B lebih dekat ke optimum pada gigi 2-3 (step 1,35 / 1,25 vs optimum 1,30 / 1,24) tetapi **step 1→2 B terlalu besar (1,61 vs ≈1,35-1,49 pada optimum)**; di situlah drop 38% terjadi.
9. **Algoritma paling andal:** GA dan PSO setara (std ≤ 0,001 s di 5 seed; keduanya mereproduksi sweep exhaustive pada masalah 1-D). Rekomendasi: **GA** sebagai default (elitism, outlier minimal), **PSO** bila evaluasi mahal (3.050 vs 4.080 evaluasi), **NSGA-II** untuk melihat trade-off, **exhaustive** sebagai baseline pada ruang diskret/Level 1.
10. **Rekomendasi akhir:** pertahankan **arsitektur System B (4-speed)**, dengan rasio total gigi 1 ≈ 19-21 dan final drive **≈ 2,3-2,43** (jika final fisiknya 34/14 = 2,4286 maka sudah praktis optimum untuk Skenario C dan dalam 0,02 s dari optimum Skenario B). Jangan memakai A dengan final 2,933 pada 201/402 m; A baru kompetitif bila final dinaikkan ke ≈ 4,1-4,6 (jika layak secara mekanis), dan itu pun hanya menyamai B tanpa keuntungan ET. Semua ini **teoretis/relatif** sampai data nyata tersedia.

---

## 5. Data yang belum ada & apa yang tetap bisa disimpulkan

**Dibutuhkan:** kurva dyno nyata (torsi/daya vs rpm), massa total, ukuran/jari-jari efektif ban belakang, wheelbase dan tinggi CG, μ lintasan, karakter engagement kopling sentrifugal (RPM engage, kapasitas torsi), waktu shift nyata, daftar sprocket/gear yang tersedia (jumlah gigi, modul, jarak sumbu, dimensi poros), konfirmasi final ratio B (2,500 atau 34/14).

**Tetap bisa disimpulkan tanpa data tersebut:** aritmetika rasio dan validasi data; RPM drop tiap shift; kecepatan per RPM (bergantung R_tire); fakta bahwa B jauh lebih "pendek" dari A; hubungan fisik Rtotal-only; serta urutan relatif (B lebih cepat dari A dengan rasio asli) yang tahan terhadap ±30% torsi, massa, ban, dan waktu shift. **Tidak bisa disimpulkan:** ET/terminal speed nyata, nilai optimum rasio yang pasti, atau kelayakan mekanis kombinasi gigi.

## 6. Keterbatasan model
Model kopling hanya slip konstan (tanpa panas/ramp); kopling sentrifugal tidak dimodelkan secara dinamis; kebijakan shift berbasis silang percepatan; transfer beban statis tanpa lift aero atau suspensi; batas wheelie (dari placeholder) sering menjadi pembatas; dt integrasi 5 ms membuat selisih < 0,01 s tidak signifikan; tidak ada analisis getaran/resonansi; optimum Level 2 kontinu dan **tidak** dibulatkan ke jumlah gigi (tidak ada data jumlah gigi/modul/sprocket).

## 7. Menjalankan ulang
```
python3 run_experiments.py s1   # baseline, sensitivitas, Level 1
python3 run_experiments.py s2   # Level 2 exhaustive/GA/PSO
python3 run_experiments.py s3   # NSGA-II
python3 run_experiments.py s4a; python3 run_experiments.py s4b   # per skenario
python3 run_experiments.py s5a; python3 run_experiments.py s5b   # reliabilitas
cp res.pkl results.pkl; python3 make_outputs.py                  # grafik + tabel
```
Ubah `Params` di `core.py` (atau `SCEN` untuk kurva dyno nyata) untuk mengganti placeholder.

---

# Lampiran - Seluruh tabel hasil
### TABLE A - Drivetrain specification (teeth-exact ratios)

| System | Primary Rp | Final Rf | Gears | Gearbox ratios | Overall Rtotal per gear | Step ratios (g_i/g_i+1) | Span 1st/top | Vmax @12k rpm top gear (km/h) |
|---|---|---|---|---|---|---|---|---|
| A: 6-speed manual | 1.627 | 2.933 | 6 | 2.692 / 2.059 / 1.650 / 1.409 / 1.222 / 1.034 | 12.853 / 9.829 / 7.877 / 6.727 / 5.835 / 4.938 | 1.308 / 1.248 / 1.171 / 1.153 / 1.181 | 2.603 | 265.655 |
| B1: 4-speed centrifugal (Rf=2.500 stated) | 3.350 | 2.500 | 4 | 2.500 / 1.550 / 1.150 / 0.923 | 20.938 / 12.981 / 9.631 / 7.731 | 1.613 / 1.348 / 1.246 | 2.708 | 169.702 |
| B2: 4-speed centrifugal (Rf=34/14) | 3.350 | 2.429 | 4 | 2.500 / 1.550 / 1.150 / 0.923 | 20.339 / 12.610 / 9.356 / 7.510 | 1.613 / 1.348 / 1.246 | 2.708 | 174.694 |

Stated vs teeth-exact: A primary 1.627 vs 83/51=1.6275; A overall @6th stated 4.938 vs 4.9384 (teeth-exact) - consistent. B final stated 2.500 vs 34/14 = 2.42857 (-2.857% relative to stated; stated is +2.94% above teeth value).


### B1 vs B2 - effect of the final-ratio discrepancy (2.500 vs 34/14)

| Config | Rf | Rtotal 1st | Rtotal 4th | Vmax 4th @12k (km/h) | ET201 (s) scen B | V201 (km/h) | ET402 (s) scen B | V402 (km/h) |
|---|---|---|---|---|---|---|---|---|
| B1: 4-speed centrifugal (Rf=2.500 stated) | 2.500 | 20.938 | 7.731 | 169.702 | 8.993 | 127.225 | 14.202 | 146.538 |
| B2: 4-speed centrifugal (Rf=34/14) | 2.429 | 20.339 | 7.510 | 174.694 | 8.983 | 126.931 | 14.182 | 147.431 |


### Rotational frequency (engine / gearbox output / wheel) - first and top gear

| System | Gear | Engine RPM | Engine Hz | Gearbox-out Hz | Wheel Hz | Wheel RPM | Speed km/h |
|---|---|---|---|---|---|---|---|
| A | 1 | 4000 | 66.667 | 15.215 | 5.187 | 311.219 | 34.025 |
| A | 6 | 4000 | 66.667 | 39.598 | 13.499 | 809.967 | 88.552 |
| A | 1 | 8000 | 133.333 | 30.430 | 10.374 | 622.438 | 68.050 |
| A | 6 | 8000 | 133.333 | 79.197 | 26.999 | 1619.934 | 177.103 |
| A | 1 | 12000 | 200.000 | 45.645 | 15.561 | 933.657 | 102.074 |
| A | 6 | 12000 | 200.000 | 118.795 | 40.498 | 2429.901 | 265.655 |
| B1 | 1 | 4000 | 66.667 | 7.960 | 3.184 | 191.045 | 20.886 |
| B1 | 4 | 4000 | 66.667 | 21.559 | 8.624 | 517.413 | 56.567 |
| B1 | 1 | 8000 | 133.333 | 15.920 | 6.368 | 382.090 | 41.773 |
| B1 | 4 | 8000 | 133.333 | 43.118 | 17.247 | 1034.826 | 113.135 |
| B1 | 1 | 12000 | 200.000 | 23.881 | 9.552 | 573.134 | 62.659 |
| B1 | 4 | 12000 | 200.000 | 64.677 | 25.871 | 1552.239 | 169.702 |
| B2 | 1 | 4000 | 66.667 | 7.960 | 3.278 | 196.664 | 21.501 |
| B2 | 4 | 4000 | 66.667 | 21.559 | 8.877 | 532.631 | 58.231 |
| B2 | 1 | 8000 | 133.333 | 15.920 | 6.555 | 393.327 | 43.001 |
| B2 | 4 | 8000 | 133.333 | 43.118 | 17.754 | 1065.262 | 116.462 |
| B2 | 1 | 12000 | 200.000 | 23.881 | 9.833 | 589.991 | 64.502 |
| B2 | 4 | 12000 | 200.000 | 64.677 | 26.632 | 1597.893 | 174.694 |

Rotational frequency only. No torsional vibration / resonance analysis was performed (no inertia/stiffness data).


### TABLE D - RPM before/after every gear change (ideal shift RPM from force crossover, per torque scenario)

| System | Shift | Rnext/Rcur | RPM drop % | After shift @12000 | Ideal shift rpm A | RPM after A | Ideal shift rpm B | RPM after B | Ideal shift rpm C | RPM after C |
|---|---|---|---|---|---|---|---|---|---|---|
| A (6-spd) | 1->2 | 0.765 | 23.529 | 9176.471 | 8275.000 | 6327.941 | 10150.000 | 7761.765 | 11900.000 | 9100.000 |
| A (6-spd) | 2->3 | 0.801 | 19.857 | 9617.143 | 8100.000 | 6491.571 | 9925.000 | 7954.179 | 11675.000 | 9356.679 |
| A (6-spd) | 3->4 | 0.854 | 14.601 | 10247.934 | 7875.000 | 6725.207 | 9650.000 | 8241.047 | 11375.000 | 9714.187 |
| A (6-spd) | 4->5 | 0.867 | 13.262 | 10408.602 | 7800.000 | 6765.591 | 9575.000 | 8305.197 | 11300.000 | 9801.434 |
| A (6-spd) | 5->6 | 0.846 | 15.361 | 10156.740 | 7900.000 | 6686.520 | 9675.000 | 8188.871 | 11425.000 | 9670.063 |
| B (4-spd, B1=B2) | 1->2 | 0.620 | 38.000 | 7440.000 | 9075.000 | 5626.500 | 11075.000 | 6866.500 | 12000.000 | 7440.000 |
| B (4-spd, B1=B2) | 2->3 | 0.742 | 25.806 | 8903.226 | 8400.000 | 6232.258 | 10275.000 | 7623.387 | 12000.000 | 8903.226 |
| B (4-spd, B1=B2) | 3->4 | 0.803 | 19.732 | 9632.107 | 8100.000 | 6501.672 | 9925.000 | 7966.555 | 11675.000 | 9371.237 |

Ideal shift RPM = lowest RPM (>=3000) where wheel force in the next gear >= current gear (drag ignored); 12000 means no crossover below the limiter. Zones: A 4000-8000, B 6000-10000, C 8000-12000.


### TABLE B - Condition A: low-mid RPM compatibility (Scenario A engine, 4000-8000 rpm = 66.7-133.3 Hz)

| System | Rtotal 1st | Wheel torque 1st @ zone-mid RPM (Nm, placeholder) | Peak a (m/s2) | Traction util. mean/peak % | Powerband util. % (201 m) | Mean RPM drop % | ET201 s | V201 km/h | ET402 s | V402 km/h | 1st gear speed @ zone top (km/h) | Launch RPM (Hz) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | 12.853 | 344.709 | 5.565 | 58 / 69 | 97.001 | 17.812 | 9.861 | 118.307 | 15.475 | 137.173 | 68.050 | 4000 (67 Hz) |
| B1 | 20.938 | 561.544 | 8.502 | 60 / 90 | 85.445 | 27.846 | 9.491 | 118.533 | 15.195 | 132.063 | 41.773 | 4000 (67 Hz) |
| B2 | 20.339 | 545.500 | 8.502 | 60 / 90 | 91.538 | 27.846 | 9.482 | 118.787 | 15.152 | 133.229 | 43.001 | 4000 (67 Hz) |


### TABLE C - Condition B: high-RPM compatibility (Scenario C engine, 8000-12000 rpm = 133.3-200 Hz)

| System | Rtotal 1st | Wheel torque 1st @ zone-mid RPM (Nm, placeholder) | Peak a (m/s2) | Traction util. mean/peak % | Powerband util. % (201 m) | Mean RPM drop % | ET201 s | V201 km/h | ET402 s | V402 km/h | Vmax top @12k km/h | RPM at 402 m (Hz) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | 12.853 | 344.709 | 5.437 | 66 / 70 | 97.864 | 21.693 | 9.294 | 131.683 | 14.246 | 156.893 | 265.655 | 11304 (188 Hz), gear 3 |
| B1 | 20.938 | 561.544 | 8.502 | 69 / 91 | 92.304 | 27.846 | 8.700 | 131.938 | 13.654 | 157.114 | 169.702 | 11110 (185 Hz), gear 4 |
| B2 | 20.339 | 545.500 | 8.502 | 69 / 91 | 93.165 | 27.846 | 8.689 | 133.508 | 13.653 | 156.911 | 174.694 | 10778 (180 Hz), gear 4 |


### Existing drivetrains under all three torque scenarios

| Scenario | System | ET201 | V201 | ET402 | V402 | Powerband % | Shifts (0-402 m run) | Shift RPM (sim, before) | Shift RPM (sim, after) |
|---|---|---|---|---|---|---|---|---|---|
| A | A | 9.861 | 118.307 | 15.475 | 137.173 | 97.001 | 4 | 7853, 7864, 7731, 7736 | 6005, 6303, 6602, 6710 |
| A | B1 | 9.491 | 118.533 | 15.195 | 132.063 | 85.445 | 3 | 8080, 7964, 7869 | 5010, 5909, 6316 |
| A | B2 | 9.482 | 118.787 | 15.152 | 133.229 | 91.538 | 3 | 8120, 7992, 7884 | 5035, 5930, 6328 |
| B | A | 9.461 | 127.078 | 14.675 | 148.359 | 97.926 | 3 | 9823, 9772, 9566 | 7512, 7831, 8170 |
| B | B1 | 8.993 | 127.225 | 14.202 | 146.538 | 95.794 | 3 | 10265, 9957, 9774 | 6364, 7388, 7846 |
| B | B2 | 8.983 | 126.931 | 14.182 | 147.431 | 95.571 | 3 | 10305, 9974, 9789 | 6389, 7400, 7857 |
| C | A | 9.294 | 131.683 | 14.246 | 156.893 | 97.864 | 2 | 11699, 11588 | 8946, 9287 |
| C | B1 | 8.700 | 131.938 | 13.654 | 157.114 | 92.304 | 3 | 12005, 11837, 11588 | 7443, 8782, 9301 |
| C | B2 | 8.689 | 133.508 | 13.653 | 156.911 | 93.165 | 3 | 12003, 11850, 11596 | 7442, 8792, 9308 |

Shift RPM columns are for the run to 402 m (the sim stops when 402 m is reached).


### Efficiency sensitivity (eta = 0.85 / 0.90 / 0.95)

| eta | Scenario | System | ET201 | ET402 | V402 |
|---|---|---|---|---|---|
| 0.850 | A | A | 10.104 | 15.836 | 134.223 |
| 0.850 | A | B1 | 9.671 | 15.469 | 130.184 |
| 0.850 | A | B2 | 9.669 | 15.436 | 131.256 |
| 0.850 | B | A | 9.706 | 15.031 | 145.148 |
| 0.850 | B | B1 | 9.150 | 14.459 | 144.205 |
| 0.850 | B | B2 | 9.147 | 14.453 | 144.910 |
| 0.850 | C | A | 9.541 | 14.608 | 153.788 |
| 0.850 | C | B1 | 8.845 | 13.910 | 153.834 |
| 0.850 | C | B2 | 8.871 | 13.944 | 153.311 |
| 0.900 | A | A | 9.861 | 15.475 | 137.173 |
| 0.900 | A | B1 | 9.491 | 15.195 | 132.063 |
| 0.900 | A | B2 | 9.482 | 15.152 | 133.229 |
| 0.900 | B | A | 9.461 | 14.675 | 148.359 |
| 0.900 | B | B1 | 8.993 | 14.202 | 146.538 |
| 0.900 | B | B2 | 8.983 | 14.182 | 147.431 |
| 0.900 | C | A | 9.294 | 14.246 | 156.893 |
| 0.900 | C | B1 | 8.700 | 13.654 | 157.114 |
| 0.900 | C | B2 | 8.689 | 13.653 | 156.911 |
| 0.950 | A | A | 9.638 | 15.143 | 139.968 |
| 0.950 | A | B1 | 9.326 | 14.945 | 133.814 |
| 0.950 | A | B2 | 9.317 | 14.899 | 135.066 |
| 0.950 | B | A | 9.236 | 14.347 | 151.359 |
| 0.950 | B | B1 | 8.848 | 13.966 | 148.672 |
| 0.950 | B | B2 | 8.840 | 13.944 | 149.733 |
| 0.950 | C | A | 9.068 | 13.922 | 158.620 |
| 0.950 | C | B1 | 8.574 | 13.425 | 160.034 |
| 0.950 | C | B2 | 8.560 | 13.424 | 160.163 |


### Robustness of the conclusions to the ASSUMED placeholders (Scenario B, 201 m)

| Variation | ET201 A | ET201 B1 | ET201 B2 | A - B1 (s) | Level-1 Rf* (A) | ET201 A @Rf* | Level-1 Rf* (B) | ET201 B @Rf* | Opt 1st-gear Rtotal A | Opt 1st-gear Rtotal B |
|---|---|---|---|---|---|---|---|---|---|---|
| baseline | 9.461 | 8.993 | 8.983 | 0.468 | 4.420 | 8.964 | 2.320 | 8.964 | 19.367 | 19.430 |
| T_peak x0.7 | 11.127 | 10.308 | 10.332 | 0.819 | 4.980 | 10.270 | 3.140 | 10.227 | 21.820 | 26.298 |
| T_peak x1.3 | 8.444 | 8.312 | 8.309 | 0.132 | 3.420 | 8.249 | 1.820 | 8.276 | 14.985 | 15.243 |
| mu 0.8 | 9.461 | 9.183 | 9.177 | 0.279 | 3.680 | 9.098 | 1.940 | 9.139 | 16.124 | 16.247 |
| mu 1.2 | 9.461 | 8.993 | 8.983 | 0.468 | 4.420 | 8.964 | 2.320 | 8.964 | 19.367 | 19.430 |
| shift 0.05s | 9.431 | 8.935 | 8.927 | 0.496 | 4.420 | 8.875 | 2.320 | 8.914 | 19.367 | 19.430 |
| shift 0.20s | 9.527 | 9.122 | 9.104 | 0.406 | 4.240 | 9.153 | 2.320 | 9.072 | 18.578 | 19.430 |
| r_tire 0.27 | 9.346 | 9.032 | 9.022 | 0.313 | 4.180 | 8.984 | 2.180 | 8.983 | 18.315 | 18.258 |
| r_tire 0.31 | 9.597 | 8.955 | 8.956 | 0.642 | 4.700 | 8.947 | 2.460 | 8.948 | 20.594 | 20.602 |
| mass 150 | 9.151 | 8.836 | 8.828 | 0.315 | 4.060 | 8.766 | 2.140 | 8.776 | 17.789 | 17.923 |
| mass 180 | 9.763 | 9.148 | 9.166 | 0.615 | 4.820 | 9.152 | 2.520 | 9.146 | 21.119 | 21.105 |


### LEVEL 1 - Final-drive optimisation with supplied primary + gearbox (theoretical ratio targets only)

| Scenario | Base system | Dist m | Ref Rf | ET @ref | Rf* (theory, <=5.0) | ET @Rf* | Rf* (<=4.3) | Gain vs ref s | Rtotal 1st @Rf* | Rtotal top @Rf* | Vtop @12k at Rf* (km/h) | V at finish km/h |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | A | 201 | 2.933 | 9.857 | 4.400 | 9.523 | 4.300 | 0.335 | 19.279 | 7.408 | 177.103 | 118.453 |
| A | A | 402 | 2.933 | 15.472 | 4.080 | 15.164 | 4.080 | 0.307 | 17.877 | 6.869 | 190.994 | 135.945 |
| A | B1 | 201 | 2.500 | 9.491 | 2.320 | 9.470 | 2.320 | 0.022 | 19.430 | 7.174 | 182.869 | 118.937 |
| A | B2 | 201 | 2.429 | 9.483 | 2.320 | 9.470 | 2.320 | 0.013 | 19.430 | 7.174 | 182.869 | 118.937 |
| A | B1 | 402 | 2.500 | 15.195 | 2.280 | 15.093 | 2.280 | 0.102 | 19.095 | 7.050 | 186.077 | 135.373 |
| A | B2 | 402 | 2.429 | 15.149 | 2.280 | 15.093 | 2.280 | 0.055 | 19.095 | 7.050 | 186.077 | 135.373 |
| B | A | 201 | 2.933 | 9.456 | 4.420 | 8.964 | 4.300 | 0.491 | 19.367 | 7.441 | 176.302 | 126.606 |
| B | A | 402 | 2.933 | 14.669 | 4.420 | 14.167 | 4.300 | 0.502 | 19.367 | 7.441 | 176.302 | 147.603 |
| B | B1 | 201 | 2.500 | 8.993 | 2.320 | 8.964 | 2.320 | 0.029 | 19.430 | 7.174 | 182.869 | 126.446 |
| B | B2 | 201 | 2.429 | 8.981 | 2.320 | 8.964 | 2.320 | 0.017 | 19.430 | 7.174 | 182.869 | 126.446 |
| B | B1 | 402 | 2.500 | 14.202 | 2.320 | 14.163 | 2.320 | 0.039 | 19.430 | 7.174 | 182.869 | 148.326 |
| B | B2 | 402 | 2.429 | 14.179 | 2.320 | 14.163 | 2.320 | 0.016 | 19.430 | 7.174 | 182.869 | 148.326 |
| C | A | 201 | 2.933 | 9.288 | 4.580 | 8.594 | 4.300 | 0.694 | 20.068 | 7.711 | 170.143 | 134.271 |
| C | A | 402 | 2.933 | 14.239 | 4.580 | 13.522 | 4.300 | 0.717 | 20.068 | 7.711 | 170.143 | 157.495 |
| C | B1 | 201 | 2.500 | 8.700 | 2.400 | 8.685 | 2.400 | 0.014 | 20.100 | 7.422 | 176.773 | 133.537 |
| C | B2 | 201 | 2.429 | 8.688 | 2.400 | 8.685 | 2.400 | 0.003 | 20.100 | 7.422 | 176.773 | 133.537 |
| C | B1 | 402 | 2.500 | 13.654 | 2.480 | 13.651 | 2.480 | 0.003 | 20.770 | 7.669 | 171.071 | 157.109 |
| C | B2 | 402 | 2.429 | 13.652 | 2.480 | 13.651 | 2.480 | 0.001 | 20.770 | 7.669 | 171.071 | 157.109 |

No sprocket candidate list was supplied, so only theoretical ratio targets are reported. B1 and B2 share the same optimum curve (same primary+gearbox); they only differ in where the baseline sits on it. Note the sweep is flat near the optimum.


### TABLE E - Algorithm performance, Level 2 (Scenario B engine, objective = ET at 201 m)

| Arch | Algorithm | Rp | Rf | Gearbox ratios | Overall Rtotal | ET201 s | ET402 s | V402 km/h | Convergence | Evals | vs Exhaustive (s) |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 6-speed | Exhaustive | 2.000 | 2.933 | 3.300 / 2.241 / 1.760 / 1.432 / 1.188 / 1.000 | 19.36 / 13.15 / 10.32 / 8.40 / 6.97 / 5.87 | 8.941 | 14.143 | 148.591 | n/a (2700 grid pts, 2700 feasible) | 2700 | 0.000 |
| 6-speed | GA | 2.233 | 3.003 | 2.899 / 2.082 / 1.588 / 1.300 / 0.973 / 0.904 | 19.44 / 13.96 / 10.65 / 8.72 / 6.52 / 6.06 | 8.933 | 14.177 | 147.805 | 20 gen to within 5 ms of final | 4080 | -0.008 |
| 6-speed | PSO | 2.078 | 4.200 | 2.227 / 1.591 / 1.201 / 0.958 / 0.780 / 0.750 | 19.43 / 13.89 / 10.48 / 8.36 / 6.81 / 6.54 | 8.935 | 14.151 | 147.639 | 16 iter to within 5 ms of final | 3050 | -0.006 |
| 6-speed | NSGA-II (min-ET member) | 2.371 | 3.783 | 2.167 / 1.610 / 1.271 / 1.070 / 0.948 / 0.750 | 19.43 / 14.45 / 11.40 / 9.60 / 8.50 / 6.73 | 8.948 | 14.172 | 148.332 | 80 Pareto pts, 50 gen | 4080 | 0.007 |
| 6-speed | NSGA-II (knee) | 2.353 | 3.830 | 2.163 / 1.628 / 1.273 / 1.078 / 0.905 / 0.756 | 19.50 / 14.67 / 11.47 / 9.71 / 8.16 / 6.81 | 8.948 | 14.157 | 148.532 | knee: equal-weight min-max on ET201 & V402 | 4080 | 0.007 |
| 4-speed | Exhaustive | 3.350 | 2.933 | 2.000 / 1.381 / 1.096 / 0.900 | 19.65 / 13.57 / 10.77 / 8.84 | 8.939 | 14.290 | 140.049 | n/a (2700 grid pts, 2124 feasible) | 2700 | 0.000 |
| 4-speed | GA | 3.368 | 2.494 | 2.322 / 1.665 / 1.278 / 1.033 | 19.50 / 13.98 / 10.73 / 8.68 | 8.933 | 14.253 | 141.153 | 9 gen to within 5 ms of final | 4080 | -0.006 |
| 4-speed | PSO | 3.322 | 2.360 | 2.478 / 1.788 / 1.361 / 1.100 | 19.43 / 14.02 / 10.67 / 8.62 | 8.933 | 14.244 | 141.489 | 13 iter to within 5 ms of final | 3050 | -0.006 |
| 4-speed | NSGA-II (min-ET member) | 2.974 | 2.728 | 2.403 / 1.712 / 1.300 / 1.074 | 19.49 / 13.88 / 10.54 / 8.71 | 8.934 | 14.261 | 140.905 | 80 Pareto pts, 50 gen | 4080 | -0.005 |
| 4-speed | NSGA-II (knee) | 3.308 | 2.310 | 2.543 / 1.701 / 1.225 / 0.914 | 19.43 / 13.00 / 9.36 / 6.98 | 8.954 | 14.167 | 148.416 | knee: equal-weight min-max on ET201 & V402 | 4080 | 0.014 |

All variables repaired to descending, step in [1.04, 1.65]; Rp in [1.2,4.0], Rf in [1.8,4.2], 1st gear <= 3.3. NOTE: only the product Rp x g_i x Rf affects the physics, so many (Rp, Rf, g) combinations are physically identical.


### Reliability over 5 random seeds (Scenario B, 201 m)

| Arch | Algorithm | Best ET201 (mean) | Std (s) | Min | Max | Seeds | Exhaustive best |
|---|---|---|---|---|---|---|---|
| 6-speed | GA | 8.933 | 0.000 | 8.933 | 8.933 | 5 | 8.941 |
| 6-speed | PSO | 8.933 | 0.000 | 8.933 | 8.934 | 5 | 8.941 |
| 4-speed | GA | 8.933 | 0.001 | 8.933 | 8.934 | 5 | 8.939 |
| 4-speed | PSO | 8.933 | 0.000 | 8.933 | 8.933 | 5 | 8.939 |


### Level-1 validation: GA/PSO vs exhaustive sweep on the 1-D final-drive problem (Scenario B)

| Base | Method | Rf* | ET201 | Evals |
|---|---|---|---|---|
| A | Exhaustive sweep (step 0.02) | 4.420 | 8.964 | 161 |
| A | GA (1-D) | 4.423 | 8.963 | 520 |
| A | PSO (1-D) | 4.424 | 8.963 | 520 |
| B1 | Exhaustive sweep (step 0.02) | 2.320 | 8.964 | 161 |
| B1 | GA (1-D) | 2.321 | 8.964 | 520 |
| B1 | PSO (1-D) | 2.320 | 8.964 | 520 |


### LEVEL 2 - theoretical optimum per engine scenario / distance (GA vs PSO, best kept)

| Engine scen | Opt. distance m | Arch | Best alg | Rp | Rf | Gearbox | Overall Rtotal | ET201 | V201 | ET402 | V402 | GA ET | PSO ET |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | 201 | 6-speed | GA | 3.138 | 1.915 | 3.235 / 2.173 / 1.599 / 1.263 / 0.886 / 0.852 | 19.44 / 13.06 / 9.61 / 7.59 / 5.33 / 5.12 | 9.463 | 118.749 | 15.124 | 135.661 | 9.463 | 9.463 |
| B | 201 | 6-speed | GA | 3.115 | 2.043 | 3.054 / 2.194 / 1.685 / 1.362 / 0.951 / 0.824 | 19.43 / 13.96 / 10.72 / 8.66 / 6.05 / 5.25 | 8.933 | 127.311 | 14.223 | 145.278 | 8.933 | 8.933 |
| C | 201 | 6-speed | GA | 2.057 | 3.754 | 2.580 / 1.931 / 1.532 / 1.257 / 0.930 / 0.786 | 19.92 / 14.91 / 11.83 / 9.71 / 7.18 / 6.07 | 8.586 | 133.656 | 13.596 | 155.115 | 8.586 | 8.586 |
| B | 402 | 6-speed | GA | 3.083 | 2.093 | 3.012 / 2.152 / 1.642 / 1.323 / 1.112 / 0.780 | 19.44 / 13.88 / 10.59 / 8.53 / 7.17 / 5.03 | 8.939 | 126.577 | 14.135 | 148.372 | 14.135 | 14.135 |
| A | 201 | 4-speed | PSO | 3.400 | 2.417 | 2.366 / 1.589 / 1.167 / 0.924 | 19.44 / 13.06 / 9.59 / 7.59 | 9.463 | 118.748 | 15.144 | 132.821 | 9.463 | 9.463 |
| B | 201 | 4-speed | GA | 3.248 | 2.593 | 2.308 / 1.670 / 1.287 / 1.036 | 19.44 / 14.07 / 10.84 / 8.73 | 8.933 | 127.210 | 14.262 | 140.810 | 8.933 | 8.933 |
| C | 201 | 4-speed | GA | 2.292 | 3.285 | 2.645 / 1.967 / 1.556 / 1.283 | 19.92 / 14.81 / 11.71 / 9.66 | 8.586 | 133.774 | 13.915 | 135.862 | 8.586 | 8.586 |
| B | 402 | 4-speed | GA | 3.152 | 2.435 | 2.532 / 1.718 / 1.257 / 0.979 | 19.44 / 13.19 / 9.65 / 7.51 | 8.948 | 127.008 | 14.146 | 147.434 | 14.146 | 14.146 |


### Existing vs Level-2 optimised (theoretical)

| Engine scen | Dist m | Arch | Existing ET | Optimised ET | Gain s | Gain % |
|---|---|---|---|---|---|---|
| A | 201 | 6-speed | 9.861 | 9.463 | 0.398 | 4.035 |
| A | 201 | 4-speed | 9.491 | 9.463 | 0.029 | 0.302 |
| B | 201 | 6-speed | 9.461 | 8.933 | 0.528 | 5.583 |
| B | 201 | 4-speed | 8.993 | 8.933 | 0.060 | 0.669 |
| C | 201 | 6-speed | 9.294 | 8.586 | 0.708 | 7.618 |
| C | 201 | 4-speed | 8.700 | 8.586 | 0.114 | 1.308 |
| B | 402 | 6-speed | 14.675 | 14.135 | 0.540 | 3.678 |
| B | 402 | 4-speed | 14.202 | 14.146 | 0.056 | 0.393 |


### TABLE F - Final recommendation (THEORETICAL optimum; see Level-1 table for what can be reached by changing the final drive only)

| Config | Arch | Engine scen | Optimised for | Rp | Rf | Gearbox | Overall Rtotal | ET201 | V201 | ET402 | V402 | Powerband % | Note |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1. Best LOW-RPM (theoretical) | 6-speed | A | 201 m | 3.138 | 1.915 | 3.235 / 2.173 / 1.599 / 1.263 / 0.886 / 0.852 | 19.44 / 13.06 / 9.61 / 7.59 / 5.33 / 5.12 | 9.463 | 118.749 | 15.124 | 135.661 | 89.456 | Level 2 continuous optimum |
| 1. Best LOW-RPM (theoretical) | 4-speed | A | 201 m | 3.400 | 2.417 | 2.366 / 1.589 / 1.167 / 0.924 | 19.44 / 13.06 / 9.59 / 7.59 | 9.463 | 118.748 | 15.144 | 132.821 | 89.402 | Level 2 continuous optimum |
| 2. Best HIGH-RPM (theoretical) | 6-speed | C | 201 m | 2.057 | 3.754 | 2.580 / 1.931 / 1.532 / 1.257 / 0.930 / 0.786 | 19.92 / 14.91 / 11.83 / 9.71 / 7.18 / 6.07 | 8.586 | 133.656 | 13.596 | 155.115 | 96.558 | Level 2 continuous optimum |
| 2. Best HIGH-RPM (theoretical) | 4-speed | C | 201 m | 2.292 | 3.285 | 2.645 / 1.967 / 1.556 / 1.283 | 19.92 / 14.81 / 11.71 / 9.66 | 8.586 | 133.774 | 13.915 | 135.862 | 96.555 | Level 2 continuous optimum |
| 3. Best OVERALL (theoretical) | 6-speed | B | 201 m | 3.115 | 2.043 | 3.054 / 2.194 / 1.685 / 1.362 / 0.951 / 0.824 | 19.43 / 13.96 / 10.72 / 8.66 / 6.05 / 5.25 | 8.933 | 127.311 | 14.223 | 145.278 | 94.313 | Level 2 continuous optimum |
| 3. Best OVERALL (theoretical) | 4-speed | B | 201 m | 3.248 | 2.593 | 2.308 / 1.670 / 1.287 / 1.036 | 19.44 / 14.07 / 10.84 / 8.73 | 8.933 | 127.210 | 14.262 | 140.810 | 92.633 | Level 2 continuous optimum |
| 3b. Overall, 402 m | 6-speed | B | 402 m | 3.083 | 2.093 | 3.012 / 2.152 / 1.642 / 1.323 / 1.112 / 0.780 | 19.44 / 13.88 / 10.59 / 8.53 / 7.17 / 5.03 | 8.939 | 126.577 | 14.135 | 148.372 | 95.533 | Level 2 continuous optimum |
| 3b. Overall, 402 m | 4-speed | B | 402 m | 3.152 | 2.435 | 2.532 / 1.718 / 1.257 / 0.979 | 19.44 / 13.19 / 9.65 / 7.51 | 8.948 | 127.008 | 14.146 | 147.434 | 96.611 | Level 2 continuous optimum |
