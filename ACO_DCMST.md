# 🐜 ACO untuk Degree Constrained Minimum Spanning Tree (DCMST)
## Panduan Lengkap dengan Perhitungan Step-by-Step

---

# BAGIAN 1: PENGANTAR ANT COLONY OPTIMIZATION (ACO)

## 1.1 Apa itu ACO?

**Ant Colony Optimization (ACO)** adalah algoritma metaheuristik yang terinspirasi dari perilaku semut dalam mencari makanan. Semut di alam nyata menemukan jalur terpendek ke sumber makanan melalui komunikasi tidak langsung menggunakan **feromon** (zat kimia).

### Prinsip Dasar ACO:
```
┌─────────────────────────────────────────────────────────────────────┐
│  PERILAKU SEMUT DI ALAM:                                            │
│                                                                     │
│  1. Semut meninggalkan feromon di jalur yang dilalui                │
│  2. Semut lain cenderung mengikuti jalur dengan feromon lebih kuat  │
│  3. Feromon menguap seiring waktu (evaporation)                     │
│  4. Jalur pendek → lebih sering dilalui → feromon lebih kuat        │
│  5. Jalur panjang → jarang dilalui → feromon menguap                │
│                                                                     │
│  Hasil: Secara kolektif, semut menemukan jalur terpendek!           │
└─────────────────────────────────────────────────────────────────────┘
```

### Aplikasi ACO dalam Optimasi:
- **TSP (Travelling Salesman Problem)**: Mencari tour terpendek
- **MST (Minimum Spanning Tree)**: Mencari pohon rentang minimum
- **DCMST**: MST dengan batasan degree pada setiap vertex

---

## 1.2 Perbedaan ACO-TSP vs ACO-MST

| Aspek | ACO-TSP | ACO-MST (Prim-based) |
|-------|---------|----------------------|
| **Tujuan** | Tour terpendek (cycle) | Tree dengan total cost minimum |
| **Struktur** | Hamiltonian cycle | Tree (n-1 edges) |
| **Konstruksi** | Sequential path | Tree growing |
| **Valid** | Kembali ke kota asal | Semua vertex terhubung, no cycle |
| **Constraint** | Kunjungi semua sekali | Degree constraint |

---

## 1.3 DCMST (Degree Constrained MST)

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
│  • V = {A, B, C, D, E} : Himpunan vertex (5 kota)                  │
│  • E = himpunan edge dengan bobot d[i][j]                          │
│  • τ[i][j] : Feromon pada edge (i,j)                               │
│  • η[i][j] : Heuristik = 1/d[i][j] (inverse jarak)                 │
│  • T : Himpunan vertex yang sudah ada di tree                      │
│  • deg[v] : Degree saat ini dari vertex v                          │
├─────────────────────────────────────────────────────────────────────┤
│  PARAMETER:                                                         │
│  • α = 1   : Bobot pengaruh feromon                                │
│  • β = 2   : Bobot pengaruh heuristik                              │
│  • ρ = 0.5 : Tingkat evaporasi feromon (50%)                       │
│  • Q = 100 : Konstanta deposit feromon                             │
│  • τ₀ = 1.0 : Feromon awal                                         │
│  • d_max = 2 : Degree maksimum per vertex                          │
│  • n_ants = 3 : Jumlah semut                                       │
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

## 3.1 Input: Graf 5 Kota

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

# BAGIAN 4: ITERASI 1 - KONSTRUKSI SOLUSI

## 4.1 SEMUT 1: Konstruksi Tree

### Step 1: Inisialisasi
```
┌─────────────────────────────────────────────────────────────────────┐
│  SEMUT 1 - START                                                    │
├─────────────────────────────────────────────────────────────────────┤
│  • Start vertex: A (central vertex)                                │
│  • T (tree vertices) = {A}                                         │
│  • Tree edges = []                                                 │
│  • deg[A]=0, deg[B]=0, deg[C]=0, deg[D]=0, deg[E]=0               │
│  • d_max = 2 (setiap vertex maksimal 2 koneksi)                   │
│  • Target: 4 edges (n-1 = 5-1 = 4)                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 2: Pilih Edge Pertama (dari A)
```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 1 - EDGE 1                                                 │
├─────────────────────────────────────────────────────────────────────┤
│  T = {A}, mencari edge ke vertex di luar T                         │
│  Candidates: (A,B), (A,C), (A,D), (A,E)                            │
│                                                                     │
│  Semua feasible karena deg[A]=0 < 2 dan deg[*]=0 < 2              │
└─────────────────────────────────────────────────────────────────────┘

