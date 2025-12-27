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

```
╔═════════════════════════════════════════════════════════════════════╗
║  DCMST CONSTRAINT:                                                  ║
║                                                                     ║
║  Untuk setiap vertex v:   degree(v) ≤ d_max = 3                     ║
║  (Setiap vertex/kota maksimal memiliki 3 koneksi)                   ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 2: RUMUS-RUMUS ACO-DCMST

## Parameter: α=1, β=2, ρ=0.5, Q=100, τ₀=1.0, d_max=3

### RUMUS 1: HEURISTIK
```
         η[i][j] = 1 / d[i][j]
```

### RUMUS 2: PROBABILITAS
```
                [τ(u,v)]^α × [η(u,v)]^β × feasible(u,v)
    P(u,v) = ──────────────────────────────────────────────
              Σ [τ(u,w)]^α × [η(u,w)]^β × feasible(u,w)

    feasible(u,v) = 1 jika deg[u] < 3 DAN deg[v] < 3
                  = 0 otherwise
```

### RUMUS 3: EVAPORASI
```
    τ[i][j] ← 0.5 × τ[i][j]
```

### RUMUS 4: DEPOSIT
```
    Δτ = Q / Cost(T) = 100 / Cost(T)
```

---

# BAGIAN 3: INPUT (Case 3 semit, 5 kota)

## Distance Matrix (5 kota: A,B,C,D,E):

```
       A      B      C      D      E
A      -      4      9     11      7
B      4      -      4     13     10
C      9      4      -      6      2
D     11     13      6      -      8
E      7     10      2      8      -
```

## Heuristic η = 1/d:

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

# BAGIAN 4: ITERASI 1 - KONSTRUKSI SOLUSI

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
┃  Semut ┃            Tree Edges         ┃    Cost   ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    1   ┃  (A,B), (B,C), (C,E), (C,D)   ┃     16    ┃ =>BEST!
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    2   ┃  (A,E), (E,C), (A,B), (C,D)   ┃     19    ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃    3   ┃  (A,B), (B,C), (C,E), (E,D)   ┃     18    ┃
┗━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━┛

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
┃  Edge ┃   ITERASI 1    ┃   ITERASI 2    ┃  Perubahan┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,B)┃      60.39%    ┃      89.56%    ┃   +29.17% ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,C)┃      11.88%    ┃       0.51%    ┃   -11.37% ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,D)┃       8.02%    ┃       0.34%    ┃    -7.68% ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━┫
┃  (A,E)┃      19.71%    ┃       9.59%    ┃   -10.12% ┃
┗━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━┛

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
