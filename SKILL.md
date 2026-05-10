---
name: gfdl-fv3
description: >
  Progressive-disclosure skill for the GFDL Finite-Volume Cubed-Sphere
  Dynamical Core (FV3), the atmospheric dynamical core used in NOAA's
  operational GFS/UFS, AM4/CM4/ESM4, SHiELD, and SPEAR. Covers the
  GFDL_atmos_cubed_sphere repo: the dycore Fortran code, GFDL Microphysics,
  driver layer, and the FV3 scientific reference. Foundation for AM4 and
  for UFS/GFS work.
version: 0.1.0-scaffold
tags:
  - earth-science
  - atmospheric-dynamics
  - dynamical-core
  - fv3
  - cubed-sphere
  - finite-volume
  - gfdl
  - noaa
  - ufs
---

# GFDL FV3 Dynamical Core Guide

> **FV3** = GFDL Finite-Volume Cubed-Sphere dynamical core, used by NOAA operational and research models.
> Maintainer: NOAA / GFDL
> Source: https://github.com/NOAA-GFDL/GFDL_atmos_cubed_sphere
> Website: https://www.gfdl.noaa.gov/fv3/
> Docs / refs: https://www.gfdl.noaa.gov/fv3/fv3-documentation-and-references/
> Skill author: Koutian Wu (ktwu01@gmail.com)
> Skill version: 0.1.0-scaffold

**What FV3 does:** A non-hydrostatic finite-volume atmospheric dynamical core on a cubed-sphere grid. Solves the Euler equations with vertically Lagrangian remapping. Scales from regional storm-resolving (a few km) to global climate (~50–100 km). Used in: GFS (operational global forecast), HRRR-style applications via UFS, AM4 (GFDL atmosphere), SHiELD (storm-resolving variant), SPEAR (seasonal prediction).

**Where it lives:** This repository (`GFDL_atmos_cubed_sphere`) holds the dycore source plus GFDL Microphysics. Driver-level code that wires FV3 into a full atmospheric model lives in the consumer (e.g., `NOAA-GFDL/atmos_drivers`, UFS atmosphere, or the host GCM).

**Who this skill is for:** Researchers and developers working with FV3 directly, porting it, modifying numerics, adding tracers, debugging instabilities, understanding the dycore output.

---

## Quick Decision Tree

```
"What do I need?"
│
├─ 🆕 What is FV3 and how does it fit into AM4/SHiELD/UFS?
│  └─ Read: reference/overview.md
│
├─ 🌐 Cubed-sphere grid layout, tile numbering, halos
│  └─ Read: reference/grid.md
│
├─ ⚙️ Numerics: vertical remapping, advection schemes, divergence damping
│  └─ Read: reference/numerics.md
│
├─ ☁️ GFDL Microphysics
│  └─ Read: reference/microphysics.md
│
├─ 🛠️ Compiling FV3 in a host (AM4 / SHiELD / UFS)
│  └─ Read: reference/build.md
│
└─ 🐛 Instabilities, blowups, CFL violations
   └─ Read: reference/debugging.md
```

---

## Repo Layout (verified from clone)

```
GFDL_atmos_cubed_sphere/
├── docs/                    # FV3 scientific reference (LaTeX + PDF), tracer howto
│   ├── doc_source/          # LaTeX sources
│   ├── examples/            # Notebooks
│   └── fv3_technical_2021.pdf
├── driver/                  # Driver-layer Fortran (host hooks)
├── GFDL_tools/              # GFDL-specific helpers
├── model/                   # Core dycore Fortran (fv_dynamics, sw_core, tp_core, ...)
├── tools/                   # Pre/post utilities
├── README.md
└── RELEASE.md               # 202411 release notes
```

The cloned tag here is the **202411** FV3 release.

---

## Critical Rules

1. **FV3 is a dycore, not a full model.** It needs a host: AM4 driver, SHiELD, UFS atmosphere, or your own coupling. See `driver/` for the host interface.
2. **Vertical Lagrangian remapping is core.** Levels move with the flow during dynamics, then remap back to reference levels at the end of each `n_split` cycle (not every acoustic substep). Misconfigured remap intervals cause noisy fields.
3. **Three nested timescales.** Physics step `dt_atmos`, then `n_split` dynamics steps per physics step, then `k_split` acoustic sub-steps per dynamics step. Total acoustic steps per physics step = `n_split * k_split`. Get these wrong and you violate CFL or burn cycles. Vertical Lagrangian remapping is performed at the end of each `n_split` cycle (not every acoustic substep).
4. **Cubed-sphere has 6 tiles.** Halo regions across tile edges need special treatment; routines like `mpp_update_domains` handle this when called correctly.
5. **Cite Putman & Lin (2007) and Harris & Lin (2013)** when describing the FV3 dycore. Cite Chen et al. (2013) and Zhou et al. (2019) when using GFDL Microphysics.

---

## Reference Index

| File | Topic |
|---|---|
| reference/overview.md | FV3 in AM4/SHiELD/UFS |
| reference/grid.md | Cubed-sphere, tiles, halos |
| reference/numerics.md | Lagrangian remap, advection, damping |
| reference/microphysics.md | GFDL MP scheme |
| reference/build.md | Building FV3 in a host model |
| reference/debugging.md | Instabilities, CFL, blowups |

## Critical agent gotchas (Gemini-reviewed)

- **FV3 cannot be built standalone.** It depends on **GFDL FMS** (see fms-skill). Cloning `GFDL_atmos_cubed_sphere` alone is insufficient; the host (UFS, AM4, SHiELD) provides the FMS link.
- **Dual mode dycore.** FV3 supports both **non-hydrostatic** (default for high-resolution / weather) and **hydrostatic** (`hydrostatic=.true.`, common for AM4/CM4 climate). Pick deliberately.
- **Required inputs.** A grid_spec / mosaic file describing the cubed-sphere geometry, plus an `input.nml` with the FV3 namelist block. Without these, FV3 will not initialize.
- **HRRR is WRF-based, not FV3.** The UFS-based successor regional forecast is **RRFS**, not HRRR.
- **Foundational citations:** Putman & Lin 2007 and Harris & Lin 2013 for the dycore; Lin & Rood 1996, 1997 for the underlying flux-form transport; Chen et al. 2013 and Zhou et al. 2019 for GFDL Microphysics.

## Status

Scaffold (v0.1.0-scaffold). Source-grounded layout verified, with Gemini critique pass on 2026-05-09 to correct sub-stepping definitions and add FMS dependency. Operational depth being filled in.
