# 🐜 ACO untuk Multi-Period Degree Constrained MST (MPDCMST)
## Panduan Lengkap dengan Perhitungan Step-by-Step
### Case Study: 30 Vertex dengan 3 Periode

---

# BAGIAN 1: PENGANTAR

## 1.1 Apa itu ACO (Ant Colony Optimization)?

**Ant Colony Optimization (ACO)** adalah algoritma metaheuristik yang terinspirasi dari perilaku semut dalam mencari makanan. Semut berkomunikasi secara tidak langsung melalui **feromon** (zat kimia) yang ditinggalkan di jalur yang dilalui.

```
╔═════════════════════════════════════════════════════════════════════╗
║  🐜 PRINSIP DASAR ACO:                                              ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  1. Semut meninggalkan FEROMON di jalur yang dilalui               ║
║                                                                     ║
║  2. Semut lain cenderung MENGIKUTI jalur dengan feromon lebih kuat ║
║                                                                     ║
║  3. Feromon MENGUAP seiring waktu (evaporation)                    ║
║                                                                     ║
║  4. Jalur PENDEK → lebih sering dilalui → feromon LEBIH KUAT       ║
║                                                                     ║
║  5. Jalur PANJANG → jarang dilalui → feromon MENGHILANG            ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║  HASIL: Secara kolektif, semut menemukan SOLUSI OPTIMAL!           ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

## 1.2 Apa itu MPDCMST?

**MPDCMST (Multi-Period Degree Constrained Minimum Spanning Tree)** adalah masalah mencari spanning tree dengan biaya minimum yang memenuhi:

```
╔═════════════════════════════════════════════════════════════════════╗
║                    MPDCMST CONSTRAINTS                              ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  1️⃣ DEGREE CONSTRAINT:                                              ║
║     Setiap vertex maksimal memiliki d_max koneksi                  ║
║     degree(v) ≤ d_max = 3                                          ║
║                                                                     ║
║  2️⃣ MULTI-PERIOD CONSTRAINT:                                        ║
║     Instalasi dibagi menjadi beberapa periode (3 periode)          ║
║     Setiap periode memiliki batas maksimum edge: MaxVT             ║
║                                                                     ║
║  3️⃣ HVT (High-priority Vertex per period T) CONSTRAINT:            ║
║     Vertex prioritas HARUS terhubung pada/sebelum periode tertentu ║
║     Contoh: HVT₁ = {2,3} → vertex 2,3 HARUS connected di periode 1 ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║  TUJUAN: Minimize total cost WHILE satisfying ALL constraints      ║
╚═════════════════════════════════════════════════════════════════════╝
```

### Ilustrasi Multi-Period Installation:

```
    PERIODE 1              PERIODE 2              PERIODE 3
    ─────────              ─────────              ─────────
    MaxVT₁ edge           MaxVT₂ edge           Sisa edge
    HVT₁ HARUS            HVT₂ HARUS            HVT₃ HARUS
    terhubung             terhubung             terhubung
    
       ●───●                 ●───●───●              ●───●───●───●
      /                     /    │                 /    │    \
     ●                     ●     ●                ●     ●     ●
                                                        │
                                                        ●
```

---

## 1.3 Perbedaan ACO-TSP vs ACO-MPDCMST

```
╔═══════════════════╦══════════════════════════╦════════════════════════════════╗
║      ASPEK        ║       ACO-TSP            ║      ACO-MPDCMST               ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Tujuan            ║ Tour terpendek (cycle)   ║ Tree dengan cost minimum       ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Struktur Output   ║ Hamiltonian Cycle        ║ Spanning Tree (n-1 edges)      ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Konstruksi        ║ Sequential path          ║ Tree growing (Prim-like)       ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Start             ║ Random vertex            ║ Fixed central vertex (v₁)      ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Constraint        ║ Visit each vertex once   ║ Degree ≤ d_max, HVT, Period    ║
╠═══════════════════╬══════════════════════════╬════════════════════════════════╣
║ Edge Selection    ║ Kota belum dikunjungi    ║ Tree growing + feasibility     ║
╚═══════════════════╩══════════════════════════╩════════════════════════════════╝
```

---

# BAGIAN 2: PARAMETER DAN RUMUS ACO-MPDCMST

## 2.1 Parameter Konfigurasi

```
╔═════════════════════════════════════════════════════════════════════╗
║                    PARAMETER UNTUK 30 VERTEX                        ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  GRAPH PARAMETERS:                                                  ║
║  • n = 30              : Jumlah vertex                              ║
║  • edges = 435         : Jumlah edge (complete graph)               ║
║  • target = 29         : Edge yang diperlukan (n-1)                 ║
║  • d_max = 3           : Degree constraint                          ║
║                                                                     ║
║  PERIOD PARAMETERS:                                                 ║
║  • n_periods = 3       : Jumlah periode                             ║
║  • MaxVT = ⌈(n-1)/3⌉   : Max edge per periode = ⌈29/3⌉ = 10         ║
║                                                                     ║
║  HVT TABLE (30 vertex):                                             ║
║  • HVT₁ = {2, 3}       : Vertex 2,3 harus connected di periode 1     ║
║  • HVT₂ = {4, 5}       : Vertex 4,5 harus connected di periode 2    ║
║  • HVT₃ = {6, 7}       : Vertex 6,7 harus connected di periode 3    ║
║                                                                     ║
║  ACO PARAMETERS:                                                    ║
║  • n_ants = 15         : Jumlah semut                               ║
║  • n_iterations = 150  : Maksimum iterasi                           ║
║  • α (alpha) = 1.0     : Bobot pengaruh feromon                     ║
║  • β (beta) = 2.5      : Bobot pengaruh heuristik                   ║
║  • ρ (rho) = 0.2       : Evaporation rate                           ║
║  • Q = 100             : Konstanta deposit feromon                  ║
║  • τ₀ = 1.0            : Nilai feromon awal                          ║
║  • τ_min = 0.01        : Feromon minimum                            ║
║  • τ_max = 20.0        : Feromon maksimum                           ║
║  • HVT_bonus = 5.0     : Bonus untuk HVT vertex                     ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

