# 🐜 ACO untuk Degree Constrained Minimum Spanning Tree (DCMST) D-MAX=3
## Panduan Lengkap dengan Perhitungan Step-by-Step
### Degree Constraint: d_max = 3

---

# BAGIAN 1: PENGANTAR ACO

## 1.1 Apa itu ACO?

**Ant Colony Optimization (ACO)** adalah algoritma metaheuristik yang terinspirasi dari perilaku semut dalam mencari makanan menggunakan **feromon**.

```
╔═════════════════════════════════════════════════════════════════════╗
║  PRINSIP DASAR ACO:                                                 ║
╠═════════════════════════════════════════════════════════════════════╣
║  1. Semut meninggalkan FEROMON di jalur yang dilalui                ║
║  2. Semut lain cenderung MENGIKUTI jalur dengan feromon lebih kuat  ║
║  3. Feromon MENGUAP seiring waktu (evaporation)                     ║
║  4. Jalur PENDEK → lebih sering dilalui → feromon LEBIH KUAT        ║
║  5. Jalur PANJANG → jarang dilalui → feromon MENGHILANG             ║
╠═════════════════════════════════════════════════════════════════════╣
║  HASIL: Secara kolektif, semut menemukan JALUR TERPENDEK!           ║
╚═════════════════════════════════════════════════════════════════════╝
```

## 1.2 DCMST (Degree Constrained MST)


**DCMST** adalah masalah mencari Minimum Spanning Tree dimana setiap vertex memiliki batasan maksimum degree (jumlah edge yang boleh terhubung).

```
┌─────────────────────────────────────────────────────────────────────┐
│  DCMST CONSTRAINT:                                                  │
│                                                                     │
│  • Setiap vertex v: degree(v) ≤ d_max                              │
│  • Dalam contoh ini: d_max = 2 (setiap kota maksimal 2 koneksi)    │
│                                                                     │
│  Mengapa penting?                                                   │
│  → Merepresentasikan kapasitas terbatas (misal: bandwidth router)  │
│  → Mencegah bottleneck pada satu node                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

# BAGIAN 2: RUMUS-RUMUS ACO-DCMST

## 2.1 Notasi dan Parameter

```
┌─────────────────────────────────────────────────────────────────────┐
│  NOTASI:                                                            │
│  • V = {A, B, C, D, E} : Himpunan vertex (5 kota)                   │
│  • E = himpunan edge dengan bobot d[i][j]                           │
│  • τ[i][j] : Feromon pada edge (i,j)                                │
│  • η[i][j] : Heuristik = 1/d[i][j] (inverse jarak)                  │
│  • T : Himpunan vertex yang sudah ada di tree                       │
│  • deg[v] : Degree saat ini dari vertex v                           │
├─────────────────────────────────────────────────────────────────────┤
│  PARAMETER:                                                         │
│  • α = 1   : Bobot pengaruh feromon                                 │
│  • β = 2   : Bobot pengaruh heuristik                               │
│  • ρ = 0.5 : Tingkat evaporasi feromon (50%)                        │
│  • Q = 100 : Konstanta deposit feromon                              │
│  • τ₀ = 1.0 : Feromon awal                                          │
│  • d_max = 2 : Degree maksimum per vertex                           │
│  • n_ants = 3 : Jumlah semut  (dalm case ini dimisalkan ada 3)                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Rumus Utama ACO-DCMST

### RUMUS 1: Heuristik (Visibility)
```
                    1
    η[i][j] = ─────────
               d[i][j]

Semakin pendek jarak, semakin tinggi nilai heuristik
→ Edge pendek lebih "menarik" untuk dipilih
```

### RUMUS 2: Probabilitas Pemilihan Edge
```
                      [τ[u][v]]^α × [η[u][v]]^β × feasible(u,v)
    P(u,v) = ──────────────────────────────────────────────────────
              Σ [τ[u][w]]^α × [η[u][w]]^β × feasible(u,w)
              w∈candidates

Dimana:
• τ[u][v]^α = faktor feromon (pengalaman kolektif)
• η[u][v]^β = faktor heuristik (kualitas edge)
• feasible(u,v) = 1 jika edge valid (tidak melanggar degree), 0 otherwise
• candidates = {w : w ∉ T, deg[u] < d_max, deg[w] < d_max}
```

### RUMUS 3: Evaporasi Feromon
```
    τ[i][j] ← (1 - ρ) × τ[i][j]

Dengan ρ = 0.5:
    τ[i][j] ← 0.5 × τ[i][j]

→ Feromon berkurang 50% setiap iterasi
→ "Melupakan" jalur yang jarang digunakan
```

