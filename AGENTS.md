# AGENTS.md - MAM (Modal Aerosol Model)

**Audience:** coding agents and developers working on MAM.
**Describes:** the MAPL3 line, branch `release/MAPL-v3`.
**Verified against:** `MAM` @ `d9ca717` (2026-11-06).
**Placement:** as in the GEOSgcm fixture (`components.yaml` @ `f50622d`): `src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/@GEOSchem_GridComp/@MAM`. Other fixtures may place it elsewhere.

**Build status on this line:** not built; not yet ported to MAPL3. On
`release/MAPL-v3`, `@GEOSchem_GridComp/CMakeLists.txt` calls
`esma_add_subdirectories(GOCART)` only, with the comment "Restore subdirs as
components are ported to MAPL3."; `MAM` appears only in a commented-out list of
subdirectories. The Chemistry top-level library (`GEOS_ChemGridComp.F90`, which
adds this component) is present but not compiled: the `esma_add_library` block
that builds it is commented out in `../CMakeLists.txt`. Build dependency
`Chem_Shared` (Chemistry `Shared` directory) is also not built on this line. The code in this repository is the MAPL2 code, kept for a later
port. It differs from the MAPL2 line only in the name of the constants module
(`use MAPL_Constants` instead of `use MAPL_ConstantsMod`, 14 files) and
trailing whitespace.

## What this is
One ESMF/MAPL gridded component, `MAM_GridCompMod`, for modal aerosol (MAM7 only). It is a child of the Chemistry component (added as `MAM` and `MAM.data`) and wraps CAM `modal_aero_*` and MOSAIC code in `microphysics/`. It provides the `AERO` and `AERO_ACI` states.

## Documents
| File | Content |
|---|---|
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | File map, entry points, state, configuration, `AERO` methods, traps, observations |
| [`docs/decisions.md`](docs/decisions.md) | Recorded design choices |
| Parent: `../docs/ARCHITECTURE.md` (Chemistry) | How Chemistry adds, runs and connects MAM |
| `TODO`, `CHANGELOG.md` | Dated 2015 to-do list; release notes (do not edit `CHANGELOG.md` for documentation) |

## Rules for agents
- Source of truth: `MAM_GridCompMod.F90` (interface), `MAM_GridComp.rc` (private config), `MAM_Registry.rc` (specs read by the ACG).
- Note bugs, do not fix them unless asked; record them under "Observations" in `docs/ARCHITECTURE.md`.
- Keep the documents current when behaviour changes; cite files and routines, not line numbers.
- `microphysics/` and `optics/` are ported scientific code: read headers only; avoid reformatting.
- `MAM` and `MAM.data` run the same Fortran; the `.data` name is stored but not used inside the component.
- Aerosol optics label here is `aerosol_optics` (see the parent's observation on the aerosol optics label).

## Build and test
- Not built on this line (see Build status). On the MAPL2 line it is built with the rest of GEOSgcm through the fixture's CMake, Chemistry adding the `MAM` subdirectory (`CMakeLists.txt`: library plus `mam_optics_calculator.xx`; `optics/` Python module only with `USE_F2PY`). No unit tests are run by CMake here (`optics/tests` holds standalone tests).