Hitung nilai τ^α × η^β untuk setiap candidate:

• (A,B): τ[A][B]^1 × η[A][B]^2 = 1.0^1 × 0.250^2 = 1 × 0.0625 = 0.0625
• (A,C): τ[A][C]^1 × η[A][C]^2 = 1.0^1 × 0.111^2 = 1 × 0.0123 = 0.0123
• (A,D): τ[A][D]^1 × η[A][D]^2 = 1.0^1 × 0.091^2 = 1 × 0.0083 = 0.0083
• (A,E): τ[A][E]^1 × η[A][E]^2 = 1.0^1 × 0.143^2 = 1 × 0.0204 = 0.0204

Total = 0.0625 + 0.0123 + 0.0083 + 0.0204 = 0.1035

PROBABILITAS:
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,B)  │   0.0625   │ 0.0625/0.1035 = 60.4%  │
│ (A,C)  │   0.0123   │ 0.0123/0.1035 = 11.9%  │
│ (A,D)  │   0.0083   │ 0.0083/0.1035 =  8.0%  │
│ (A,E)  │   0.0204   │ 0.0204/0.1035 = 19.7%  │
└────────┴────────────┴─────────────────────────┘

→ PILIH (A,B) dengan probabilitas tertinggi (60.4%)
  (Atau gunakan roulette wheel selection)
```

### Update setelah Edge 1:
```
┌─────────────────────────────────────────────────────────────────────┐
│  SETELAH PILIH (A,B):                                               │
├─────────────────────────────────────────────────────────────────────┤
│  • T = {A, B}                                                      │
│  • Tree edges = [(A,B,4)]                                          │
│  • deg[A]=1, deg[B]=1, deg[C]=0, deg[D]=0, deg[E]=0               │
│  • Current cost = 4                                                │
└─────────────────────────────────────────────────────────────────────┘

Graf saat ini:
    A ═══4═══ B
```

### Step 3: Pilih Edge Kedua
```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 1 - EDGE 2                                                 │
├─────────────────────────────────────────────────────────────────────┤
│  T = {A, B}, mencari edge ke vertex di luar T                      │
│  Candidates dari A: (A,C), (A,D), (A,E) [deg[A]=1 < 2 ✓]          │
│  Candidates dari B: (B,C), (B,D), (B,E) [deg[B]=1 < 2 ✓]          │
│                                                                     │
│  Total candidates: (A,C), (A,D), (A,E), (B,C), (B,D), (B,E)        │
└─────────────────────────────────────────────────────────────────────┘

Hitung nilai τ^α × η^β:

Dari A:
• (A,C): 1.0^1 × 0.111^2 = 0.0123
• (A,D): 1.0^1 × 0.091^2 = 0.0083
• (A,E): 1.0^1 × 0.143^2 = 0.0204

Dari B:
• (B,C): 1.0^1 × 0.250^2 = 0.0625
• (B,D): 1.0^1 × 0.077^2 = 0.0059
• (B,E): 1.0^1 × 0.100^2 = 0.0100

Total = 0.0123 + 0.0083 + 0.0204 + 0.0625 + 0.0059 + 0.0100 = 0.1194

PROBABILITAS:
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,C)  │   0.0123   │ 0.0123/0.1194 = 10.3%  │
│ (A,D)  │   0.0083   │ 0.0083/0.1194 =  7.0%  │
│ (A,E)  │   0.0204   │ 0.0204/0.1194 = 17.1%  │
│ (B,C)  │   0.0625   │ 0.0625/0.1194 = 52.3%  │ ← Tertinggi!
│ (B,D)  │   0.0059   │ 0.0059/0.1194 =  4.9%  │
│ (B,E)  │   0.0100   │ 0.0100/0.1194 =  8.4%  │
└────────┴────────────┴─────────────────────────┘

→ PILIH (B,C) dengan probabilitas tertinggi (52.3%)
```

### Update setelah Edge 2:
```
┌─────────────────────────────────────────────────────────────────────┐
│  SETELAH PILIH (B,C):                                               │
├─────────────────────────────────────────────────────────────────────┤
│  • T = {A, B, C}                                                   │
│  • Tree edges = [(A,B,4), (B,C,4)]                                 │
│  • deg[A]=1, deg[B]=2, deg[C]=1, deg[D]=0, deg[E]=0               │
│  • Current cost = 8                                                │
│                                                                     │
│  ⚠️ PERHATIAN: deg[B] = 2 = d_max                                  │
│  → B sudah tidak bisa menerima edge lagi!                          │
└─────────────────────────────────────────────────────────────────────┘

