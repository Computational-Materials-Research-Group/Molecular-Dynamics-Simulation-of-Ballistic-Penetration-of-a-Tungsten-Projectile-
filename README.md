# Molecular Dynamics Simulation of Ballistic Penetration of a Tungsten Projectile through a Thin Aluminum Sheet

<p align="center">
  <img src="https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tungsten-Projectile-silver?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Aluminum-Target%20Sheet-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Ballistic-Penetration-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/EAM-CuAlW%20Potential-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A fully atomistic <b>molecular dynamics simulation of ballistic penetration</b>
  of a tungsten spherical projectile through a thin aluminum sheet using LAMMPS.
  Models a BCC tungsten hard sphere fired at ~1500 m/s toward a clamped FCC aluminum
  sheet, capturing full penetration dynamics, stress wave propagation, petaling,
  plastic deformation, and spallation via the <i>EAM/alloy potential (CuAlW.txt)</i>.
  Von Mises stress and hydrostatic pressure are computed per-atom across approach,
  impact, penetration, and post-impact relaxation phases.
</p>
<img width="1600" height="1200" alt="ball impact2" src="https://github.com/user-attachments/assets/661ce3f4-1d0c-47a0-bf13-5ecc606fcb18" />

---

## Physics

The simulation captures the full sequence of ballistic penetration at the atomistic scale:

- FCC aluminum sheet and BCC tungsten sphere modeled with the EAM/alloy interatomic potential (CuAlW.txt)
- Tungsten projectile assigned ballistic impact velocity (~1500 m/s) directed normal to sheet surface
- Clamped boundary conditions on sheet edges simulate a real ballistic fixture
- NVE integration for both projectile and sheet during impact phase for full momentum conservation
- Per-atom stress tensor computation via `compute stress/atom`
- Von Mises stress and hydrostatic pressure fields extracted across all phases
- Full penetration captured: projectile enters sheet top surface and exits through bottom

---

## Impact Regime

| Regime | Velocity Range | This Simulation |
|--------|---------------|-----------------|
| Low velocity impact | < 100 m/s | ❌ |
| High velocity impact | 100–1000 m/s | ❌ |
| **Ballistic impact** | **1000–3000 m/s** | **✅** |
| Hypervelocity impact | > 3000 m/s | ❌ |

---

## Geometry

```
  Simulation box (units: Angstrom):

  z = 120  +------------------------------------------+  <- box top
           |                                          |
  z =  70  |        ●●●●●●●●●●●●●●●                  |
           |      ●               ●                  |
           |     ●   W Ball        ●  <- r = 25 Ang  |
           |      ●               ●   center(100,100,70)
           |        ●●●●●●●●●●●●●●●                  |
           |              ↓ v = -15 Ang/ps            |
  z =  12  +==========================================+  <- Al sheet top
           |         ALUMINUM SHEET                  |  <- 12 Ang thick
  z =   0  +==========================================+  <- Al sheet bottom
           |                                          |
  z = -60  +------------------------------------------+  <- box bottom

  Box XY: 0 to 200 Angstrom (periodic in x and y)
  Box Z:  -60 to +120 Angstrom (free in z)
  Sheet edges: clamped (setforce 0) — 4 Ang border
  Sheet inner: free NVE dynamics during impact
```

- **Al Sheet**: FCC Al slab, 200 × 200 × 12 Å; edges clamped (4 Å border frozen)
- **W Ball**: BCC tungsten sphere, radius = 25 Å, center at (100, 100, 70)
- **Impact velocity**: -15 Å/ps (~1500 m/s) in -Z direction
- **Boundary**: Periodic in x, y; free (f) in z

---

## Simulation Phases

| Phase | Run Steps | Simulated Time | Description |
|-------|-----------|----------------|-------------|
| Equilibration | 3,000 | 3 ps | Sheet thermalization at 300 K; ball frozen in place |
| Approach | ~2,200 | ~2.2 ps | Ball travels from z=70 to first contact at z=37 |
| Impact & Penetration | ~2,500 | ~2.5 ps | Ball punches through 12 Å sheet |
| Exit & Aftermath | ~7,300 | ~7.3 ps | Ball exits below sheet; hole and debris stabilize |
| Post-impact relaxation | 5,000 | 5 ps | Sheet relaxes; stress wave dissipates |
| **Total** | **20,000** | **20 ps** | — |

---

## Key Event Timeline

