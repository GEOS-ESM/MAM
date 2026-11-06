# AGENTS.md - MAM (Modal Aerosol Model)

**Audience:** coding agents and developers working on MAM.
**Describes:** the MAPL2 line, branch `main`.
**Verified against:** `MAM` @ `f651cb7` (2026-11-06).
**Placement:** as in the GEOSgcm fixture (`components.yaml` @ `0f2e0fb`): `src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/@GEOSchem_GridComp/@MAM`. Other fixtures may place it elsewhere.

## What this is
One ESMF/MAPL gridded component, `MAM_GridCompMod`, for modal aerosol (MAM7 only). It is a child of the Chemistry component (added as `MAM` and `MAM.data`) and wraps CAM `modal_aero_*` and MOSAIC code in `microphysics/`. It provides the `AERO` and `AERO_ACI` states.

## Documents
| File | Content |
|---|---|
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | File map, entry points, state, configuration, `AERO` methods, traps, observations |
| [`docs/decisions.md`](docs/decisions.md) | Recorded design choices |
| Parent: `../docs/ARCHITECTURE.md` (Chemistry; on its feature branch) | How Chemistry adds, runs and connects MAM |
| `TODO`, `CHANGELOG.md` | Dated 2015 to-do list; release notes (do not edit `CHANGELOG.md` for documentation) |

## Rules for agents
- Source of truth: `MAM_GridCompMod.F90` (interface), `MAM_GridComp.rc` (private config), `MAM_Registry.rc` (specs read by the ACG).
- Note bugs, do not fix them unless asked; record them under "Observations" in `docs/ARCHITECTURE.md`.
- Keep the documents current when behaviour changes; cite files and routines, not line numbers.
- `microphysics/` and `optics/` are ported scientific code: read headers only; avoid reformatting.
- `MAM` and `MAM.data` run the same Fortran; the `.data` name is stored but not used inside the component.
- Aerosol optics label here is `aerosol_optics` (see parent Observation 10).

## Build and test
- Built as part of GEOSgcm through Chemistry (`CMakeLists.txt`: library plus `mam_optics_calculator.xx`; `optics/` Python module only with `USE_F2PY`). No unit tests are run by CMake here (`optics/tests` holds standalone tests).
- Built with the rest of GEOSgcm through the fixture's CMake (Chemistry adds the `MAM` subdirectory).