### RUMUS 4: Deposit Feromon
```
    Untuk setiap edge (i,j) dalam solusi valid:
    
              Q
    Δτ = ─────────
          Cost(T)

    τ[i][j] ← τ[i][j] + Δτ

Dengan Q = 100:
• Solusi dengan cost rendah → deposit lebih besar
• Memperkuat jalur yang menghasilkan solusi baik
```

### RUMUS 5: Update Feromon Lengkap
```
    τ[i][j] ← (1 - ρ) × τ[i][j] + Σ Δτₖ
                                  k=1..m

Dimana m = jumlah semut yang menggunakan edge (i,j)
```

---

# BAGIAN 3: INPUT DAN INISIALISASI

## 3.1 Input: Graf 5 Kota (case dri Prof Wamiliana/Ibu Dian)

```
        A ──────4────── B
       /|\              |\
      / | \             | \
     7  |  9            4  13
    /   |   \           |   \
   E    11   \          C    D
    \   |     \        /    /
     \  |      \      6    /
      8 |       \    /    8
       \|        \  /    /
        D ────────\/────/
                  C

Distance Matrix d[i][j]:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│     │  A  │  B  │  C  │  D  │  E  │
├─────┼─────┼─────┼─────┼─────┼─────┤
│  A  │  0  │  4  │  9  │ 11  │  7  │
│  B  │  4  │  0  │  4  │ 13  │ 10  │
│  C  │  9  │  4  │  0  │  6  │  2  │
│  D  │ 11  │ 13  │  6  │  0  │  8  │
│  E  │  7  │ 10  │  2  │  8  │  0  │
└─────┴─────┴─────┴─────┴─────┴─────┘
```

## 3.2 Inisialisasi Feromon τ₀ = 1.0

```
Initial Pheromone Matrix τ[i][j]:
┌─────┬─────┬─────┬─────┬─────┬─────┐
│     │  A  │  B  │  C  │  D  │  E  │
├─────┼─────┼─────┼─────┼─────┼─────┤
│  A  │  -  │ 1.0 │ 1.0 │ 1.0 │ 1.0 │
│  B  │ 1.0 │  -  │ 1.0 │ 1.0 │ 1.0 │
│  C  │ 1.0 │ 1.0 │  -  │ 1.0 │ 1.0 │
│  D  │ 1.0 │ 1.0 │ 1.0 │  -  │ 1.0 │
│  E  │ 1.0 │ 1.0 │ 1.0 │ 1.0 │  -  │
└─────┴─────┴─────┴─────┴─────┴─────┘
```

## 3.3 Hitung Heuristik η[i][j] = 1/d[i][j]

```
Heuristic Matrix η[i][j]:
┌─────┬───────┬───────┬───────┬───────┬───────┐
│     │   A   │   B   │   C   │   D   │   E   │
├─────┼───────┼───────┼───────┼───────┼───────┤
│  A  │   -   │ 0.250 │ 0.111 │ 0.091 │ 0.143 │
│  B  │ 0.250 │   -   │ 0.250 │ 0.077 │ 0.100 │
│  C  │ 0.111 │ 0.250 │   -   │ 0.167 │ 0.500 │
│  D  │ 0.091 │ 0.077 │ 0.167 │   -   │ 0.125 │
│  E  │ 0.143 │ 0.100 │ 0.500 │ 0.125 │   -   │
└─────┴───────┴───────┴───────┴───────┴───────┘

Contoh perhitungan:
• η[A][B] = 1/4 = 0.250
• η[C][E] = 1/2 = 0.500 (tertinggi! → edge terpendek)
• η[A][D] = 1/11 = 0.091 (terendah → edge terpanjang)
```

---

# Sebelum ke ITERASI 1 :
## SEMUT 1: Konstruksi Tree
### SEMUT 1-Step 1: Inisialisasi
```
┌─────────────────────────────────────────────────────────────────────┐
│  SEMUT 1 - START                                                    │
├─────────────────────────────────────────────────────────────────────┤
│  • Start vertex: A (central vertex)                                 │
│  • T (tree vertices) = {A}                                          │
│  • Tree edges = []                                                  │
│  • deg[A]=0, deg[B]=0, deg[C]=0, deg[D]=0, deg[E]=0                 │
│  • d_max = 2 (setiap vertex maksimal 2 koneksi)                     │
│  • Target: 4 edges (n-1 = 5-1 = 4)                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### SEMUT 1-Step 2: Pilih Edge Pertama (dari A)
```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 1 - EDGE 1                                                 │
├─────────────────────────────────────────────────────────────────────┤
│  T = {A}, mencari edge ke vertex di luar T                          │
│  Candidates: (A,B), (A,C), (A,D), (A,E)                             │
│                                                                     │
│  Semua feasible karena deg[A]=0 < 2 dan deg[*]=0 < 2                │
└─────────────────────────────────────────────────────────────────────┘