## 2.2 Rumus-Rumus Utama

### ══════════════════════════════════════════════════════════════════
### RUMUS 1: HEURISTIK (Visibility)
### ══════════════════════════════════════════════════════════════════

```
                         1
         η[i][j] = ─────────────
                     d[i][j]

    Dimana:
    • d[i][j] = jarak/bobot edge dari vertex i ke vertex j
    • Semakin PENDEK jarak → semakin TINGGI nilai heuristik
```

**Contoh:**
```
Jika d[1][5] = 50, maka η[1][5] = 1/50 = 0.02
Jika d[1][2] = 10, maka η[1][2] = 1/10 = 0.10 ← lebih menarik!
```

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 2: PROBABILITAS PEMILIHAN EDGE (Extended)
### ══════════════════════════════════════════════════════════════════

```
               τ[u,v]^α × η[u,v]^β × DegPenalty × HVTBonus × PeriodFactor
    P(u,v) = ──────────────────────────────────────────────────────────────
                                      Σ (semua)
```

**Komponen-komponen:**

| Komponen | Formula | Penjelasan |
|----------|---------|------------|
| τ[u,v]^α | τ^1.0 | Faktor FEROMON (pengalaman kolektif) |
| η[u,v]^β | (1/d)^2.5 | Faktor HEURISTIK (kualitas edge) |
| DegPenalty | 1/(1 + totalDeg × 0.2) | Penalti untuk vertex dengan degree tinggi |
| HVTBonus | 5.0 jika v ∈ HVT, else 1.0 | Bonus untuk vertex prioritas |
| PeriodFactor | τ_period[v][p]^0.5 | Feromon khusus periode |

**Implementasi dalam kode:**
```python
def calculate_probability(self, u, v, period, hvt_remaining, degrees, installed, slots_left):
    # Base factors
    tau_factor = max(self.tau[u][v], tau_min) ** alpha      # τ^α
    eta_factor = max(self.eta[u][v], 0.0001) ** beta        # η^β
    
    # Degree penalty - prefer lower degree vertices
    total_deg = degrees[u] + degrees[v]
    deg_penalty = 1.0 / (1.0 + total_deg * 0.2)
    
    # HVT bonus - strong preference for HVT vertices
    if v in hvt_remaining:
        hvt_bonus = 5.0
        if len(hvt_remaining) >= slots_left:  # Urgent!
            hvt_bonus *= 3.0  # Triple bonus
    else:
        hvt_bonus = 1.0
    
    # Period feromon
    period_factor = max(self.tau_period[v][period], tau_min) ** 0.5
    
    prob = tau_factor * eta_factor * deg_penalty * hvt_bonus * period_factor
    return prob
```

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 3: FEASIBILITY CHECK
### ══════════════════════════════════════════════════════════════════

```
Edge (u,v) FEASIBLE jika:
    1. u ∈ Tree (sudah dalam tree)
    2. v ∉ Tree (belum dalam tree)  
    3. degree[u] < d_max (u belum penuh)
    4. degree[v] < d_max (v belum penuh)
    5. Edge (u,v) exists (d[u][v] > 0)
```

**Implementasi dalam kode:**
```python
def get_candidates(self, in_tree, degrees):
    candidates = []
    for u in in_tree:
        if degrees[u] >= self.params.max_degree:  # Skip if u is full
            continue
        for v in range(self.n):
            if v not in in_tree and degrees[v] < self.params.max_degree:
                if self.dist[u][v] > 0:  # Edge exists
                    candidates.append((u, v))
    return candidates
```

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 4: EVAPORASI FEROMON
### ══════════════════════════════════════════════════════════════════