Graf saat ini:
    A ═══4═══ B ═══4═══ C
```

### Step 4: Pilih Edge Ketiga
```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 1 - EDGE 3                                                 │
├─────────────────────────────────────────────────────────────────────┤
│  T = {A, B, C}, mencari edge ke {D, E}                             │
│                                                                     │
│  Dari A (deg[A]=1 < 2): (A,D), (A,E) ✓                             │
│  Dari B (deg[B]=2 = d_max): ❌ TIDAK BISA menambah edge            │
│  Dari C (deg[C]=1 < 2): (C,D), (C,E) ✓                             │
│                                                                     │
│  Valid candidates: (A,D), (A,E), (C,D), (C,E)                      │
└─────────────────────────────────────────────────────────────────────┘

Hitung nilai τ^α × η^β:

• (A,D): 1.0^1 × 0.091^2 = 0.0083
• (A,E): 1.0^1 × 0.143^2 = 0.0204
• (C,D): 1.0^1 × 0.167^2 = 0.0279
• (C,E): 1.0^1 × 0.500^2 = 0.2500  ← JAUH LEBIH TINGGI!

Total = 0.0083 + 0.0204 + 0.0279 + 0.2500 = 0.3066

PROBABILITAS:
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,D)  │   0.0083   │ 0.0083/0.3066 =  2.7%  │
│ (A,E)  │   0.0204   │ 0.0204/0.3066 =  6.7%  │
│ (C,D)  │   0.0279   │ 0.0279/0.3066 =  9.1%  │
│ (C,E)  │   0.2500   │ 0.2500/0.3066 = 81.5%  │ ← SANGAT TINGGI!
└────────┴────────────┴─────────────────────────┘

→ PILIH (C,E) dengan probabilitas 81.5%
  (Edge terpendek = 2, sangat menarik!)
```

### Update setelah Edge 3:
```
┌─────────────────────────────────────────────────────────────────────┐
│  SETELAH PILIH (C,E):                                               │
├─────────────────────────────────────────────────────────────────────┤
│  • T = {A, B, C, E}                                                │
│  • Tree edges = [(A,B,4), (B,C,4), (C,E,2)]                        │
│  • deg[A]=1, deg[B]=2, deg[C]=2, deg[D]=0, deg[E]=1               │
│  • Current cost = 10                                               │
│                                                                     │
│  ⚠️ deg[B] = 2, deg[C] = 2 (keduanya sudah maksimal)              │
└─────────────────────────────────────────────────────────────────────┘

Graf saat ini:
    A ═══4═══ B ═══4═══ C
                        ║
                        2
                        ║
                        E
```

### Step 5: Pilih Edge Keempat (Terakhir)
```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 1 - EDGE 4 (FINAL)                                         │
├─────────────────────────────────────────────────────────────────────┤
│  T = {A, B, C, E}, mencari edge ke {D}                             │
│                                                                     │
│  Dari A (deg[A]=1 < 2): (A,D) ✓                                    │
│  Dari B (deg[B]=2): ❌                                              │
│  Dari C (deg[C]=2): ❌                                              │
│  Dari E (deg[E]=1 < 2): (E,D) ✓                                    │
│                                                                     │
│  Valid candidates: (A,D), (E,D)                                    │
└─────────────────────────────────────────────────────────────────────┘

Hitung nilai τ^α × η^β:

• (A,D): 1.0^1 × 0.091^2 = 0.0083    (d[A][D] = 11)
• (E,D): 1.0^1 × 0.125^2 = 0.0156    (d[E][D] = 8)

Total = 0.0083 + 0.0156 = 0.0239

PROBABILITAS:
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,D)  │   0.0083   │ 0.0083/0.0239 = 34.7%  │
│ (E,D)  │   0.0156   │ 0.0156/0.0239 = 65.3%  │ ← Lebih tinggi
└────────┴────────────┴─────────────────────────┘