Hitung nilai τ^α × η^β untuk setiap candidate:

• (A,B): τ[A][B]^1 × η[A][B]^2 = 1.0^1 × 0.250^2 = 1 × 0.0625 = 0.0625
• (A,C): τ[A][C]^1 × η[A][C]^2 = 1.0^1 × 0.111^2 = 1 × 0.0123 = 0.0123
• (A,D): τ[A][D]^1 × η[A][D]^2 = 1.0^1 × 0.091^2 = 1 × 0.0083 = 0.0083
• (A,E): τ[A][E]^1 × η[A][E]^2 = 1.0^1 × 0.143^2 = 1 × 0.0204 = 0.0204

Total = 0.0625 + 0.0123 + 0.0083 + 0.0204 = 0.1035

PROBABILITAS :
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,B)  │   0.0625   │ 0.0625/0.1035 = 60.4%   │
│ (A,C)  │   0.0123   │ 0.0123/0.1035 = 11.9%   │
│ (A,D)  │   0.0083   │ 0.0083/0.1035 =  8.0%   │
│ (A,E)  │   0.0204   │ 0.0204/0.1035 = 19.7%   │
└────────┴────────────┴─────────────────────────┘

→ PILIH (A,B) dengan probabilitas tertinggi (60.4%)
  (Atau bisa juga dengan menggunakan roulette wheel selection/dengan menggunakan cara random terhadap komulatif probability)
```

## SEMUT 1-Hasil Heuristic η = 1/d:

```
         A        B        C        D        E
A        -     0.2500   0.1111   0.0909   0.1429
B     0.2500      -     0.2500   0.0769   0.1000
C     0.1111   0.2500      -     0.1667   0.5000
D     0.0909   0.0769   0.1667      -     0.1250
E     0.1429   0.1000   0.5000   0.1250      -

⭐ η[C][E] = 0.5000 TERTINGGI (edge terpendek C-E = 2)
```

---

# BAGIAN 4: ITERASI 1 - KONSTRUKSI SOLUSI (HASIL HITUNG PROBABILITAS atas η = 1/d)

## 🐜 SEMUT 1

### Edge 1: Dari A ke {B,C,D,E}

```
Candidates (semua feasible karena deg=0 < 3):
┌─────────┬─────────┬─────────┬─────────────────────┬─────────────┐
│  Edge   │    τ    │    η    │   τ^1 × η^2         │    Prob     │
├─────────┼─────────┼─────────┼─────────────────────┼─────────────┤
│  (A,B)  │   1.0   │  0.2500 │  1.0 × 0.0625=0.0625│   60.39%    │
│  (A,C)  │   1.0   │  0.1111 │  1.0 × 0.0123=0.0123│   11.88%    │
│  (A,D)  │   1.0   │  0.0909 │  1.0 × 0.0083=0.0083│    8.02%    │
│  (A,E)  │   1.0   │  0.1429 │  1.0 × 0.0204=0.0204│   19.71%    │
└─────────┴─────────┴─────────┴─────────────────────┴─────────────┘
TOTAL = 0.1035

✅ PILIH (A,B) cost=4 [prob tertinggi 60.39%]
   T={A,B}, deg[A]=1, deg[B]=1
```

### Edge 2: Dari {A,B} ke {C,D,E}

```
Candidates (semua feasible karena deg < 3):
┌─────────┬──────────┬─────────────┐
│  Edge   │   Nilai  │    Prob     │
├─────────┼──────────┼─────────────┤
│  (A,C)  │  0.0123  │   10.30%    │
│  (A,D)  │  0.0083  │    6.95%    │
│  (A,E)  │  0.0204  │   17.09%    │
│  (B,C)  │  0.0625  │   52.35%    │ =>TERTINGGI
│  (B,D)  │  0.0059  │    4.94%    │
│  (B,E)  │  0.0100  │    8.38%    │
└─────────┴──────────┴─────────────┘
TOTAL = 0.1194

✅ PILIH (B,C) cost=4
   T={A,B,C}, deg[A]=1, deg[B]=2, deg[C]=1