```
    τ[i][j] ← (1 - ρ) × τ[i][j]
    
    Dengan ρ = 0.2:
    τ[i][j] ← 0.8 × τ[i][j]
```

**Interpretasi:**
- Feromon berkurang 20% setiap iterasi
- Lebih lambat dari sebelumnya (ρ=0.5) untuk lebih banyak eksplorasi
- Mencegah konvergensi prematur

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 5: DEPOSIT FEROMON
### ══════════════════════════════════════════════════════════════════

```
    Untuk setiap edge (i,j) dalam solusi VALID:
    
                    Q           100
         Δτ = ─────────── = ───────────
               Cost(T)       Cost(T)

    τ[i][j] ← τ[i][j] + Δτ
    τ[j][i] ← τ[j][i] + Δτ  (symmetric)
```

**Period Bonus (untuk HVT):**
```python
if hvt_verification[period]['satisfied']:
    for v in period_vertices[period]:
        tau_period[v][period] += delta * 0.3
```

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 6: ELITIST STRATEGY
### ══════════════════════════════════════════════════════════════════

```
    Untuk BEST solution yang pernah ditemukan:
    
                    Q                        100
    Δτ_elite = ─────────── × elitist_weight = ───────────── × 2.0
                best_cost                      best_cost

    → Solusi terbaik mendapat deposit EKSTRA setiap iterasi
    → Memperkuat jalur optimal yang sudah ditemukan
```

---

### ══════════════════════════════════════════════════════════════════
### RUMUS 7: CLAMPING FEROMON
### ══════════════════════════════════════════════════════════════════

```
    τ[i][j] = CLAMP(τ[i][j], τ_min, τ_max)
            = CLAMP(τ[i][j], 0.01, 20.0)

    → Mencegah feromon terlalu kecil (stuck)
    → Mencegah feromon terlalu besar (dominasi berlebihan)
```

---

# BAGIAN 3: ALGORITMA STEP-BY-STEP

## 3.1 Flowchart Algoritma ACO-MPDCMST

```
                              ┌───────────────┐
                              │     START     │
                              └───────┬───────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │         INISIALISASI            │
                    │  • Load graph (30 vertices)     │
                    │  • Set τ[i][j] = 1.0            │
                    │  • Hitung η[i][j] = 1/d[i][j]   │
                    │  • Load HVT table               │
                    │  • Set MaxVT = ⌈29/3⌉ = 10      │
                    └───────────────┬─────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────┐
                    │     FOR iteration = 1 to 150    │
                    └───────────────┬─────────────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │     FOR ant = 1 to 15         │
                    └───────────────┬───────────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────────────┐
              │           CONSTRUCT SOLUTION                │
              │  ┌─────────────────────────────────────┐    │
              │  │ FOR period = 1 to 3:                │    │
              │  │   • Get HVT remaining for period    │    │
              │  │   • target = MaxVT (or remaining)   │    │
              │  │                                     │    │
              │  │   WHILE edges < target:             │    │
              │  │     1. Get feasible candidates      │    │
              │  │     2. Force HVT if urgent          │    │
              │  │     3. Calculate probabilities      │    │
              │  │     4. Roulette selection           │    │
              │  │     5. Add edge, update degrees     │    │
              │  │     6. Remove from HVT_remaining    │    │
              │  └─────────────────────────────────────┘    │
              └─────────────────────┬───────────────────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────────────┐
              │  Verify: Is solution VALID?                 │
              │  • 29 edges? ok!                            │
              │  • All HVT satisfied? ok!                   │
              │  • All degrees ≤ 3? ok!                     │
              └─────────────────────┬───────────────────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────────────┐
              │  If cost < best_cost:                       │
              │     Update best_solution                    │
              └─────────────────────┬───────────────────────┘
                                    │
                    └───────────────┬───────────────┘
                                    │ (end ant loop)
                                    ▼
              ┌─────────────────────────────────────────────┐
              │           UPDATE PHEROMONE                  │
              │  1. Evaporation: τ *= 0.8                   │
              │  2. Deposit dari semua valid solutions      │
              │  3. Elitist deposit dari best solution      │
              │  4. Clamp ke [0.01, 20.0]                   │
              └─────────────────────┬───────────────────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────────────┐
              │  Early stopping check:                      │
              │  No improvement for 45 iterations? → STOP   │
              └─────────────────────┬───────────────────────┘
                                    │
                    └───────────────┬───────────────┘
                                    │ (end iteration loop)
                                    ▼
                              ┌───────────────┐
                              │  Return BEST  │
                              │   Solution    │
                              └───────────────┘
```

---

