# 🚀 Uniconf v3.0.0-beta

Standalone, OpenMP-parallelized conformer, transition-state, and solvation-cluster generator & deduplicator for Linux (`x86_64`). Compiled as a fully static single binary (~2 MB) with zero runtime dependencies.

## 1. Download & Quick Start

1. Download the pre-compiled `uniconf` binary from the **Releases** page (or directly from this branch).
2. Make it executable:
   ```bash
   chmod +x uniconf
   ```
3. Run on any `.xyz` file (use `-j=<N>` to set the number of OpenMP threads):
   ```bash
   ./uniconf input.xyz --nconf=500 --max-conf=30 -j=4
   ```
4. View all available flags:
   ```bash
   ./uniconf --help
   ```

---

## 2. Examples (`examples/`)

### A. Organic Molecules & Peptides
* `examples/organic/GFA10.xyz` — [краткое описание молекулы / число атомов]
  ```bash
  ./uniconf examples/organic/GFA10.xyz --nconf=500 --top-energy=100 --max-conf=15 -j=4
  ```
* `examples/organic/GPPGPP_0.xyz` — hexapeptide with 4 proline rings (ring flaps + backbone torsions)
  ```bash
  ./uniconf examples/organic/GPPGPP_0.xyz --nconf=500 --top-energy=100 --max-conf=15 -j=4
  ```
* `examples/organic/m1_00_YGGFL.xyz` — Leu-enkephalin pentapeptide
  ```bash
  ./uniconf examples/organic/m1_00_YGGFL.xyz --nconf=500 --top-energy=100 --max-conf=15 -j=4
  ```

### B. Transition-Metal Complexes
* `examples/tm/akugod_10_b973c.xyz` — DFT-optimized transition-metal complex (`B97-3c`)
  ```bash
  ./uniconf examples/tm/akugod_10_b973c.xyz --dftref=true --tmfr=true --nconf=500 --max-conf=20 -j=4
  ```
* `examples/tm/amozeh_10_b973c.xyz` — DFT-optimized transition-metal complex (`B97-3c`)
  ```bash
  ./uniconf examples/tm/amozeh_10_b973c.xyz --dftref=true --tmfr=true --nconf=500 --max-conf=20 -j=4
  ```

### C. Solvation Shells & Clusters
* `examples/clusters/m1_01_NaDMC6.xyz` — $[\text{Na}(\text{DMC})_6]^+$ solvation shell (`Na+` at atom index 73)
  ```bash
  ./uniconf examples/clusters/m1_01_NaDMC6.xyz --center=73 --nconf=500 --top-center=100 --max-conf=10 -j=4
  ```
* `examples/clusters/m1_01_NaPC6.xyz` + `examples/clusters/m1_01_NaPC6.bond` — $[\text{Na}(\text{PC})_6]^+$ solvation shell (`Na+` at atom index 79)
  ```bash
  ./uniconf examples/clusters/m1_01_NaPC6.xyz --readbond=examples/clusters/m1_01_NaPC6.bond --center=79 --nconf=500 --top-center=100 --max-conf=10 -j=4
  ```

### D. Transition States (TS)
* `examples/ts/m1_00_Ru-TS.xyz` + `examples/ts/m1_00_Ru-TS.bond` — Ruthenium transition state with custom reactive-core connectivity
  ```bash
  # 1. Export initial connectivity template (writes .bond and exits):
  ./uniconf examples/ts/m1_00_Ru-TS.xyz --geomprint=true

  # 2. Run conformational sampling with edited .bond and frozen reactive center:
  ./uniconf examples/ts/m1_00_Ru-TS.xyz --readbond=examples/ts/m1_00_Ru-TS.bond --dftref=true --tmfr=true --nconf=500 --max-conf=20 -j=4
  ```

---

## 3. Main Options Summary

| Flag | Description |
| :--- | :--- |
| `--nconf=<int>` | Number of quasi-random (`Sobol+Kronecker`) conformers to generate (default: `100`) |
| `--dftref=true\|false` | Calibrate equilibrium bond lengths and angles to the input geometry |
| `--tmfr=true\|false` | Freeze transition metal(s) and 1–2 coordination sphere during L-BFGS |
| `--geomprint=true` | Write perceived connectivity & UFF atom types to `<stem>.bond` and exit |
| `--readbond=<file.bond>` | Load custom connectivity / partial bonds from `.bond` |
| `--center=<int>` | Anchor cluster / solvation shell around 1-based atom index |
| `--top-center=<int>` | Keep $N$ most centered shells around `--center` before diversity selection |
| `--top-energy=<int>` | Keep $N$ lowest-energy unique conformers (`0` = all, no FPS) |
| `--max-conf=<int>` | Select $K$ maximally diverse conformers via MaxMin FPS |
| `-j=<int>` | Number of OpenMP threads (`0` = all available cores) |
| `-o=<file>` | Output multi-frame `.xyz` file (default: `<input>-conf.xyz`) |

---

## 4. Citation

If you use **Uniconf** in your research, please cite:

1. [Citation 1 — Authors, Title, Journal, Year, Volume, Pages / DOI]
2. [Citation 2 — Authors, Title, Journal, Year, Volume, Pages / DOI]

---

## 5. Contact & Feedback

For bug reports, questions, or edge-case `.xyz` files, please open an Issue or contact:  
**Yury Minenkov** — `Yury.Minenkov@gmail.com`