→ PILIH (E,D) dengan probabilitas 65.3%
```

### Hasil Akhir Semut 1:
```
╔═════════════════════════════════════════════════════════════════════╗
║  SEMUT 1 - HASIL AKHIR                                              ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  Tree edges: [(A,B,4), (B,C,4), (C,E,2), (E,D,8)]                  ║
║                                                                     ║
║  TOTAL COST = 4 + 4 + 2 + 8 = 18                                   ║
║                                                                     ║
║  Degree Check:                                                      ║
║  • deg[A] = 1 ≤ 2 ✓                                                ║
║  • deg[B] = 2 ≤ 2 ✓                                                ║
║  • deg[C] = 2 ≤ 2 ✓                                                ║
║  • deg[D] = 1 ≤ 2 ✓                                                ║
║  • deg[E] = 2 ≤ 2 ✓                                                ║
║                                                                     ║
║  ✅ VALID DCMST SOLUTION!                                           ║
╚═════════════════════════════════════════════════════════════════════╝

Graf Akhir Semut 1:
    A ═══4═══ B ═══4═══ C
                        ║
                        2
                        ║
              D ═══8═══ E
```

---

## 4.2 SEMUT 2: Konstruksi Tree (Alternatif Path)

### Step 1-2: Start dan Edge Pertama
```
┌─────────────────────────────────────────────────────────────────────┐
│  SEMUT 2 - Menggunakan roulette wheel, mungkin pilih berbeda       │
├─────────────────────────────────────────────────────────────────────┤
│  Misalkan roulette jatuh di (A,E) (prob 19.7%)                     │
│                                                                     │
│  T = {A, E}                                                        │
│  Tree edges = [(A,E,7)]                                            │
│  deg[A]=1, deg[E]=1                                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 3: Pilih Edge Kedua
```
Candidates dari A: (A,B), (A,C), (A,D)
Candidates dari E: (E,B), (E,C), (E,D)

Perhitungan:
• (A,B): 0.0625
• (A,C): 0.0123
• (A,D): 0.0083
• (E,B): 1.0 × 0.100^2 = 0.0100
• (E,C): 1.0 × 0.500^2 = 0.2500  ← Tertinggi!
• (E,D): 1.0 × 0.125^2 = 0.0156

→ Pilih (E,C) dengan nilai tertinggi
  T = {A, E, C}
  deg[E] = 2 (maksimal)
```

### Step 4: Pilih Edge Ketiga
```
Dari A (deg=1): (A,B), (A,D)
Dari E (deg=2): ❌
Dari C (deg=1): (C,B), (C,D)

Perhitungan:
• (A,B): 0.0625
• (A,D): 0.0083
• (C,B): 0.0625
• (C,D): 0.0279

→ Pilih (A,B) atau (C,B) (sama-sama 0.0625)
  Misalkan pilih (C,B)
  T = {A, E, C, B}
  deg[C] = 2 (maksimal)
```

### Step 5: Pilih Edge Keempat
```
Dari A (deg=1): (A,D) ✓
Dari B (deg=1): (B,D) ✓

• (A,D): 1.0 × 0.091^2 = 0.0083
• (B,D): 1.0 × 0.077^2 = 0.0059

→ Pilih (A,D)
```

### Hasil Akhir Semut 2:
```
╔═════════════════════════════════════════════════════════════════════╗
║  SEMUT 2 - HASIL AKHIR                                              ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  Tree edges: [(A,E,7), (E,C,2), (C,B,4), (A,D,11)]                 ║
║                                                                     ║
║  TOTAL COST = 7 + 2 + 4 + 11 = 24                                  ║
║                                                                     ║
║  ✅ VALID DCMST (degree ≤ 2)                                        ║
╚═════════════════════════════════════════════════════════════════════╝

Graf Akhir Semut 2:
    A ═══7═══ E ═══2═══ C ═══4═══ B
    ║
   11
    ║
    D
```

---

## 4.3 SEMUT 3: Konstruksi Tree

```
┌─────────────────────────────────────────────────────────────────────┐
│  SEMUT 3 - Path Alternatif Lain                                     │
├─────────────────────────────────────────────────────────────────────┤
│  Misalkan urutan pilihan berbeda via roulette:                      │
│                                                                     │
│  1. (A,B) = 4                                                      │
│  2. (B,C) = 4   [deg[B] = 2]                                       │
│  3. (A,E) = 7   [deg[A] = 2]                                       │
│  4. (C,D) = 6   [deg[C] = 2]                                       │
│                                                                     │
│  Tree: [(A,B,4), (B,C,4), (A,E,7), (C,D,6)]                        │
│  TOTAL COST = 4 + 4 + 7 + 6 = 21                                   │
╚═════════════════════════════════════════════════════════════════════╝

Graf Akhir Semut 3:
    E ═══7═══ A ═══4═══ B ═══4═══ C ═══6═══ D
```