## 3.2 Detail Konstruksi Solusi untuk Satu Semut

### PERIODE 1 (MaxVT = 10 edges)

```
╔═════════════════════════════════════════════════════════════════════╗
║                        PERIODE 1                                    ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  INISIALISASI:                                                      ║
║  • Tree = {v₁}  (central vertex, 0-indexed = vertex 0)              ║
║  • HVT₁ = {2, 3} → hvt_remaining = {1, 2} (0-indexed)               ║
║  • target_edges = 10                                                ║
║  • degrees = {0: 0, 1: 0, 2: 0, ..., 29: 0}                         ║
║                                                                     ║
║  LOOP UNTIL 10 edges installed:                                     ║
║                                                                     ║
║  Edge 1:                                                            ║
║  ─────────                                                          ║
║  • Candidates dari v₀: semua vertex dengan deg < 3                  ║
║  • HVT check: vertex 1,2 dalam HVT → dapat BONUS × 5                ║
║  • Misalkan pilih (0,1) [vertex 1-2] dengan cost = 45               ║
║  • Update: Tree={0,1}, deg[0]=1, deg[1]=1                           ║
║  • hvt_remaining = {2}                                              ║
║                                                                     ║
║  Edge 2:                                                            ║
║  ─────────                                                          ║
║  • Candidates dari v₀ dan v₁                                         ║
║  • HVT urgent: vertex 2 masih belum connected                       ║
║  • Misalkan pilih (0,2) [vertex 1-3] dengan cost = 38               ║
║  • Update: Tree={0,1,2}, deg[0]=2, deg[2]=1                         ║
║  • hvt_remaining = {} ← HVT₁ SATISFIED!                             ║
║                                                                     ║
║  Edge 3-10:                                                         ║
║  ──────────                                                         ║
║  • Tidak ada HVT constraint, pilih berdasarkan probabilitas         ║
║  • Kombinasi τ^α × η^β × deg_penalty                                ║
║  • Install 8 edge lagi untuk total 10 edges di periode 1            ║
║                                                                     ║
║  HASIL PERIODE 1:                                                   ║
║  • 10 edges installed                                               ║
║  • HVT {2,3} ✅ SATISFIED                                           ║
║  • Cost periode 1 = 310                                             ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

### PERIODE 2 (MaxVT = 10 edges)

```
╔═════════════════════════════════════════════════════════════════════╗
║                        PERIODE 2                                    ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STATUS AWAL:                                                       ║
║  • Tree = {11 vertices dari periode 1}                              ║
║  • HVT₂ = {4, 5} → hvt_remaining = {3, 4} (0-indexed)                ║
║  • target_edges = 10                                                ║
║                                                                     ║
║  PRIORITAS:                                                         ║
║  • Vertex 4 dan 5 HARUS terhubung di periode ini                    ║
║  • Jika mendekati batas, HVT mendapat TRIPLE BONUS (15×)            ║
║                                                                     ║
║  PROSES:                                                            ║
║  • Edge pertama: prioritas HVT (vertex 4 atau 5)                    ║
║  • Edge berikutnya: campuran HVT dan non-HVT                        ║
║  • Total 10 edge baru ditambahkan                                   ║
║                                                                     ║
║  HASIL PERIODE 2:                                                   ║
║  • 10 edges installed                                               ║
║  • HVT {4,5} ✅ SATISFIED                                           ║
║  • Cost periode 2 = 329                                             ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

### PERIODE 3 (Sisa = 9 edges)

