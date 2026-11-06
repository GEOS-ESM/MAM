# MAM architecture

**Audience:** coding agents and developers working on MAM (Modal Aerosol Model).
**Describes:** the MAPL3 line, branch `release/MAPL-v3`.
**Verified against:** `MAM` @ `d9ca717` (2026-11-06).
**Placement:** as in the GEOSgcm fixture (`components.yaml` @ `f50622d`).
Other fixtures (for example GEOSadas, GEOSldas) may place this repository
elsewhere; paths inside this repository do not change.

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

Fixture path: `src/Components/@GEOSgcm_GridComp/GEOSagcm_GridComp/GEOSphysics_GridComp/@GEOSchem_GridComp/@MAM`.
Parent: the Chemistry gridded component, `../docs/ARCHITECTURE.md` (`../../docs/ARCHITECTURE.md` in the fixture) (on this line the parent's documents are not written yet; the MAPL2-line versions are on its `develop` branch); this document does not repeat how Chemistry adds, runs and connects its children.

## 1. Map of the repository

One gridded component, `MAM_GridCompMod`, wraps modal aerosol microphysics code ported from CAM (the Community Atmosphere Model) and MOSAIC (an aerosol chemistry code). All Fortran files are in one CMake library (`CMakeLists.txt`, `esma_add_library`).

| Path | Content |
|---|---|
| `MAM_GridCompMod.F90` | The gridded component (about 5,100 lines): `SetServices`, `Initialize_`, `Run_`, `Finalize_`, aqueous chemistry, diagnostics, and the two `AERO` methods. This is the GEOS interface layer. |
| `MAM_BaseMod.F90` | Scheme types (`MAM_Scheme`, `MAM_AerosolMode`, `MAM_AerosolSpecies`), `MAM_SchemeInit`, scheme ids `MAM3_SCHEME`, `MAM4_SCHEME`, `MAM7_SCHEME` (`MAM_BaseMod.F90`, header and public list). |
| `MAM3_DataMod.F90`, `MAM7_DataMod.F90` | Mode and species names and parameters of each scheme (header: "basic MAM3/MAM7 parameters and types"). |
| `MAM_ComponentsDataMod.F90`, `MAM_ConstituentsDataMod.F90` | Component (species) densities, constituent data. |
| `MAM_*Mod.F90` process modules | One per process, each called from `Run_`: `MAM_SizeMod` (dry and wet size), `MAM_SeasaltMod`, `MAM_DustMod`, `MAM_BlackCarbonMod`, `MAM_OrganicCarbonMod`, `MAM_SulfateMod` (emissions and some diagnostics), `MAM_DryRemovalMod` (sedimentation and deposition), `MAM_WetRemovalMod`. |
| `MAML_*Mod.F90` | Lower-level routines used by the `MAM_*` modules and by the optics: `MAML_SizeMod`, `MAML_SettlingMod`, `MAML_DryDepositionMod`, `MAML_DryRemovalMod`, `MAML_WetRemovalMod`, `MAML_OpticsTableMod` (reads Mie tables), `MAML_OpticsMod` (interpolation of optical properties). |
| `MAM_NucleationMod`, `MAM_CoagulationMod`, `MAM_GasAerosolExchangeMod`, `MAML_NucleationMod`, `MAML_CoagulationMod`, `MAML_GasAerosolExchangeMod` | Present in the directory but **not listed** in `CMakeLists.txt` `srcs` (see section 9). |
| `microphysics/` | CAM `modal_aero_*` files and MOSAIC `module_mosaic_*`, `module_data_mosaic_*` files, plus CAM support stubs (`abortutils`, `cam_logfile`, `chem_mods`, `constituents`, `infnan`). Directory level only. |
| `optics/` | Offline Mie lookup-table generation (Python, `mie.F90` via f2py, `miev/` Mie code) and tests (`optics/CMakeLists.txt`; `optics/tests`). Not part of the run-time library. |
| `mam_optics_calculator.{F90,py,csh,rc}` | Offline extinction calculator (section 6). |
| `MAM_GridComp.rc` | Private resource file of the component (section 5). |
| `MAM_Registry.rc` | Import, export (and an empty internal) spec for the ACG (Automatic Code Generator, `esma_generate_gocart_code`). |
| `MAM7_ExtData.yaml`, `MAM7_ExtData.rc` | ExtData (external data) entries for emissions, used by the experiment setup (section 4). |
| `README.md` (one line, "# MAM"), `README` (empty), `TODO` | Notes. `TODO` is a list of items dated 2015 (*Unverified:* current status). |

## 2. Purpose

MAM computes the life cycle of modal aerosol: emissions of sea salt, dust, black carbon, organic carbon and sulfate; aqueous chemistry; box-model microphysics (gas-aerosol exchange, rename, nucleation, coagulation); mode merging; dry and wet removal; and size and hygroscopic growth. It exports the aerosol tracers as friendly tracers (see 4), and an `AERO` state for Radiation and an `AERO_ACI` state for cloud microphysics (`MAM_GridCompMod.F90`, `SetServices`, `Run_`). Only the MAM7 scheme is selectable (section 5).

## 3. Place in the hierarchy

- Parent: Chemistry (`GEOS_ChemGridComp.F90`). Chemistry adds two children with the same `SetServices`: `MAM` when `ENABLE_MAM` is true and `MAM.data` when `ENABLE_MAM_DATA` is true (`../GEOS_ChemGridComp.F90`, SetServices). Chemistry runs `Run_` in one phase (parent document).
- Children: none. `SetServices` calls `MAPL_GenericSetServices` and registers no child.
- Connections made by Chemistry into MAM: `AIRDENS`, `DELP`, `CN_PRCP`, `NCN_PRCP` from `CHEMENV`; gas, aqueous-production and tendency fields from `ACHEM` when `ACHEM` is enabled (`SO2`, `H2SO4`, `NH3`, `SOA_GAS` renamed from `ACHEM::SOAG`, `pSO4_aq`, `pNH4_aq`, `DDT_*`, `_*_gas`, `_*_aq`); `ALK4`, `CH4`, `C2H6`, `C3H6`, `C3H8`, `ISOP` from `GMICHEM` (`../GEOS_ChemGridComp.F90`, SetServices). Other imports (surface and meteorological fields, `MAM_Registry.rc` `ImportSpec`) come through Chemistry's own imports.
- Consumers: `AERO` is read by Radiation, and `AERO_ACI` by Moist, when `AERO_PROVIDER` selects `MAM` (or `MAM.data`); the parent document records this, including the `ENABLE_AERO_ACI` condition (parent document).

## 4. Entry points

- **SetServices** (`MAM_GridCompMod.F90`): allocates the private `MAM_State` and stores it with `ESMF_UserCompSetInternalState` under `MAM_state`; sets `data_driven` from the component name (see 8); loads `MAM_GridComp.rc` into a private config; selects the scheme; adds timers; sets entry points `Initialize_`, `Run_`, `Finalize_` (`MAPL_GridCompSetEntryPoint`, single run phase); includes the ACG-generated `MAM_ImportSpec___.h` and `MAM_ExportSpec___.h`; calls `MAM_SchemeInit`; adds, in loops over modes and species, internal specs (`NUM_A_<mode>`, `<species>_A_<mode>`, cloud-borne `*_C_*`, `WTR_A_*`, `DGN_*`) and export specs (diagnostics per mode, per microphysics process `GAEX`, `RNAM`, `NUCL`, `COND`); adds exports `AERO` and `AERO_ACI` (`MAPL_StateItem`) and `AERO_DP` (`MAPL_BundleItem`, "not filled in by MAM, just a place holder"); reads the seven Mie table files named by `MAM7_AIT_OPTICS:` ... `MAM7_CDU_OPTICS:` from the global (universal) config into the module variable `MAM7_MieTable(7)`; then `MAPL_GenericSetServices`.
- **Initialize_**: `MAPL_GenericInitialize`; `extract_` (grid, sizes, time step); resolution-dependent `seasalt_femis` and `dust_femis` through `Chem_UtilResVal`; `microphysics_initialize` (registers and initializes the CAM modal-aerosol data: `modal_aero_register`, `modal_aero_initialize`); builds `MAPL_SimpleBundle`s over the internal state: `qa` (interstitial aerosols), `qc` (cloud-borne), `Da` (dry and wet diameters) and over the import state: `qg` (gases); fills `AERO` (section 7); fills `AERO_ACI`; sets the attribute `ScavengingFractionPerKm` on internal fields from `f_conv_scav_*` (zero for cloud-borne); reads the seven narrow-band tables named in `MAM_GridComp.rc` into `self%mie_*`.
- **Run_** (single phase). Returns immediately unless the MAPL run alarm rings. Order (`Run_`): clip negative mixing ratios of `qa`, `qc`, `qg` to `tiny`; partition aqueous-phase sulfate and ammonium production; `AqueousChemistry`; dry and wet size; microphysics core `modal_aero_amicphys_intr` (condensation, rename, nucleation, coagulation, each switched by a flag), then positivity fix; emissions (`MAM_SS_Emission`, `MAM_DU_Emission`, `MAM_BC_Emission`, `MAM_OC_Emission`, `MAM_SO4_Emission`), applied after microphysics so the pre-gas and pre-aqueous mixing ratios are available (comment in `Run_`); mode merging (`modal_aero_calcsize_sub`) if `mode_merging`; if `dry_removal`: sizes then `MAM_DryRemoval`; if `wet_removal`: `MAM_WetRemoval`; size and water update; diagnostics (`MAM_SS_Diagnostics`, `MAM_DU_Diagnostics`, `CIM_Diagnostics`, `SFC_Diagnostics`, `AOT_Diagnostics`).
- **Finalize_**: `MAPL_GenericFinalize`; destroys the four bundles and all Mie tables; deallocates the state.
- **`AERO` methods**: `aerosol_optics` (module routine, attached in `Initialize_`) and `aerosol_activation_properties` (see 7).

## 5. State

- **Imports**: `MAM_Registry.rc` `ImportSpec` (ACG-generated header) lists surface, meteorological and precipitation fields; gas imports are bundled as `MAM_GAS_SPECIES`.
- **Internal state**: created in `SetServices` by `MAPL_AddInternalSpec`, not from `MAM_Registry.rc` (its `InternalSpec` block is empty). Interstitial number and mass fields (`NUM_A_*`, `*_A_*`) are `RESTART = MAPL_RestartOptional` with `FRIENDLYTO = 'DYNAMICS:TURBULENCE:MOIST'`; cloud-borne fields (`NUM_C_*`, `*_C_*`) are `RESTART = MAPL_RestartOptional` with `FRIENDLYTO = 'MOIST'` only; `DGN_DRY_*` and `DGN_WET_*` are `MAPL_RestartSkip` with `FRIENDLYTO = trim(COMP_NAME)` (`MAM_GridCompMod.F90`, `SetServices`). *Inferred:* the `WTR_A_*` water fields were not read.
- **Exports**: ACG `ExportSpec`, per-mode diagnostics added in `SetServices`, and `AERO`, `AERO_ACI`, `AERO_DP`. Parent re-export and consumers: parent document.
- **Private state**: `MAM_State` (type in `MAM_GridCompMod.F90`): config, grid, the four bundles, scheme, flags, scavenging factors, narrow-band Mie tables. `MAM7_MieTable` is module-level, shared by all instances of the component in one process (`MAM`, `MAM.data`).
- **Restarts**: internal fields only; the parent template names the `MAM` restart files (parent document).

## 6. Configuration that changes structure

`MAM_GridComp.rc` (private; loaded by `SetServices` from the file name `MAM_GridComp.rc`):

| Key | Effect |
|---|---|
| `scheme:` | Default `MAM7`; the value is upper-cased and only `MAM7` is accepted, anything else raises `MAM_UNKNOWN_SCHEME_ERROR` (`SetServices`). `MAM_BaseMod` also defines `MAM3_SCHEME`, `MAM4_SCHEME`, and `MAM_SchemeInit` handles MAM3 and MAM7 only, but the gridded component never selects them. |
| `dry_removal:`, `wet_removal:`, `nucleation:`, `coagulation:` | Process switches (set in the file). `condensation:`, `rename:`, `microphysics:`, `mode_merging:` are read with default `.true.` but are absent from the file. |
| `seasalt_femis:`, `dust_femis:` | Six values by horizontal resolution. |
| `pom_oc_ratio:`, `f_conv_scav_*`, `f_wet_*` | Emission and scavenging parameters per mode. |
| `narrowband_optics_*`, `broadband_optics_*` | Per-mode Mie table paths (the shipped values are absolute paths in a user's home directory, *Inferred:* placeholders). Only `narrowband_optics_*` is read by the component. |
| `PRIMARY_AEROSOL_EMISSIONS:`, `EMISSIONS_*` tables | Mode, species, weight, width, size for primary emissions. *Inferred:* read by the emission modules (not checked). |

Global (universal) config keys `MAM7_*_OPTICS:` (seven; no default, so a missing key is an error through `__RC__`) give the Mie tables used by `aerosol_optics`. The component name (`MAM` or `MAM.data`) only sets `data_driven`, which nothing reads, so `MAM` and `MAM.data` run the same code; differences come only from what Chemistry connects and from ExtData (section 8). Parent switches: `ENABLE_MAM`, `ENABLE_MAM_DATA`, `AERO_PROVIDER` (parent document).

Build options: preprocessor symbols `GEOS5 MODAL_AERO MODAL_AERO_7MODE GEOS5_PORT` are set privately for the library (`CMakeLists.txt`, `target_compile_definitions`): the CAM code is built for 7 modes at compile time. *Inferred:* this is the build-time counterpart of `scheme: MAM7`.

## 7. The `AERO` and `AERO_ACI` states

- `AERO` (`Initialize_`): sets `implements_aerosol_optics_method` to true unconditionally; adds a field bundle `AEROSOLS` holding, per mode, `NUM_A_`, `WTR_A_`, `DGN_WET_` and `<species>_A_` internal fields (references, not copies); sets attributes `air_pressure_for_aerosol_optics` (`PLE`), `relative_humidity_for_aerosol_optics` (`RH`), `cloud_area_fraction_for_aerosol_optics` (empty), `band_for_aerosol_optics` (0), and names `EXT`, `SSA`, `ASY`; creates the `PLE`, `RH`, `EXT`, `SSA`, `ASY` fields in the state; attaches the method with `ESMF_MethodAdd(aero, label='aerosol_optics', userRoutine=aerosol_optics)`.
- `aerosol_optics` reads `band_for_aerosol_optics` and calls `MAML_OpticsInterpolate` per mode with `MAM7_MieTable(1..7)` to fill `EXT`, `SSA`, `ASY`.
- `AERO_ACI` (`Initialize_`): sets `implements_aerosol_activation_properties_method` true; adds per-mode activation-property fields; attaches `ESMF_MethodAdd(aero_aci, label='aerosol_activation_properties', userRoutine=aerosol_activation_properties)`. The nested routine `aap_` computes the properties.
- Label: MAM attaches `aerosol_optics`, not `run_aerosol_optics`. How this relates to the label Radiation executes is an observation of the parent document (`../../docs/ARCHITECTURE.md`, the parent's aerosol optics label observation); not repeated here.

## 8. The `.data` modifier

`isDataDrivenGC_` returns true when the component name ends with `.data`; `SetServices` stores it in `MAM_State%data_driven` (`MAM_GridCompMod.F90`). A search of the file finds no other use of `data_driven`: `Initialize_` and `Run_` do not branch on it. So in this version `MAM` and `MAM.data` run the same code; any difference comes from what the parent connects and from ExtData (parent document). (Contrast with GOCART's `isDataDrivenGC_`, cited in the parent document; not read here.)

## 9. Dependencies, traps, decisions, observations

**Dependencies.** `MAPL`, `Chem_Shared` (`Chem_UtilResVal`, `Chem_ConstMod`), `GMAO_mpeu`, `ESMF`, `NetCDF_Fortran` (`CMakeLists.txt`, `dependencies`). `optics/` builds `miev` (needs `OpenMP`) and, when `USE_F2PY` is set and F2PY3 is found, the f2py module `optics_` (`optics/CMakeLists.txt`). `CMakeLists.txt` also builds the executable `mam_optics_calculator.xx` (links the library), installs `*.rc` and `*.yaml` into `etc` and the two scripts into `bin`. `esma_generate_gocart_code(${this} -F)` generates the Fortran include files from `MAM_Registry.rc`. *Unverified:* MAM is written to the same ACG conventions as GOCART.

**`microphysics/`.** Two code families: CAM `modal_aero_*` (called by `Run_`: `modal_aero_amicphys_intr`, `modal_aero_calcsize_sub`; data `modal_aero_data`; initialization `modal_aero_initialize_data`) and MOSAIC `module_mosaic_*` / `module_data_mosaic_*` (listed in `CMakeLists.txt`; *Inferred:* called from `modal_aero_amicphys` for aerosol chemistry; not checked). `microphysics/chem_mods.F90`, `constituents.F90` stand in for CAM modules. *Unverified:* origin of each file.

**Offline tools.** `mam_optics_calculator.F90` is a program ("Extinction calculator", usage `mam_optics_calculator.xx`, configured by `mam_optics_calculator.rc`) using `MAM_BaseMod`, `MAML_OpticsTableMod`, `MAML_OpticsMod`. The `.py` and `.csh` scripts reference `ana_lpe.x`, `Chem_Aod3d.x`, `lcv2prs.x`, tools of other packages (*Inferred:* leftovers from another component). `optics/mam7-optics.lut.py` creates the lookup tables the component reads.

**MAM_ and MAML_ prefixes.** The `MAM_*` process modules `use` the `MAML_*` modules (for example `MAM_DryRemovalMod` uses `MAML_SettlingMod`, `MAML_DryDepositionMod`, `MAML_DryRemovalMod`). No header or build file says what the `L` stands for. *Unverified:* meaning of `MAML`.

**Known traps.**
1. `MAM` and `MAM.data` share code; the `.data` name has no effect inside the component (section 8).
2. The Mie tables are read from two places: `MAM7_*_OPTICS:` (global config, used for `AERO` optics) and `narrowband_optics_*` (private `MAM_GridComp.rc`, used by `AOT_Diagnostics`; the call passes `self%mie_*`). `broadband_optics_*` is not read by the component.
3. Emissions run after microphysics, and dry sizes are computed more than once per step (before microphysics, before dry removal, after wet removal).
4. `AERO_DP` is an empty placeholder.
5. `Run_` does nothing on steps where the run alarm is not ringing.
6. `TODO` says the number of levels is hardcoded to 72 in `modal_aero_amicphys.F90` (*Unverified:* current).

**Decisions** are in [`decisions.md`](decisions.md).

**Observations (noted, not fixed).**
1. `CMakeLists.txt` omits `MAM_NucleationMod`, `MAM_CoagulationMod`, `MAM_GasAerosolExchangeMod` and the `MAML_` nucleation, coagulation and gas-aerosol-exchange modules, and `MAM_GridCompMod` does not `use` them. They only use each other (`MAM_NucleationMod` uses `MAML_NucleationMod`, `MAM_CoagulationMod` uses `MAML_CoagulationMod`; the `MAML_GasAerosolExchangeMod` use in `MAM_GasAerosolExchangeMod` is commented out): an unbuilt, unreferenced cluster. *Inferred:* nucleation and coagulation are done by the CAM code, since `Run_` sets `amc_do_newnuc` and `amc_do_coag` from `nucleation` and `coagulation` and passes them to `modal_aero_amicphys_intr`. Intended?
2. `MAM_Registry.rc` header says "GEOS-Chem Grid Component Registry" and credits a GEOS-Chem import spec (copy-paste text).
3. `MAM_GridComp.rc` default Mie table paths point into a personal directory; `MAM7_ExtData.yaml` has three aviation black-carbon templates under `/home/adarmeno/...`.
4. `scheme:` accepts only `MAM7` though `MAM3` support code exists; `MAM4_SCHEME` is defined and `MAM4_DataMod` is commented out in `MAM_BaseMod`.
5. `ESMF_AttributeGet` for `asymmetry_parameter_of_ambient_aerosol` uses `RC=STATUS` where the neighbouring calls use `__RC__` (`Initialize_`): the status is not checked there.
6. `mam_optics_calculator.py` docstring names another program (`ana_lpe.x`, `Chem_Ext3d.py`).
7. Chemistry exports `AERO_ACI` only with `ENABLE_AERO_ACI`, which the parent document found undefined; MAM always fills its own `AERO_ACI`.

**Open questions for maintainers.** Is `MAM.data` meant to differ from `MAM` inside the component? Are the unlisted process modules (Observation 1) retired? Which label should `aerosol_optics` be (see the parent's aerosol optics label observation)?