| Event | Step | Frame (dump/100) |
|-------|------|-----------------|
| Ball released | 3,000 | 30 |
| Ball bottom touches sheet top | ~5,200 | ~52 |
| Ball center at sheet top | ~6,867 | ~69 |
| Ball center at sheet midplane | ~7,267 | ~73 |
| Ball center exits sheet bottom | ~7,667 | ~77 |
| Ball fully clear of sheet | ~9,333 | ~93 |

---

## Material Parameters

### Aluminum Sheet Properties

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Crystal structure | — | FCC | — |
| Lattice constant | a | 4.05 | Angstrom |
| Atomic mass | m | 26.982 | g/mol |
| Sheet dimensions | — | 200 × 200 × 12 | Angstrom |

### Tungsten Projectile Properties

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Crystal structure | — | BCC | — |
| Lattice constant | a | 3.165 | Angstrom |
| Atomic mass | m | 183.84 | g/mol |
| Sphere radius | r | 25 | Angstrom |
| Impact velocity | v | -15 | Angstrom/ps (~1500 m/s) |

### Interatomic Potential

| Parameter | Value |
|-----------|-------|
| Potential file | CuAlW.txt |
| Potential style | EAM/alloy |
| Al mapping | Type 1 → Al |
| W mapping | Type 2 → W |
| pair_coeff | `* * CuAlW.txt Al W` |

### Boundary Conditions

| Group | Condition | Description |
|-------|-----------|-------------|
| Sheet edges (4 Å border) | `setforce 0 0 0` | Clamped — fixed like Charpy fixture |
| Sheet inner | NVE | Full impact dynamics |
| W ball | NVE | Conserves projectile momentum |
| Z boundary | Free (f) | Allows ball to exit below sheet |

---

## Governing Equations

### Ballistic Velocity

```
v_impact = -15 Ang/ps = 1500 m/s  (ballistic regime: 1000–3000 m/s)

Kinetic energy of W ball:
KE = 0.5 * N_W * m_W * v^2

where:
  N_W  = number of W atoms in sphere
  m_W  = 183.84 amu per atom
  v    = 15 Ang/ps
```

### EAM/Alloy Potential Energy

```
E_i = F_alpha(sum_{j != i} rho_alpha_beta(r_ij)) + 0.5 * sum_{j != i} phi_alpha_beta(r_ij)

where:
  F_alpha = embedding energy function
  rho     = electron density contribution
  phi     = pair interaction potential
```

### Per-Atom Stress Tensor (LAMMPS units: bar * Ang^3)

```
sigma_ab = (1/V_i) * [ -m_i * v_ia * v_ib + 0.5 * sum_j (r_iab * f_ijb) ]
```

### Von Mises Stress

```
sigma_VM = sqrt( 0.5 * [ (sxx-syy)^2 + (syy-szz)^2 + (szz-sxx)^2
                        + 6*(sxy^2 + sxz^2 + syz^2) ] )
```

### Hydrostatic Pressure

```
P = -(sxx + syy + szz) / 3

Compression: P > 0  (beneath impact zone)
Tension:     P < 0  (spallation zone — behind sheet)
```

---

## Group Definitions

```
al_sheet      <- FCC Al slab atoms (z = 0 to 12 Ang)
w_ball        <- BCC W sphere atoms (center z=70, r=25)
sheet_inner   <- Al atoms excluding 4 Ang edge border (free NVE)
sheet_edge    <- Al border atoms — clamped with setforce 0 0 0
```

---

## Output Files

| File | Dump Frequency | Content |
|------|---------------|---------|
| `impact_stress.lammpstrj` | every 100 steps | id, type, x, y, z, vx, vy, vz, von Mises, pressure, full stress tensor |
| `impact_vonmises.lammpstrj` | every 100 steps | id, type, x, y, z, von Mises, hydrostatic pressure |
| `stress_history.dat` | every 100 steps | avg/max Von Mises, avg/min/max pressure vs timestep |
| `impact_final.data` | — | Final atomic positions and velocities (LAMMPS data format) |
| `impact_final.restart` | — | Binary restart file for continuation runs |

---

## Repository Structure

```
ballistic_impact_md/
|
|-- tungsten_impact_ballmoves.lammps     # Main LAMMPS simulation script
|-- CuAlW.txt                            # EAM potential file (required)
|-- README.md                            # This file
|
|-- outputs/                             # Generated on run
    |-- impact_stress.lammpstrj          # Full stress trajectory
    |-- impact_vonmises.lammpstrj        # Von Mises + pressure trajectory
    |-- stress_history.dat               # Stress vs time data
    |-- impact_final.data                # Final atomic configuration
    |-- impact_final.restart             # Restart file
```

