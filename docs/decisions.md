# MAM decisions

**Audience:** coding agents and developers working on MAM.
**Describes:** the MAPL3 line, branch `release/MAPL-v3`.
**Verified against:** `MAM` @ `d9ca717` (2026-11-06).
**Placement:** as in the GEOSgcm fixture (`components.yaml` @ `f50622d`).
Other fixtures may place this repository elsewhere; paths inside this repository do not change.

### One source file, two component names
- Pattern: Chemistry adds `MAM` and `MAM.data` with the same `SetServices`; the component records `.data` in `data_driven` (`MAM_GridCompMod.F90`, `isDataDrivenGC_`, `SetServices`). Nothing else in the file reads it.
- Likely reason: the pattern was copied from GOCART so the data-driven twin could differ later. [Inferred]
- Question for maintainers: should `.data` change behaviour inside MAM?

### Scheme chosen at run time, modes fixed at build time
- Pattern: `scheme:` in `MAM_GridComp.rc` picks the scheme (only `MAM7` accepted); the CAM code is compiled with `MODAL_AERO_7MODE` (`MAM_GridCompMod.F90`, `SetServices`; `CMakeLists.txt`, `target_compile_definitions`).
- Likely reason: the CAM code uses compile-time mode counts. [Inferred]

### Ported CAM and MOSAIC code kept in `microphysics/`
- Pattern: CAM `modal_aero_*` and MOSAIC modules are compiled into the same library and called from `Run_` (`modal_aero_amicphys_intr`, `modal_aero_calcsize_sub`) (`CMakeLists.txt`, `srcs`).
- Likely reason: reuse of published aerosol code with a thin GEOS wrapper. [Inferred]

### Optics by lookup tables, attached as a method on `AERO`
- Pattern: `aerosol_optics` interpolates precomputed Mie tables (`MAML_OpticsInterpolate`) and is attached with `ESMF_MethodAdd(..., label='aerosol_optics')`; tables are made offline in `optics/` (`MAM_GridCompMod.F90`, `Initialize_`).
- Likely reason: avoid Mie computation at run time. [Inferred]
- Question for maintainers: which label should Radiation call (see the parent's aerosol optics label observation)?

### Aerosol tracers as friendly tracers in the internal state
- Pattern: interstitial number and mass fields have `FRIENDLYTO = 'DYNAMICS:TURBULENCE:MOIST'`, cloud-borne ones `'MOIST'` only, both with optional restart (`MAM_GridCompMod.F90`, `SetServices`).
- Likely reason: so the host components advect and mix them (cloud-borne ones only in Moist). [Inferred]

### `MAM_` modules over `MAML_` modules
- Pattern: `MAM_*` process modules `use` `MAML_*` modules (`MAM_DryRemovalMod.F90`, `MAM_SizeMod.F90`, `MAM_WetRemovalMod.F90`).
- Question for maintainers: what does `MAML` stand for, and are the unlisted `MAM_NucleationMod` / `MAM_CoagulationMod` / `MAM_GasAerosolExchangeMod` retired?