```
╔═════════════════════════════════════════════════════════════════════╗
║                        PERIODE 3                                    ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STATUS AWAL:                                                       ║
║  • Tree = {21 vertices dari periode 1+2}                            ║
║  • HVT₃ = {6, 7} → hvt_remaining = {5, 6} (0-indexed)               ║
║  • target_edges = 29 - 20 = 9 (sisa)                                ║
║                                                                     ║
║  PRIORITAS:                                                         ║
║  • Vertex 6 dan 7 HARUS terhubung                                   ║
║  • Selesaikan spanning tree (semua 30 vertex connected)             ║
║                                                                     ║
║  HASIL PERIODE 3:                                                   ║
║  • 9 edges installed                                                ║
║  • HVT {6,7} ✅ SATISFIED                                           ║
║  • Cost periode 3 = 921                                             ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 4: CONTOH PERHITUNGAN PROBABILITAS

## 4.1 Setup Contoh

Misalkan di tengah konstruksi periode 1:
- Tree = {0, 1, 2, 5, 8} (5 vertex sudah terhubung)
- degrees = {0: 2, 1: 1, 2: 2, 5: 1, 8: 1}
- hvt_remaining = {} (HVT sudah terpenuhi)
- Mencari edge ke-6

## 4.2 Kandidat Edge

```
Dari vertex 0 (deg=2 < 3): bisa ke v3, v4, v6, v7, v9, ...
Dari vertex 1 (deg=1 < 3): bisa ke v3, v4, v6, v7, v9, ...
Dari vertex 2 (deg=2 < 3): bisa ke v3, v4, v6, v7, v9, ...
Dari vertex 5 (deg=1 < 3): bisa ke v3, v4, v6, v7, v9, ...
Dari vertex 8 (deg=1 < 3): bisa ke v3, v4, v6, v7, v9, ...
```

## 4.3 Perhitungan untuk Beberapa Kandidat

Misalkan kita hitung probabilitas untuk:
- Edge (1, 3) dengan d[1][3] = 25
- Edge (2, 4) dengan d[2][4] = 60
- Edge (5, 6) dengan d[5][6] = 15

```
┌─────────────────────────────────────────────────────────────────────┐
│  PERHITUNGAN PROBABILITAS                                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Edge (1,3): d = 25                                                 │
│  ─────────────────────────────────────────────────                  │
│  • τ[1][3] = 1.0 (awal), τ^α = 1.0^1.0 = 1.0                        │
│  • η[1][3] = 1/25 = 0.04, η^β = 0.04^2.5 = 0.000320                 │
│  • deg_penalty = 1/(1 + (1+0)*0.2) = 1/1.2 = 0.833                  │
│  • hvt_bonus = 1.0 (tidak ada HVT)                                  │
│  • period_factor = 1.0^0.5 = 1.0                                    │
│  • nilai = 1.0 × 0.000320 × 0.833 × 1.0 × 1.0 = 0.000267            │
│                                                                     │
│  Edge (2,4): d = 60                                                 │
│  ─────────────────────────────────────────────────                  │
│  • τ[2][4] = 1.0, τ^α = 1.0                                         │
│  • η[2][4] = 1/60 = 0.0167, η^β = 0.0167^2.5 = 0.0000361            │
│  • deg_penalty = 1/(1 + (2+0)*0.2) = 1/1.4 = 0.714                  │
│  • hvt_bonus = 1.0                                                  │
│  • period_factor = 1.0                                              │
│  • nilai = 1.0 × 0.0000361 × 0.714 × 1.0 × 1.0 = 0.0000258          │
│                                                                     │
│  Edge (5,6): d = 15                                                 │
│  ─────────────────────────────────────────────────                  │
│  • τ[5][6] = 1.0, τ^α = 1.0                                         │
│  • η[5][6] = 1/15 = 0.0667, η^β = 0.0667^2.5 = 0.001149             │
│  • deg_penalty = 1/(1 + (1+0)*0.2) = 0.833                          │
│  • hvt_bonus = 1.0                                                  │
│  • period_factor = 1.0                                              │
│  • nilai = 1.0 × 0.001149 × 0.833 × 1.0 × 1.0 = 0.000957            │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  NORMALISASI:                                                       │
│  Total = 0.000267 + 0.0000258 + 0.000957 + ... = (misalkan 0.005)   │
│                                                                     │
│  P(1,3) = 0.000267 / 0.005 = 5.34%                                  │
│  P(2,4) = 0.0000258 / 0.005 = 0.52%                                 │
│  P(5,6) = 0.000957 / 0.005 = 19.14% ← TERTINGGI (edge terpendek!)   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Observasi:**
- Edge (5,6) dengan jarak 15 memiliki probabilitas tertinggi
- Efek β = 2.5 membuat perbedaan jarak sangat berpengaruh
- Edge pendek jauh lebih disukai daripada edge panjang

---

# BAGIAN 5: UPDATE FEROMON

## 5.1 Setelah Semua Semut Selesai (1 Iterasi)

Misalkan dari 15 semut, ada 12 solusi VALID dengan cost:
- Semut 1: Cost = 1650 ok!
- Semut 2: Cost = 1580 ok!
- Semut 3: Cost = 1720 ok!
- ...
- Semut 8: Cost = 1560 ok! ← BEST!
- ...

## 5.2 Proses Update