---

## 4.4 Ringkasan Iterasi 1

```
╔═════════════════════════════════════════════════════════════════════╗
║  RINGKASAN ITERASI 1 - SEMUA SEMUT                                  ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  ┌─────────┬────────────────────────────────────┬───────────────┐  ║
║  │  Semut  │           Tree Edges               │     Cost      │  ║
║  ├─────────┼────────────────────────────────────┼───────────────┤  ║
║  │    1    │  (A,B), (B,C), (C,E), (E,D)       │      18       │  ║
║  │    2    │  (A,E), (E,C), (C,B), (A,D)       │      24       │  ║
║  │    3    │  (A,B), (B,C), (A,E), (C,D)       │      21       │  ║
║  └─────────┴────────────────────────────────────┴───────────────┘  ║
║                                                                     ║
║  🏆 BEST SOLUTION: Semut 1 dengan Cost = 18                        ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 5: UPDATE FEROMON

## 5.1 Langkah 1: Evaporasi

```
┌─────────────────────────────────────────────────────────────────────┐
│  EVAPORASI: τ_new = (1 - ρ) × τ_old = 0.5 × τ_old                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Semua feromon dikurangi 50%:                                      │
│  τ[i][j] = 0.5 × 1.0 = 0.5 untuk semua edge                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Pheromone setelah evaporasi:
┌─────┬──────┬──────┬──────┬──────┬──────┐
│     │  A   │  B   │  C   │  D   │  E   │
├─────┼──────┼──────┼──────┼──────┼──────┤
│  A  │  -   │ 0.50 │ 0.50 │ 0.50 │ 0.50 │
│  B  │ 0.50 │  -   │ 0.50 │ 0.50 │ 0.50 │
│  C  │ 0.50 │ 0.50 │  -   │ 0.50 │ 0.50 │
│  D  │ 0.50 │ 0.50 │ 0.50 │  -   │ 0.50 │
│  E  │ 0.50 │ 0.50 │ 0.50 │ 0.50 │  -   │
└─────┴──────┴──────┴──────┴──────┴──────┘
```

## 5.2 Langkah 2: Deposit Feromon

```
┌─────────────────────────────────────────────────────────────────────┐
│  DEPOSIT: Δτ = Q / Cost                                             │
├─────────────────────────────────────────────────────────────────────┤

SEMUT 1 (Cost = 18):
Δτ₁ = 100 / 18 = 5.556

Edges: (A,B), (B,C), (C,E), (E,D)
• τ[A][B] += 5.556
• τ[B][C] += 5.556
• τ[C][E] += 5.556
• τ[E][D] += 5.556

─────────────────────────────────────────────────────────────────────

SEMUT 2 (Cost = 24):
Δτ₂ = 100 / 24 = 4.167

Edges: (A,E), (E,C), (C,B), (A,D)
• τ[A][E] += 4.167
• τ[E][C] += 4.167  (sama dengan τ[C][E])
• τ[C][B] += 4.167  (sama dengan τ[B][C])
• τ[A][D] += 4.167

─────────────────────────────────────────────────────────────────────

SEMUT 3 (Cost = 21):
Δτ₃ = 100 / 21 = 4.762

Edges: (A,B), (B,C), (A,E), (C,D)
• τ[A][B] += 4.762
• τ[B][C] += 4.762
• τ[A][E] += 4.762
• τ[C][D] += 4.762

└─────────────────────────────────────────────────────────────────────┘
```

## 5.3 Hasil Update Feromon

```
┌─────────────────────────────────────────────────────────────────────┐
│  PERHITUNGAN FINAL FEROMON:                                         │
├─────────────────────────────────────────────────────────────────────┤

