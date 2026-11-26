# 🚀 Uniconf v3.0.0

Standalone, `UFF`-based OpenMP-parallelized conformer, transition-state, and solvation-cluster generator & deduplicator for Linux (`x86_64`). Compiled as a fully static single binary (~2 MB) with zero runtime dependencies.

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

## 3. Command-Line Options

```bash
./uniconf <input.xyz> [options]
```

### General & Parallel Execution
* `-o=<file>`, `--out=<file>` — Output multi-frame `.xyz` file containing the final ensemble (default: `<input>-conf.xyz`).
* `-j=<int>`, `--j=<int>` — Number of OpenMP threads (`0` = use all available CPU cores; recommended `-j=4` or `-j=8` on shared login nodes).
* `-h`, `--help` — Print full CLI help message and exit.

### Sampling & Force-Field Relaxation (UFF + L-BFGS)
* `--nconf=<int>` — Number of conformers to generate (default: `100`):
  * `--nconf=0`: **Full systematic enumeration** over all discrete torsion/flap/inversion combinations (automatically enabled when the total combination count is $\le 10^6$).
  * `--nconf=N` ($N > 0$): **Quasi-random sampling** using a deterministic `Sobol` (1–40D) + `Kronecker` (41D+) low-discrepancy sequence.
* `--niter=<int>` — Maximum number of L-BFGS energy/gradient evaluations per conformer (default: `5500`).
* `--dftref=true|false` — Calibrate equilibrium bond lengths ($r_0$) and valence angles ($	heta_0$) directly from the input reference `.xyz` geometry (default: `false`; recommended when starting from a DFT-optimized structure).
* `--tmfr=true|false` — Freeze transition-metal center(s) and their 1–2 coordination sphere during L-BFGS relaxation while allowing spectator ligands to rotate freely (default: `false`).
* `--geomprint=true` — Perceive molecular connectivity and UFF atom types, write them to `<stem>.bond`, and **exit immediately** without running conformer generation (default: `false`).
* `--readbond=<file.bond>` — Override automatic bond perception by reading custom connectivity and UFF atom types from `<file.bond>` (essential for transition states with stretched partial bonds or non-covalent solvation shells).

### Cluster, Solvation & Non-Covalent Interaction (NCI) Mode
*(Activated automatically whenever the molecular graph contains more than one disconnected fragment; supports explicit partial charges in column 5 of `.xyz`, formal monatomic ions, and fragment-wise QEq polarization.)*
* `--center=<int>` — 1-based atom index of the central atom/ion to anchor at the origin $(0,0,0)$ and surround with a spherical solvation shell (default: `0` = cluster center of mass).
* `--sort=energy|center` — Rank optimized conformers either by UFF energy (`energy`, default) or by ascending offset $d_{	ext{center}} = \|\mathbf{r}_{	ext{anchor}} - \mathbf{r}_{	ext{COM}}\|$ (`center`).
* `--top-center=<int>` — Retain only the $N$ most symmetrically centered shells around `--center` prior to diversity selection (default: `0` = keep all).
* `--wall-r=<float>` — Radius $R_{	ext{wall}}$ (in Å) of the smooth quartic confining cavity preventing cluster evaporation during L-BFGS (default: `0.0` = automatic sizing from fragment radii).
* `--eps=<float>` — Effective dielectric constant $arepsilon$ screening intermolecular Coulomb interactions (default: `2.0`).

### Deduplication, Energy Truncation & Diversity Selection
* `--dedupl=<mode>` — Duplicate conformer removal algorithm (default: `angle`):
  * `angle` — Graph-isomorphic Hungarian permutation + Kabsch alignment + heavy-atom RMSD + max atomic displacement ($D_{\max}$) + max torsion deviation ($\Delta\phi_{\max}$).
  * `rmsd-graph` — Graph-isomorphic Hungarian + Kabsch heavy-atom RMSD only.
  * `rmsd-hungarian` — Element-wise Hungarian + Kabsch heavy-atom RMSD.
  * `rmsd-kabsch` — Direct atom-order Kabsch heavy-atom RMSD.
  * `none` — Disable deduplication.
* `--rmsd=<float>` — Heavy-atom RMSD threshold in Å for duplicate detection (default: `0.50` in `angle` mode, `0.125` in `rmsd-*` modes).
* `--maxdist=<float>` — Maximum single-atom displacement threshold $D_{\max}$ in Å in `angle` mode (default: `1.50`).
* `--angle=<float>` — Maximum torsion-angle deviation threshold in degrees in `angle` mode (default: `30.0`).
* `--keepenantio=true|false` — Preserve non-superimposable mirror-image enantiomers as distinct conformers (default: `false`).
* `--top-energy=<int>` — Truncate the deduplicated ensemble to the $N$ lowest-energy unique conformers (default: `0` = keep all). Can be used standalone for energy-ranked output without clustering, or combined with `--max-conf=<K>` as a two-stage energy-then-diversity filter.
* `--max-conf=<int>` — Select $K$ maximally diverse representatives via parallel Farthest-Point Sampling (MaxMin FPS) in symmetry-invariant heavy-atom RMSD space (default: `0` = output all unique conformers).

---

## 4. Citation

If you use **Uniconf** in your research, please cite:

1. Uniconf: An alternative conformer generator with broad applicability\
   Y. Minenkov Chem. Phys.Lett. 2025, 860, 141813 (DOI: 10.1016/j.cplett.2024.141813)
2. Molcomp: A Software Tool for the Structural Characterization and Deduplication of Conformers\
   A. M. Genaev, A. D. Moshchenkov, A. S. Ryzhako, A. A. Otlyotov, and Y. Minenkov Adv. Theor. Simul. 2026, 9, e70604 (DOI: 10.1002/adts.70604)
---

## 5. Contact & Feedback

For bug reports, questions, or edge-case `.xyz` files, please open an Issue or contact:  
**Yury Minenkov** — `Yury.Minenkov@gmail.com`