---

## How to Run

### Requirements

- LAMMPS (any recent version with MANYBODY package): https://www.lammps.org
- EAM potential file `CuAlW.txt` (place in same directory as script)
- OVITO or VMD for trajectory visualization: https://www.ovito.org

### Step 1 — Place potential file

```bash
ls CuAlW.txt   # must be in same directory as script
```

### Step 2 — Run the simulation

```bash
lmp -in tungsten_impact_ballmoves.lammps
```

Or in parallel with MPI:

```bash
mpirun -np 4 lmp -in tungsten_impact_ballmoves.lammps
```

### Step 3 — Visualize in OVITO

1. Open OVITO > `File > Load File` > select `impact_vonmises.lammpstrj`
2. Add modifier: **Color Coding** → set property to `v_von_mises`
3. Step through frames to observe ball approach, contact, and penetration
4. At frame ~52: first contact — watch stress ring appear
5. At frame ~77: ball exits — observe hole, petaling, debris
6. Add modifier: **Slice** to view cross-section of penetration channel
7. Use **Common Neighbor Analysis (CNA)** to identify disordered Al atoms around hole

---

## What to Look for in Results

### Stress Wave Propagation

At first contact (~frame 52), a circular compressive stress wave (red in OVITO) emanates radially outward from the impact point. The wave travels at the longitudinal speed of sound in Al (~6400 m/s) and reflects off the clamped edges as a tensile wave.

### Petaling and Plugging

Al atoms around the penetration channel are pushed radially outward, forming petal-like flaps — a classic ballistic penetration failure mode visible in the trajectory dump.

### Spallation

Negative hydrostatic pressure (tension) appears at the sheet back surface and periphery of the impact zone, indicating spallation — internal fracture driven by reflected tensile stress waves.

### Von Mises Stress Distribution

Peak Von Mises stress concentrates at the projectile-sheet interface during penetration. Post-impact, residual stress remains around the hole perimeter even after the ball has fully exited.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot open potential file CuAlW.txt` | Potential file not found | Place `CuAlW.txt` in run directory |
| Ball not moving | `frz_ball` fix not removed before impact | `unfix frz_ball` before setting velocity |
| `Number of element mappings does not match` | Wrong pair_coeff format | Use `pair_coeff * * CuAlW.txt Al W` |
| Ball passes through sheet with no interaction | Neighbor cutoff too small | Increase `neighbor 3.0 bin` |
| Atoms lost during impact | Atoms ejected beyond box | Extend box z below -60 |
| Sheet vibrates violently at start | Equilibration too short | Increase equilibration run steps |
| Hole closes after impact | Surface tension pulls edge atoms | Already using `boundary p p f` — correct |

---

## Extending the Model

| Extension | What to Change |
|-----------|----------------|
| Higher velocity (hypervelocity) | Change `v_impact` to -30 or -50 Ang/ps |
| Thicker sheet | Increase `sheet_lz`; adjust box z |
| Oblique impact angle | Set `velocity w_ball set vx vy vz` with non-zero x or y |
| Multiple projectiles | Define `ball2`, `ball3` regions; union into one group |
| Compute impact temperature | Add `compute temp_impact al_sheet temp` |
| Larger projectile | Increase `ball_r`; raise `ball_cz` accordingly |
| Different sheet material (Cu) | Change lattice, mass, and pair_coeff element mapping |
| Radial distribution function | Add `compute rdf al_sheet rdf 100` post-impact |

---

## Citation

If you use this code in your research, please cite:

```bibtex
@software{mishra_2026_ballisticpenetration,
  author    = {Mishra, A.},
  title     = {Molecular Dynamics Simulation of Ballistic Penetration of a
               Tungsten Projectile through a Thin Aluminum Sheet},
  year      = {2026},
  month     = {may},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20114336},
  url       = {https://doi.org/10.5281/zenodo.20114336}
}
```

Plain text citation:

> Mishra, A. (2026, May). *Molecular Dynamics Simulation of Ballistic Penetration of a Tungsten Projectile through a Thin Aluminum Sheet*. Zenodo. https://doi.org/10.5281/zenodo.20114336

---

## Author

**akshansh11**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:

- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:

- **Attribution** — You must give appropriate credit to akshansh11 and provide a link to this repository
- **NonCommercial** — You may not use the material for commercial purposes

Copyright 2026 akshansh11. All rights reserved for commercial use.