```

### Edge 3: Dari {A,B,C} ke {D,E}

```
Candidates (semua feasible karena deg < 3):
┌─────────┬──────────┬─────────────┐
│  Edge   │   Nilai  │    Prob     │
├─────────┼──────────┼─────────────┤
│  (A,D)  │  0.0083  │    2.57%    │
│  (A,E)  │  0.0204  │    6.33%    │
│  (B,D)  │  0.0059  │    1.83%    │
│  (B,E)  │  0.0100  │    3.10%    │
│  (C,D)  │  0.0278  │    8.62%    │
│  (C,E)  │  0.2500  │   77.54%    │ =>SANGAT TINGGI!
└─────────┴──────────┴─────────────┘
TOTAL = 0.3224

✅ PILIH (C,E) cost=2 [edge terpendek!]
   T={A,B,C,E}, deg[A]=1, deg[B]=2, deg[C]=2, deg[E]=1
```

### Edge 4: Dari {A,B,C,E} ke {D}

```
Candidates untuk menghubungkan D:
┌─────────┬──────────┬─────────────┐
│  Edge   │   Nilai  │    Prob     │
├─────────┼──────────┼─────────────┤
│  (A,D)  │  0.0083  │   14.41%    │
│  (B,D)  │  0.0059  │   10.24%    │
│  (C,D)  │  0.0278  │   48.26%    │ =>TERTINGGI
│  (E,D)  │  0.0156  │   27.08%    │
└─────────┴──────────┴─────────────┘
TOTAL = 0.0576

✅ PILIH (C,D) cost=6
   T={A,B,C,D,E} - SELESAI!
```

### ⭐ HASIL SEMUT 1:

```
╔═════════════════════════════════════════════════════════════════════╗
║  Tree Edges: [(A,B,4), (B,C,4), (C,E,2), (C,D,6)]                   ║
║  TOTAL COST = 4 + 4 + 2 + 6 = 16                                    ║
║                                                                     ║
║  Degree Check (d_max = 3):                                          ║
║  • deg(A) = 1 ≤ 3                                                   ║
║  • deg(B) = 2 ≤ 3                                                   ║
║  • deg(C) = 3 ≤ 3   (terhubung ke B, E, D)                          ║
║  • deg(D) = 1 ≤ 3                                                   ║
║  • deg(E) = 1 ≤ 3                                                   ║
║                                                                     ║
║  OK! VALID DCMST!                                                   ║
╚═════════════════════════════════════════════════════════════════════╝

    (A)════4════(B)════4════(C)════6════(D)
                             ║
                             2
                             ║
                            (E)
```

---

## 🐜 SEMUT 2 (Jalur Alternatif)

```
Edge 1: Random=0.85 → pilih (A,E) cost=7
Edge 2: (E,C) tertinggi → pilih (E,C) cost=2
Edge 3: Pilih (A,B) cost=4
Edge 4: Pilih (C,D) cost=6

╔═════════════════════════════════════════════════════════════════════╗
║  Tree Edges: [(A,E,7), (E,C,2), (A,B,4), (C,D,6)]                   ║
║  TOTAL COST = 7 + 2 + 4 + 6 = 19                                    ║
║  Degree: A=2, B=1, C=2, D=1, E=2 (semua ≤ 3 )                    	  ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

## 🐜 SEMUT 3 (Jalur Alternatif Lain)

```
Edge 1: (A,B) cost=4
Edge 2: (B,C) cost=4
Edge 3: (C,E) cost=2
Edge 4: Random pilih (E,D) cost=8

╔═════════════════════════════════════════════════════════════════════╗
║  Tree Edges: [(A,B,4), (B,C,4), (C,E,2), (E,D,8)]                   ║
║  TOTAL COST = 4 + 4 + 2 + 8 = 18                                    ║
║  Degree: A=1, B=2, C=2, D=1, E=2 (semua ≤ 3 )                       ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

## RINGKASAN ITERASI 1

```
┏━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃  Semut  ┃            Tree Edges                ┃    Cost     ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    1    ┃  (A,B), (B,C), (C,E), (C,D)         ┃     16      ┃ =>THE BEST!
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    2    ┃  (A,E), (E,C), (A,B), (C,D)         ┃     19      ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    3    ┃  (A,B), (B,C), (C,E), (E,D)         ┃     18      ┃
┗━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━┛

🏆 BEST = SEMUT 1 dengan Cost = 16

🏆 BEST = SEMUT 1 dengan Cost = 16
```

---

# BAGIAN 5: UPDATE FEROMON

## 5.1 Evaporasi: τ = 0.5 × τ

```
Semua edge: τ = 0.5 × 1.0 = 0.500
```

## 5.2 Deposit

```
Semut 1 (cost=16): Δτ₁ = 100/16 = 6.250
   → deposit pada (A,B), (B,C), (C,E), (C,D)

