# Cantilever Bracket — SolidWorks Simulation Design Iterations

Three configurations of a cantilever bracket, each analysed in SolidWorks Simulation. Configuration 2 was an attempt to lighten the solid baseline; configuration 3 re-architected the arm as a triangulated truss after configuration 2 showed a severe stress concentration and an unusable mesh.

All numeric values below were verified against the text extracted from the six original `.docx` study reports in this repository. Values are reported with the units used in those reports.

> **Scope:** simulation-only study. No physical test, no mesh-convergence study, and no native SolidWorks CAD source file is present. See [Limitations](#limitations).

---

## Design evolution

|  | Configuration 1 — solid baseline | Configuration 2 — slotted | Configuration 3 — triangulated truss |
|---|---:|---:|---:|
| Geometry | solid box-section arm | lightening slots in the web | top/bottom chords with diagonal bracing |
| Mass (kg) | 406.23 | 321.07 | 269.82 |
| Reaction force, Y (N) | 5,534.24 | 4,699.04 | 4,196.02 |
| Peak von Mises stress (MPa) | 6.35 | 26.77 | 10.85 |
| Max displacement (mm) | 0.122 | 0.301 | 0.216 |
| Mesh nodes | 1,457,934 | 10,445,156 | 458,961 |
| Mesh elements | 942,318 | 6,889,296 | 241,739 |
| Max aspect ratio | 4.3427 | 40,657 | 9.2901 |
| Mesh time | 00:01:02 | 00:44:51 | 00:00:35 |
| Factor of safety on yield | 97.6 | 23.2 | 57.2 |

Derived comparisons (from the table above):

- Configuration 1 → 2 removes 20.96 % of the mass but multiplies peak stress by 4.22 and displacement by 2.47.
- Configuration 2 → 3 removes a further 15.96 % of the mass, reduces peak stress by a factor of 2.47 and displacement by 28.2 %; the mesh is also dramatically cheaper (35 s versus 44 min 51 s).
- Configuration 3 is 33.58 % lighter than the baseline while retaining a factor of safety of 57.2.

The qualitative statements in earlier documentation about *where* the hot spot sits (slot end in configuration 2, mount in configuration 3) are not stated in the report text; only peak-stress node identifiers are recorded, so those location claims remain unverified from the available evidence.

---

## Shared model definition

These settings are recorded identically in all six study reports (`config_1_parent/`, `config_2_slotted/`, `config_3_truss/`):

| Item | Value |
|---|---|
| Material | Alloy Steel, linear elastic isotropic |
| Young's modulus | 210 GPa |
| Poisson's ratio | 0.28 |
| Density | 7,700 kg/m³ |
| Yield strength | 620.4 MPa |
| Fixture | `Fixed-1`, one face, Fixed Geometry |
| Applied force | `Force-1`, 500 N normal force on three faces |
| Gravity | `Gravity-1`, 9.81 m/s² on the body |
| Remote mass | `Distributed Mass-1`, 5 kg on two faces |
| Mesher | Blended curvature-based, 16 Jacobian points |
| Static solver | Automatic; thermal on, large displacement off, inertial relief off, soft spring off |

The fixture face, loaded faces and remote-mass faces are identified only by SolidWorks selection IDs in the reports; no coordinate-system figure or face map is included, so an independent re-creation of the exact load case is not possible from this repository alone. The reported reaction force is dominated by self-weight rather than the 500 N applied load, so the three configurations are not a perfectly clean like-for-like structural comparison.

---

## Studies by configuration

### Configuration 1 — Parent Model (`config_1_parent/`)

- `config_1_parent/static_1_report.docx` — Static study. Mass 406.23 kg, reaction 5,534.24 N (Y), peak von Mises 6.35 MPa, displacement 0.122 mm. Mesh 1,457,934 nodes / 942,318 elements, max aspect ratio 4.3427, 1 min 02 s to mesh.
- `config_1_parent/geometry.jpg`, `config_1_parent/von_mises.jpg`, `config_1_parent/displacement.jpg`, `config_1_parent/static_1_image.png` — original plots exported from the study.

The parent geometry is heavily over-designed for the load case; most of the material carries little stress.

### Configuration 2 — Slotted (`config_2_slotted/`)

- `config_2_slotted/static_2_report.docx` — Static study. Mass 321.07 kg, reaction 4,699.04 N, peak von Mises 26.77 MPa, displacement 0.301 mm. Mesh 10,445,156 nodes / 6,889,296 elements, max aspect ratio 40,657, 44 min 51 s to mesh.
- `config_2_slotted/geometry.jpg`, `config_2_slotted/von_mises.jpg`, `config_2_slotted/displacement.jpg` — original plots.

The slotted variant reduces mass but produces the highest stress and the worst mesh in the study. With an aspect ratio of 40,657 and 0.0876 % of elements above aspect ratio 10, its stress result should not be treated as converged or trustworthy.

### Configuration 3 — Triangulated truss (`config_3_truss/`)

Four studies were run on this configuration.

**Static** — `config_3_truss/static_3_report.docx`: mass 269.82 kg, reaction 4,196.02 N, peak von Mises 10.85 MPa, displacement 0.216 mm. Mesh 458,961 nodes / 241,739 elements, max aspect ratio 9.2901, 35 s to mesh, three mesh controls. Factor of safety 57.2 on yield.

**Frequency (modal)** — `config_3_truss/frequency_report.docx`: FFEPlus solver, three modes. Mode 1 = 31.883 Hz with Z-direction participation 41.558 %; mode 2 = 59.586 Hz with Y participation 8.84 %; mode 3 = 81.906 Hz with Y participation 41.75 %. Sum of Z participation = 41.558 %. Mesh as for Static 3, 37 s to mesh.

**Modal time history** — `config_3_truss/modal_time_history_report.docx`: FFEPlus, 15 modes, 0–1 s at Δt = 0.01 s. Base excitation is an acceleration with components `(0, 0.5, −10) in/s²` referenced to the top plane — note the mixed inch-based unit as written in the report. Maximum stress at 1 s is 53.22 MPa and maximum displacement is 0.788 mm. Reported reaction force is 267,298 N resultant.

**Linear dynamic (harmonic)** — `config_3_truss/dynamic_1_report.docx`: FFEPlus, 15 frequencies over 0–100 kHz, logarithmic spacing. Maximum stress at 100 kHz is 14.73 Pa and displacement 1.007e−7 mm — both negligible, and the very high sweep range is not justified by the 31.9 Hz first mode. Mode 1 here is 31.903 Hz with Z participation 41.549 % (the source of the rounded "31.9 Hz / 41.6 %" headline).

**Topology discrepancy:** `config_3_truss/dynamic_1_report.docx` describes **two** solid bodies (Mirror7, 68.455 kg, plus Cut-Extrude2, 201.365 kg, totalling 269.82 kg), whereas Static 3 describes a **single** Mirror7 body of 269.82 kg. The harmonic study is therefore not the same feature tree as the static study, despite the shared total mass.

`config_3_truss/geometry.jpg`, `config_3_truss/von_mises.jpg`, `config_3_truss/displacement.jpg`, `config_3_truss/mesh.jpg` and `config_3_truss/static_3_image.png` are the original plots for this configuration.

---

## Engineering trade-offs

- **Mass vs stiffness:** removing web material in configuration 2 cut 21 % of mass but nearly tripled displacement. Moving that material into chords and diagonals (configuration 3) recovered stiffness while removing another 16 % of mass.
- **Mass vs stress:** the slot ends acted as stress concentrators; the truss layout brought peak stress back down close to baseline levels while retaining most of the mass saving.
- **Computational cost:** configuration 2's mesh took 44 min 51 s with over 10 million nodes; configuration 3 achieved a better aspect ratio with 459k nodes in 35 s. Mesh quality, not just mesh size, dominated cost and reliability here.
- **Dynamic behaviour:** the final configuration's first mode is near 32 Hz with about 41.6 % vertical participation. The harmonic and transient studies exercise that geometry but do not provide a comparative dynamic study across all three configurations.

---

## Limitations

1. **No mesh-convergence study.** Only single-mesh results and mesh-quality metrics are available; configuration 2's mesh is pathological (aspect ratio 40,657).
2. **No physical testing.** All values are simulation outputs; no strain-gauge, deflection or modal test data exist.
3. **Native CAD is missing.** No `.sldprt`, `.sldasm` or `.slddrw` file is present. Every report references the same local path, `C:\Users\asus\Documents\Solidworks assignments\Chair\Cantilever Bracket.SLDPRT`, which is not portable and cannot be reproduced here.
4. **Boundary-condition detail is incomplete.** Face IDs, coordinate orientation, remote-mass coupling and mesh controls are not fully documented.
5. **Mixed units in the base-excitation definition** (inch-based acceleration in an otherwise SI model).
6. **Configuration drift** between Static 3 and the harmonic study (one body versus two bodies).
7. The 0–100 kHz harmonic sweep is not tied to the measured first mode, and the modal time-history run covers only 1 s.

---

## Repository layout

| Path | Contents |
|---|---|
| `config_1_parent/` | Baseline geometry plots, static report and plot export |
| `config_2_slotted/` | Slotted geometry plots and static report |
| `config_3_truss/` | Truss geometry/mesh plots, static, modal, time-history and harmonic reports |
| `README.md` | This document |

## Source material

Six SolidWorks Simulation study reports (`.docx`) and ten exported plot images are preserved in the configuration folders. No component datasheet, CAD source file or experimental record is included. The design-report path embedded in the original `.docx` files is documented above but is not available in this repository.