τ[A][B] = 0.50 + 5.556 + 4.762 = 10.818  (digunakan Semut 1 & 3)
τ[A][C] = 0.50 + 0 = 0.50               (tidak digunakan)
τ[A][D] = 0.50 + 4.167 = 4.667          (digunakan Semut 2)
τ[A][E] = 0.50 + 4.167 + 4.762 = 9.429  (digunakan Semut 2 & 3)
τ[B][C] = 0.50 + 5.556 + 4.167 + 4.762 = 14.985 (digunakan semua!)
τ[B][D] = 0.50 + 0 = 0.50               (tidak digunakan)
τ[B][E] = 0.50 + 0 = 0.50               (tidak digunakan)
τ[C][D] = 0.50 + 4.762 = 5.262          (digunakan Semut 3)
τ[C][E] = 0.50 + 5.556 + 4.167 = 10.223 (digunakan Semut 1 & 2)
τ[D][E] = 0.50 + 5.556 = 6.056          (digunakan Semut 1)

└─────────────────────────────────────────────────────────────────────┘

Updated Pheromone Matrix τ[i][j] setelah Iterasi 1:
┌─────┬────────┬────────┬────────┬────────┬────────┐
│     │   A    │   B    │   C    │   D    │   E    │
├─────┼────────┼────────┼────────┼────────┼────────┤
│  A  │   -    │ 10.818 │  0.50  │  4.667 │  9.429 │
│  B  │ 10.818 │   -    │ 14.985 │  0.50  │  0.50  │
│  C  │  0.50  │ 14.985 │   -    │  5.262 │ 10.223 │
│  D  │  4.667 │  0.50  │  5.262 │   -    │  6.056 │
│  E  │  9.429 │  0.50  │ 10.223 │  6.056 │   -    │
└─────┴────────┴────────┴────────┴────────┴────────┘

OBSERVASI:
• τ[B][C] = 14.985 (TERTINGGI!) - digunakan oleh semua semut
• τ[A][B] = 10.818 (tinggi) - jalur bagus
• τ[C][E] = 10.223 (tinggi) - edge terpendek, sering dipilih
• τ[A][C] = 0.50 (rendah) - tidak pernah digunakan
```

---

# BAGIAN 6: ITERASI 2 (SINGKAT)

## 6.1 Efek Feromon Baru pada Probabilitas

```
┌─────────────────────────────────────────────────────────────────────┐
│  ITERASI 2 - SEMUT BARU DENGAN FEROMON UPDATED                      │
├─────────────────────────────────────────────────────────────────────┤

Dari A, probabilitas edge pertama:

• (A,B): τ^1 × η^2 = 10.818 × 0.0625 = 0.676
• (A,C): τ^1 × η^2 = 0.50 × 0.0123 = 0.006
• (A,D): τ^1 × η^2 = 4.667 × 0.0083 = 0.039
• (A,E): τ^1 × η^2 = 9.429 × 0.0204 = 0.192

Total = 0.913

PROBABILITAS BARU:
┌────────┬────────────┬─────────────────────────┐
│  Edge  │   Value    │     Probability         │
├────────┼────────────┼─────────────────────────┤
│ (A,B)  │   0.676    │ 0.676/0.913 = 74.0%    │ ← NAIK dari 60.4%!
│ (A,C)  │   0.006    │ 0.006/0.913 =  0.7%    │ ← TURUN dari 11.9%
│ (A,D)  │   0.039    │ 0.039/0.913 =  4.3%    │ ← TURUN dari 8.0%
│ (A,E)  │   0.192    │ 0.192/0.913 = 21.0%    │ ← NAIK dari 19.7%
└────────┴────────────┴─────────────────────────┘

→ Feromon membuat edge bagus (A,B) dan (A,E) lebih mungkin dipilih!
→ Edge buruk (A,C) hampir tidak mungkin dipilih