Semut 2 (cost=19): Δτ₂ = 100/19 = 5.263
   → deposit pada (A,E), (E,C), (A,B), (C,D)

Semut 3 (cost=18): Δτ₃ = 100/18 = 5.556
   → deposit pada (A,B), (B,C), (C,E), (E,D)
```

## 5.3 Feromon Final

```
┏━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━┓
┃   Edge    ┃              Perhitungan       ┃   Final  ┃
┣━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━┫
┃  (A,B)    ┃  0.50 + 6.250 + 5.263 + 5.556   ┃  17.569 ┃ =>TERTINGGI
┃  (C,E)    ┃  0.50 + 6.250 + 5.263 + 5.556   ┃  17.569 ┃ =>TERTINGGI
┃  (B,C)    ┃  0.50 + 6.250 + 5.556           ┃  12.306 ┃
┃  (C,D)    ┃  0.50 + 6.250 + 5.263           ┃  12.013 ┃
┃  (D,E)    ┃  0.50 + 5.556                   ┃   6.056 ┃
┃  (A,E)    ┃  0.50 + 5.263                   ┃   5.763 ┃
┃  (A,C)    ┃  0.50 + 0                       ┃   0.500 ┃
┃  (B,D)    ┃  0.50 + 0                       ┃   0.500 ┃
┃  (B,E)    ┃  0.50 + 0                       ┃   0.500 ┃
┃  (A,D)    ┃  0.50 + 0                       ┃   0.500 ┃
┗━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━┛

⭐ Edge (A,B) dan (C,E) paling tinggi karena digunakan SEMUA SEMUT!
```

---

# BAGIAN 6: EFEK FEROMON PADA ITERASI 2

## Perubahan Probabilitas dari A:

```
┏━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃  Edge   ┃   ITERASI 1       ┃   ITERASI 2       ┃  Perubahan  ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,B)  ┃      60.39%       ┃      89.56%       ┃   +29.17%   ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,C)  ┃      11.88%       ┃       0.51%       ┃   -11.37%   ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,D)  ┃       8.02%       ┃       0.34%       ┃    -7.68%   ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,E)  ┃      19.71%       ┃       9.59%       ┃   -10.12%   ┃
┗━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━┛

🔺 (A,B): semakin DIPERKUAT → hampir pasti dipilih
🔻 (A,C), (A,D): semakin DILEMAHKAN → hampir tidak mungkin dipilih

🔺 (A,B): semakin DIPERKUAT → hampir pasti dipilih
🔻 (A,C), (A,D): semakin DILEMAHKAN → hampir tidak mungkin dipilih
```

---

# BAGIAN 7: HASIL AKHIR

```
╔═════════════════════════════════════════════════════════════════════╗
║                                                                     ║
║           HASIL AKHIR ACO-DCMST (d_max = 3)                         ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║     (A)════4════(B)════4════(C)════6════(D)                         ║
║                              ║                                      ║
║                              2                                      ║
║                              ║                                      ║
║                             (E)                                     ║
║                                                                     ║
║  EDGES: (A,B)=4, (B,C)=4, (C,E)=2, (C,D)=6                          ║
║  TOTAL COST = 16                                                    ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║  DEGREE CHECK:                                                      ║
║  • A=1, B=2, C=3, D=1, E=1  (semua ≤ 3 )                            ║
╠═════════════════════════════════════════════════════════════════════╣
║  PERBANDINGAN:                                                      ║
║  • MST Optimal = 16 (memenuhi d_max=3)                              ║
║  • DCMST (ACO) = 16 (memenuhi d_max=3)                              ║
║  • Gap = 0% → OK! OPTIMAL!                                           ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 8: KESIMPULAN

```
╔═════════════════════════════════════════════════════════════════════╗
║                        📋 KESIMPULAN 📋                            ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  ACO-DCMST menggabungkan:                                           ║
║  • ACO: feromon + probabilistic selection                           ║
║  • Prim's: tree growing, always connected                           ║
║  • Degree Constraint: deg[v] < d_max sebelum add edge               ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  HASIL (3 semut, 1 iterasi):                                        ║
║  • Semut 1: Cost 16 =>BEST!                                         ║
║  • Semut 2: Cost 19                                                 ║
║  • Semut 3: Cost 18                                                 ║
║                                                                     ║
║  Dengan d_max = 3:                                                  ║
║  • Semua vertex punya ruang untuk koneksi                           ║
║  • Tidak ada edge yang di-block                                     ║
║  • MST murni sudah memenuhi constraint                              ║
║  • ACO menemukan solusi OPTIMAL!                                    ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

**END OF DOCUMENT**