```
╔═════════════════════════════════════════════════════════════════════╗
║                      UPDATE FEROMON                                 ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STEP 1: EVAPORASI                                                  ║
║  ─────────────────                                                  ║
║  τ[i][j] = 0.8 × τ[i][j] untuk SEMUA edge                           ║
║                                                                     ║
║  Contoh: τ[0][1] = 0.8 × 1.0 = 0.8                                  ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STEP 2: DEPOSIT DARI SEMUA SOLUSI VALID                            ║
║  ────────────────────────────────────────                           ║
║                                                                     ║
║  Semut 1 (cost=1650): Δτ₁ = 100/1650 = 0.0606                        ║
║    → deposit ke semua edge dalam solusi semut 1                     ║
║                                                                     ║
║  Semut 2 (cost=1580): Δτ₂ = 100/1580 = 0.0633                        ║
║    → deposit ke semua edge dalam solusi semut 2                     ║
║                                                                     ║
║  ...                                                                ║
║                                                                     ║
║  Semut 8 (cost=1560): Δτ₈ = 100/1560 = 0.0641                        ║
║    → deposit ke semua edge dalam solusi semut 8                     ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STEP 3: ELITIST DEPOSIT (Best Solution)                            ║
║  ───────────────────────────────────────                            ║
║                                                                     ║
║  Best cost = 1560                                                   ║
║  Δτ_elite = (100/1560) × 2.0 = 0.1282                               ║
║                                                                     ║
║  → Edge dalam best solution mendapat DOUBLE deposit                 ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  STEP 4: CLAMPING                                                   ║
║  ───────────────                                                    ║
║  τ[i][j] = max(0.01, min(20.0, τ[i][j]))                            ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

## 5.3 Efek pada Iterasi Berikutnya

```
┌─────────────────────────────────────────────────────────────────────┐
│  PERUBAHAN FEROMON                                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Edge SERING DIGUNAKAN (dalam best solution):                       │
│  ────────────────────────────────────────────                       │
│  τ_old = 1.0                                                        │
│  τ_after_evap = 0.8                                                 │
│  τ_after_deposit = 0.8 + 0.0641 + ... + 0.1282 = ~1.5               │
│  → MENINGKAT! → Lebih mungkin dipilih lagi                          │
│                                                                     │
│  Edge JARANG DIGUNAKAN:                                             │
│  ─────────────────────                                              │
│  τ_old = 1.0                                                        │
│  τ_after_evap = 0.8                                                 │
│  τ_after_deposit = 0.8 + 0 = 0.8                                    │
│  → MENURUN! → Kurang mungkin dipilih                                │
│                                                                     │
│  Edge TIDAK PERNAH DIGUNAKAN:                                       │
│  ────────────────────────────                                       │
│  τ iterasi 1: 1.0 → 0.8                                             │
│  τ iterasi 2: 0.8 → 0.64                                            │
│  τ iterasi 3: 0.64 → 0.512                                          │
│  ...                                                                │
│  → Terus menurun sampai τ_min = 0.01                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# BAGIAN 6: HASIL AKHIR - 30 VERTEX

## 6.1 Konfigurasi yang Digunakan

```
╔════════════════════════════════════════════════════════════════════╗
║                    📌 KONFIGURASI                                  ║
╠════════════════════════════════════════════════════════════════════╣
║  • Vertices (JV)     : 30                                          ║
║  • Problem Number    : 1                                           ║
║  • Degree Constraint : ≤ 3                                         ║
║  • Periods           : 3                                           ║
║  • Dataset           : Generated (seed=1)                          ║
╚════════════════════════════════════════════════════════════════════╝
```

## 6.2 Output Program

```
══════════════════════════════════════════════════════════════════════
  🐜 ACO MPDCMST SOLVER
══════════════════════════════════════════════════════════════════════
  Graph: 30 vertices, 435 edges
  Target: 29 edges (spanning tree)
  Periods: 3, MaxVT: 10 edges/period
  Degree Constraint: ≤ 3
──────────────────────────────────────────────────────────────────────
  HVT Period 1: {2, 3}
  HVT Period 2: {4, 5}
  HVT Period 3: {6, 7}
──────────────────────────────────────────────────────────────────────
  ACO: 15 ants × 150 iterations
  α=1.0, β=2.5, ρ=0.2, HVT_bonus=5.0
══════════════════════════════════════════════════════════════════════
  Early stopping at iteration 10
══════════════════════════════════════════════════════════════════════
  ✅ SOLUTION FOUND: Cost = 1560
══════════════════════════════════════════════════════════════════════
```

## 6.3 Hasil Detail

