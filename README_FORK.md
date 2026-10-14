# ModelMuse 5.4 — community bug-fix fork (unofficial)

This repository contains the source code of **ModelMuse 5.4.0.0** as released by the U.S. Geological
Survey (USGS), plus a small set of bug fixes. It is **not** an official USGS product and is not
endorsed by the USGS. The official ModelMuse is available at
<https://www.usgs.gov/software/modelmuse-a-graphical-user-interface-groundwater-models>.

The first commit is the unmodified USGS source (`ModelMuseSource5_4.zip`); every later commit is one
self-contained change with its rationale, so each fix can be reviewed (and upstreamed) on its own.

## Fixes

| Commit | Area | Problem in 5.4.0.0 | Effect |
|---|---|---|---|
| SFR2: keep kinematic routing when ISFROPT=0 | MODFLOW-2005/NWT export | `TRANSROUTE` written without `REACHINPUT`/`IRTFLG NUMTIM WEIGHT FLWTOL`; MODFLOW-NWT then ignores kinematic routing | Models silently ran without kinematic-wave routing (transient budget 1–5 % off) |
| MF2005/NWT import: SFR2 width/depth for ICALC=0 with ISFROPT>0 | MODFLOW-2005/NWT import | DEPTH only imported for ISFROPT=0 | ISFROPT>0 models got depth = 1 (stream leakage ×500 in a test model) |
| MF6 import: fix access violation importing any LAK package | MODFLOW 6 import | Local `TDis` used uninitialized in `ImportLak` | Every MF6 lake import failed; SFR/MVR/IMS after LAK were silently skipped |
| MF6 import: fix MVR imports | MODFLOW 6 import | `TDis` assigned too late in `ImportSfr`; SFR/LAK single-ID mover sources rejected by an assertion | SFR+MVR (or SFT) imports crashed; SFR→LAK movers never imported (also in 5.3.1) |
| NWT/MF2005 export: no 1.0 format on a missing exe | MODFLOW-2005/NWT export | Missing/mistyped MODFLOW-NWT location silently selected the 2014 input format | Exported files rejected by current MODFLOW-NWT (`ERROR CONVERTING "TABFILES"`) |

Build-only changes (no behavior change): dependency search path, optional madExcept, VirtualTreeView 8
`VirtualTrees.Types` imports, default platform Win64.

## Verification
Each fix was checked by round trips (flopy model → ModelMuse import → ModelMuse export → MODFLOW run →
comparison with the original run), with the original 5.4 source built the same way as the control:

- the control build reproduces the official 5.4 executable's behavior (same results, same bugs);
- with the fixes: MF6 LAK, WEL→SFR and SFR→LAK movers, and ModelMuse's own `LakSftCnc` example import
  and give heads identical to the original (≤ 1e-8 m); NWT SFR2 with kinematic routing is identical
  without any post-processing; all other tested packages are unchanged (CHD, GHB, WEL, RCH(A), DRN, RIV,
  EVT(A), HFB, UZF, MAW/MNW2, CSUB, LAK3, SFR6 with time series and observations, DIS/DISV).

## Building (Delphi 13 Community Edition)
Delphi Community Edition only compiles from the IDE. No packages need to be installed in the IDE:
the project's unit search path points at the third-party sources in `../deps`:

| Library | Version used | Notes |
|---|---|---|
| JCL | GitHub master (2026-09) | copy `jcl.template.inc` to `jcld37win32.inc` / `jcld37win64.inc`; init the `jedi` submodule |
| JVCL | GitHub master (2026-05) | |
| VirtualTreeView | V8.1.2 | |
| Abbrevia | GitHub master | `UnzipZipxSupport` disabled in `AbDefine.inc` |
| Graphics32 | 1.9.1 fixes r2148 | patches from the USGS readme (GR32.pas `hyp <> 0`, GR32_Compiler.inc) |
| GLScene | 1.8 (RX 10.3 archive) | `USE_GRAPHICS32` defined; `FGL <> nil` guard in GLContext.pas |
| madExcept | not used | optional; define `madExcept` to enable crash reports |

Open `ModelMuse/ModelMuse.dproj`, answer **Cancel** to the "Error reading form" dialogs (they come from
the form designer, not the compiler), select Release / Windows 64-bit and build (Shift+F9).

## License
ModelMuse is a work of the U.S. Government and is in the public domain in the United States. The
third-party libraries keep their own licenses (MPL 1.1/2.0 for JCL, JVCL, Graphics32, GLScene,
VirtualTreeView and Abbrevia).