└─────────────────────────────────────────────────────────────────────┘
```

---

# BAGIAN 7: HASIL AKHIR DCMST

## 7.1 Setelah Beberapa Iterasi

```
╔═════════════════════════════════════════════════════════════════════╗
║                      HASIL AKHIR ACO-DCMST                          ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  🏆 BEST DCMST FOUND:                                               ║
║                                                                     ║
║  Edges: (A,B), (B,C), (C,E), (E,D)                                 ║
║  Weights: 4 + 4 + 2 + 8 = 18                                       ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐   ║
║  │                                                             │   ║
║  │     A ═══4═══ B ═══4═══ C                                  │   ║
║  │                         ║                                  │   ║
║  │                         2                                  │   ║
║  │                         ║                                  │   ║
║  │               D ═══8═══ E                                  │   ║
║  │                                                             │   ║
║  └─────────────────────────────────────────────────────────────┘   ║
║                                                                     ║
║  DEGREE VERIFICATION:                                               ║
║  ┌────────┬─────────┬────────────┐                                 ║
║  │ Vertex │ Degree  │  Status    │                                 ║
║  ├────────┼─────────┼────────────┤                                 ║
║  │   A    │    1    │  ≤ 2 ✓    │                                 ║
║  │   B    │    2    │  ≤ 2 ✓    │                                 ║
║  │   C    │    2    │  ≤ 2 ✓    │                                 ║
║  │   D    │    1    │  ≤ 2 ✓    │                                 ║
║  │   E    │    2    │  ≤ 2 ✓    │                                 ║
║  └────────┴─────────┴────────────┘                                 ║
║                                                                     ║
║  ✅ ALL DEGREE CONSTRAINTS SATISFIED!                               ║
║  ✅ TREE IS CONNECTED (4 edges for 5 vertices)                      ║
║  ✅ NO CYCLES                                                        ║
║                                                                     ║
║  📊 TOTAL COST: 18                                                  ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

## 7.2 Perbandingan dengan MST Tanpa Degree Constraint

```
┌─────────────────────────────────────────────────────────────────────┐
│  PERBANDINGAN: MST vs DCMST                                         │
├─────────────────────────────────────────────────────────────────────┤

MST OPTIMAL (tanpa degree constraint):
• Edges: (C,E)=2, (A,B)=4, (B,C)=4, (C,D)=6
• Total: 2 + 4 + 4 + 6 = 16
• deg[C] = 3 ← MELANGGAR degree constraint!

    A ═══4═══ B ═══4═══ C ═══6═══ D
                        ║
                        2
                        ║
                        E

─────────────────────────────────────────────────────────────────────

DCMST OPTIMAL (dengan degree ≤ 2):
• Edges: (C,E)=2, (A,B)=4, (B,C)=4, (E,D)=8
• Total: 2 + 4 + 4 + 8 = 18
• Semua degree ≤ 2 ✓

    A ═══4═══ B ═══4═══ C
                        ║
                        2
                        ║
              D ═══8═══ E

─────────────────────────────────────────────────────────────────────

KESIMPULAN:
• MST = 16, DCMST = 18
• Gap = 18 - 16 = 2 (12.5% lebih mahal)
• Trade-off: reliability (degree constraint) vs cost

└─────────────────────────────────────────────────────────────────────┘
```

---

# BAGIAN 8: RINGKASAN RUMUS