```
╔════════════════════════════════════════════════════════════════════╗
║                    📋 HASIL ACO MPDCMST                            ║
╠════════════════════════════════════════════════════════════════════╣
║  Graph          : 30 vertices, 29 edges target                     ║
║  Max Degree     : 3                                                ║
║  Periods        : 3                                                ║
║  MaxVT/Period   : 10                                               ║
╠════════════════════════════════════════════════════════════════════╣
║                         HVT TABLE                                  ║
╠════════════════════════════════════════════════════════════════════╣
║  Period │ HVT Required     │ Connected        │ Status             ║
║─────────┼──────────────────┼──────────────────┼─────────────────── ║
║    1    │ {2, 3}           │ {2, 3}           │ ✅ SATISFIED       ║
║    2    │ {4, 5}           │ {4, 5}           │ ✅ SATISFIED       ║
║    3    │ {6, 7}           │ {6, 7}           │ ✅ SATISFIED       ║
╠════════════════════════════════════════════════════════════════════╣
║  Overall Status : ✅ ALL SATISFIED                                 ║
╠════════════════════════════════════════════════════════════════════╣
║                      PERIOD DETAILS                                ║
╠════════════════════════════════════════════════════════════════════╣
║  Period 1: 10 edges, cost =    310                                 ║
║  Period 2: 10 edges, cost =    329                                 ║
║  Period 3:  9 edges, cost =    921                                 ║
╠════════════════════════════════════════════════════════════════════╣
║  🏆 TOTAL COST: 1560                                               ║
╚════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 7: RINGKASAN RUMUS

```
╔═════════════════════════════════════════════════════════════════════╗
║                                                                     ║
║              📚 RINGKASAN SEMUA RUMUS ACO-MPDCMST 📚               ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  1. HEURISTIK                                               │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       η[i][j] = 1 / d[i][j]                                 │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  2. PROBABILITAS PEMILIHAN EDGE                             │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │  P(u,v) = [τ^α × η^β × DegPenalty × HVTBonus × PeriodFac]   │    ║
║  │           ───────────────────────────────────────────────── │    ║
║  │                            Σ (semua kandidat)               │    ║
║  │                                                             │    ║
║  │  dengan:                                                    │    ║
║  │  • α = 1.0, β = 2.5                                         │    ║
║  │  • DegPenalty = 1/(1 + totalDeg × 0.2)                      │    ║
║  │  • HVTBonus = 5.0 (atau 15.0 jika urgent)                   │    ║
║  │  • PeriodFac = τ_period[v][p]^0.5                           │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  3. EVAPORASI                                               │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       τ[i][j] ← (1 - ρ) × τ[i][j] = 0.8 × τ[i][j]           │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  4. DEPOSIT                                                 │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       Δτ = Q / Cost = 100 / Cost                            │    ║
║  │       τ[i][j] ← τ[i][j] + Δτ                                │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  5. ELITIST DEPOSIT                                         │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       Δτ_elite = (Q / best_cost) × elitist_weight           │    ║
║  │                = (100 / best_cost) × 2.0                    │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  6. CLAMPING                                                │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       τ[i][j] = CLAMP(τ[i][j], 0.01, 20.0)                  │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  7. MaxVT (Max Vertices per Period)                         │    ║
║  │  ─────────────────────────────────────────────────────────  │    ║
║  │                                                             │    ║
║  │       MaxVT = ⌈(n-1) / n_periods⌉                           │    ║
║  │             = ⌈29/3⌉ = 10                                   │    ║
║  │                                                             │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
╚═════════════════════════════════════════════════════════════════════╝
```

---

# BAGIAN 8: KESIMPULAN

```
╔═════════════════════════════════════════════════════════════════════╗
║                                                                     ║
║                      KESIMPULAN AWAL 	                              ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  ACO-MPDCMST menggabungkan beberapa komponen:                       ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  1.  ANT COLONY OPTIMIZATION                                │    ║
║  │     • Feromon untuk menyimpan pengalaman kolektif           │    ║
║  │     • Multiple agents (15 semut) exploring                  │    ║
║  │     • Probabilistic selection dengan roulette wheel         │    ║
║  │     • Learning dari solusi yang baik (elitist)              │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  2.  PRIM-LIKE TREE GROWING                                 │    ║
║  │     • Start dari central vertex (v₁)                         │    ║
║  │     • Tree grows by adding feasible edges                   │    ║
║  │     • Always maintain connected tree                        │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  3.  DEGREE CONSTRAINT                                      │    ║
║  │     • degree(v) ≤ d_max = 3                                 │    ║
║  │     • Feasibility check sebelum memilih edge                │    ║
║  │     • Degree penalty dalam probabilitas                     │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
║  ┌─────────────────────────────────────────────────────────────┐    ║
║  │  4.  MULTI-PERIOD INSTALLATION                              │    ║
║  │     • 3 periode dengan MaxVT = 10 per periode               │    ║
║  │     • HVT constraint per periode                            │    ║
║  │     • HVT bonus untuk prioritas instalasi                   │    ║
║  └─────────────────────────────────────────────────────────────┘    ║
║                                                                     ║
╠═════════════════════════════════════════════════════════════════════╣
║                                                                     ║
║  HASIL UNTUK 30 VERTEX:                                             ║
║                                                                     ║
║  ┏━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    ║
║  ┃        Metrik          ┃             Nilai                ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ Total Cost             ┃ 1560                             ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ Period 1 Cost          ┃ 310 (10 edges)                   ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ Period 2 Cost          ┃ 329 (10 edges)                   ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ Period 3 Cost          ┃ 921 (9 edges)                    ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ HVT Status             ┃ ✅ ALL SATISFIED                 ┃       ║
║  ┣━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    ║
║  ┃ Convergence            ┃ Early stop at iteration 10       ┃       ║
║  ┗━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛    ║
║                                                                      ║
║  🏆 ACO-MPDCMST berhasil menemukan solusi VALID dengan               ║
║     cost = 1560 dalam waktu singkat (10 iterasi)!                    ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

# LAMPIRAN: PARAMETER REFERENSI

## A. HVT Table untuk Berbagai Ukuran Graf

```
┏━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Vertex  ┃      HVT₁       ┃        HVT₂         ┃         HVT₃          ┃
┣━━━━━━━━━╋━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━┫
┃   10    ┃ {2}             ┃ {3}                 ┃ {4}                   ┃
┃   20    ┃ {2}             ┃ {3}                 ┃ {4}                   ┃
┃   30    ┃ {2, 3}          ┃ {4, 5}              ┃ {6, 7}                ┃
┃   40    ┃ {2, 3, 4}       ┃ {5, 6, 7}           ┃ {8, 9, 10}            ┃
┃   50    ┃ {2, 3, 4, 5}    ┃ {6, 7, 8, 9}        ┃ {10, 11, 12, 13}      ┃
┃   60    ┃ {2, 3, 4, 5, 6} ┃ {7, 8, 9, 10, 11}   ┃ {12, 13, 14, 15}      ┃
┃   70    ┃ {2..7}          ┃ {8..13}             ┃ {14..19}              ┃
┃   80    ┃ {2..8}          ┃ {9..15}             ┃ {16..22}              ┃
┃   90    ┃ {2..8}          ┃ {9..15}             ┃ {16..22}              ┃
┃  100    ┃ {2..9}          ┃ {10..17}            ┃ {18..25}              ┃
┗━━━━━━━━━┻━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━┛
```

## B. ACO Parameters by Problem Size

```
┏━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━┳━━━━━━━━┳━━━━━━━┓
┃ Vertex Size ┃ n_ants  ┃ n_iterations ┃   α    ┃   β    ┃   ρ   ┃
┣━━━━━━━━━━━━━╋━━━━━━━━━╋━━━━━━━━━━━━━━╋━━━━━━━━╋━━━━━━━━╋━━━━━━━┫
┃   ≤ 30      ┃   15    ┃     150      ┃  1.0   ┃  2.5   ┃  0.2  ┃
┃   31-50     ┃   25    ┃     200      ┃  1.0   ┃  2.5   ┃  0.2  ┃
┃   51-70     ┃   35    ┃     300      ┃  1.0   ┃  2.5   ┃  0.2  ┃
┃   > 70      ┃   50    ┃     400      ┃  1.0   ┃  2.5   ┃  0.2  ┃
┗━━━━━━━━━━━━━┻━━━━━━━━━┻━━━━━━━━━━━━━━┻━━━━━━━━┻━━━━━━━━┻━━━━━━━┛
```

--- 
Referensi:
Wamiliana, Sakethi, D., & Yuniarti, R. (2010). Computational aspect of WADR1 and WADR2 algorithms for the multiperiod degree constrained minimum spanning tree problem. Proceeding of the National Conference on Mathematics and its Applications (SNMAP), 208–214.
Wamiliana, Amanto, & Usman, M. (2013). Comparative analysis for the multiperiod degree constrained minimum spanning tree problem. Proceedings of the International Conference on Engineering and Technology Development (ICETD), 39–43.
Wamiliana, Elfaki, F. A. M., Usman, M., & Azram, M. (2015a). Some greedy based algorithms for multiperiod degree constrained minimum spanning tree problem. ARPN Journal of Engineering and Applied Sciences, 10(21), 10147–10152.
Wamiliana, Usman, M., Sakethi, D., Yuniarti, R., & Cucus, A. (2015b). The hybrid of depth first search technique and Kruskal’s algorithm for solving the multiperiod degree constrained minimum spanning tree problem. 2015 4th International Conference on Interactive Digital Media (ICIDM), 1–4. IEEE.
Wamiliana, Junaidi, A., Amanto, Usman, M., & Warsono. (2020a). WAC4 algorithm to solve the multiperiod degree constrained minimum spanning tree problem. Journal of Physics: Conference Series, 1524, 012046.
Wamiliana, Usman, M., Warsito, W., Warsono, W., & Daoud, J. I. (2020b). Using modification of Prim’s algorithm and GNU Octave to solve the multiperiod installation problem. IIUM Engineering Journal, 21(1), 100–112.
Wamiliana, W., Junaidi, A., Gamal, M. D. H., & Thamrin, T. (2024). The use of probability and edge analysis to solve the multiperiod degree constrained minimum spanning tree problem. Science and Technology Indonesia, 9(4).​

**═══════════════════════════════════════════════════════════════════**
**                        END OF DOCUMENT                            **
**═══════════════════════════════════════════════════════════════════**