```
╔═════════════════════════════════════════════════════════════════════╗
║                    RINGKASAN SEMUA RUMUS ACO-DCMST                  ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  1. HEURISTIK (Visibility):                                        ║
║     ┌─────────────────────────┐                                    ║
║     │        1                │                                    ║
║     │ η[i][j] = ────────      │                                    ║
║     │          d[i][j]        │                                    ║
║     └─────────────────────────┘                                    ║
║                                                                     ║
║  2. PROBABILITAS PEMILIHAN EDGE:                                   ║
║     ┌─────────────────────────────────────────────────────────┐    ║
║     │           [τ[u][v]]^α × [η[u][v]]^β × feasible(u,v)     │    ║
║     │ P(u,v) = ─────────────────────────────────────────────  │    ║
║     │          Σ [τ[u][w]]^α × [η[u][w]]^β × feasible(u,w)    │    ║
║     │          w                                               │    ║
║     └─────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║     feasible(u,v) = 1 jika deg[u] < d_max AND deg[v] < d_max       ║
║                   = 0 otherwise                                     ║
║                                                                     ║
║  3. EVAPORASI FEROMON:                                             ║
║     ┌─────────────────────────────────────┐                        ║
║     │ τ[i][j] ← (1 - ρ) × τ[i][j]         │                        ║
║     └─────────────────────────────────────┘                        ║
║                                                                     ║
║  4. DEPOSIT FEROMON:                                               ║
║     ┌─────────────────────────────────────┐                        ║
║     │         Q                           │                        ║
║     │ Δτ = ────────                       │                        ║
║     │      Cost(T)                        │                        ║
║     │                                     │                        ║
║     │ τ[i][j] ← τ[i][j] + Σ Δτₖ          │                        ║
║     │                      k              │                        ║
║     └─────────────────────────────────────┘                        ║
║                                                                     ║
║  5. UPDATE FEROMON LENGKAP:                                        ║
║     ┌─────────────────────────────────────────────────────────┐    ║
║     │                          m                               │    ║
║     │ τ[i][j] ← (1-ρ) × τ[i][j] + Σ Δτₖ                       │    ║
║     │                          k=1                             │    ║
║     │                                                          │    ║
║     │ m = jumlah semut yang menggunakan edge (i,j)            │    ║
║     └─────────────────────────────────────────────────────────┘    ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 9: FLOWCHART ALGORITMA

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLOWCHART ACO-DCMST                              │
└─────────────────────────────────────────────────────────────────────┘

                           ┌───────────┐
                           │   START   │
                           └─────┬─────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  Initialize:           │
                    │  • τ[i][j] = τ₀        │
                    │  • η[i][j] = 1/d[i][j] │
                    │  • best_cost = ∞       │
                    └───────────┬────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────┐
              │  FOR iter = 1 TO max_iterations     │
              └────────────────┬────────────────────┘
                               │
                               ▼
              ┌────────────────────────────────────┐
              │  FOR ant = 1 TO n_ants             │
              └───────────────┬────────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  Initialize:                     │
               │  • T = {start_vertex}            │
               │  • tree_edges = []               │
               │  • deg[v] = 0 for all v          │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  WHILE |tree_edges| < n-1        │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  Get candidates:                 │
               │  {(u,v): u∈T, v∉T,              │
               │   deg[u]<d_max, deg[v]<d_max}   │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  Calculate P(u,v) for all       │
               │  candidates using formula       │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  Select edge (u,v) using        │
               │  roulette wheel selection       │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  Add edge to tree:              │
               │  • tree_edges.add((u,v))        │
               │  • T.add(v)                     │
               │  • deg[u]++, deg[v]++           │
               └──────────────┬───────────────────┘
                              │
                              ▼
               ┌──────────────────────────────────┐
               │  |tree_edges| = n-1?            │
               └──────────────┬───────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │ NO               │ YES
                    ▼                   ▼
             [loop back]        ┌──────────────────┐
                                │  Calculate cost  │
                                │  Update best if  │
                                │  cost < best_cost│
                                └────────┬─────────┘
                                         │
                                         ▼
                           ┌─────────────────────────┐
                           │  All ants done?         │
                           └─────────────────────────┘
                                    │
                         ┌──────────┴──────────┐
                         │ NO                 │ YES
                         ▼                     ▼
                   [next ant]        ┌─────────────────────┐
                                     │  UPDATE PHEROMONE:  │
                                     │  1. Evaporation     │
                                     │  2. Deposit         │
                                     └──────────┬──────────┘
                                                │
                                                ▼
                                   ┌────────────────────────┐
                                   │  max_iterations done?  │
                                   └────────────────────────┘
                                            │
                                  ┌─────────┴─────────┐
                                  │ NO               │ YES
                                  ▼                   ▼
                           [next iter]        ┌──────────────┐
                                              │ Return best  │
                                              │ solution     │
                                              └──────┬───────┘
                                                     │
                                                     ▼
                                              ┌──────────┐
                                              │   END    │
                                              └──────────┘
```

---

# KESIMPULAN

```
╔═════════════════════════════════════════════════════════════════════╗
║                         KESIMPULAN                                   ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  ACO-DCMST adalah algoritma hybrid yang menggabungkan:              ║
║                                                                     ║
║  1. 🐜 ANT COLONY OPTIMIZATION:                                     ║
║     • Feromon untuk menyimpan "pengalaman" kolektif                 ║
║     • Probabilistic selection                                       ║
║     • Multiple agents (semut) exploring                             ║
║                                                                     ║
║  2. 🌳 PRIM'S MST APPROACH:                                         ║
║     • Tree growing dari central vertex                              ║
║     • Selalu maintain connected tree                                ║
║     • Greedy selection dengan ACO probability                       ║
║                                                                     ║
║  3. 🔒 DEGREE CONSTRAINT:                                           ║
║     • Setiap vertex maksimal d_max koneksi                          ║
║     • Feasibility check sebelum selection                           ║
║     • Menjamin solusi valid                                         ║
║                                                                     ║
║  HASIL CONTOH:                                                      ║
║  • Input: 5 kota, degree ≤ 2                                        ║
║  • MST optimal: 16 (melanggar degree)                               ║
║  • DCMST optimal: 18 (semua degree ≤ 2)                            ║
║  • ACO menemukan DCMST = 18 ✓                                       ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```
