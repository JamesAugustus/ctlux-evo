# Related work

Addresses and published code were checked on 2026-09-27. Declared licences were
checked on 2026-09-28. Repository creation dates below are context, not proof
of who first interpreted a record. Other work may have been missed.

## General format specifications

- **PKWARE APPNOTE** describes the ZIP container used by `.evo` files
  <https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT>
- **ISO 10303-21:2016** describes the clear-text exchange syntax used by the
  project record. It does not define the meaning of DIALux-specific fields
  <https://www.iso.org/standard/63141.html>

## Public work on DIALux formats

- **EvoParse** (GitHub user luciodias), created 2026-06-26. Its README states MIT, but
  the repository has no licence file.
  <https://github.com/luciodias/EvoParse>
  Reads the `ProjectData` record of `.evo` / `.dat` files as ISO 10303-21 (STEP)
  text and turns it into typed Python models generated from `ProjectData.xml`

- **evo-reader** (GitHub user abubakr3800), created 2026-09-15, no licence declared.
  <https://github.com/abubakr3800/evo-reader>
  Reads `.evo` files without DIALux. From the STEP project record it recovers
  luminaire instances (position from each instance's own `CoordSys3D`, product
  identity by following references) and, on a best-effort basis, rooms. From
  `.rsl` result files it recovers candidate illuminance values by statistical
  (heuristic) search of the binary data. Per-luminaire maps are estimated with a
  point-source model

- **evo-builder** (GitHub user abubakr3800), created 2026-09-21, no licence declared.
  <https://github.com/abubakr3800/evo-builder>
  Writes `.evo` files from Python and documents the format: the ZIP layout, the
  STEP project record, `ProjectData.xml`, the `ScenegraphScene` binary file (as a
  Boost.Serialization archive) and the `.rsl` result files. Its arrangement module
  documents that each luminaire element of a field arrangement carries its own
  `CoordSys3D` holding its offset from the arrangement origin. Its photometry
  module documents the `LightDistributionData` layout (C planes, gamma angles,
  intensity values in cd/klm, ordered by C plane)

- **BHoM DIALux_Toolkit** (BHoM), 2019 to 2024.
  <https://github.com/BHoM/DIALux_Toolkit>
  Exchanges data with DIALux through its STF interchange format, not through `.evo`
  files

## Context and credit

EvoParse was used to check the STEP project record against a saved file.
The public source review also identified evo-reader and evo-builder. The
evo-builder project documents the arrangement frames and `LightDistributionData`
layout discussed in METHOD.md, sections 3 and 4. evo-reader explores project
and result records. Their work is credited where descriptions overlap.
This review is not exhaustive and does not establish priority.

## Scope alongside public work

As far as we could see in the published code and documents on 2026-09-27:

1. **Direction of use.** These notes read a project in order to rebuild it as a
   Radiance scene and recompute the lighting. evo-builder writes projects for
   DIALux. evo-reader displays results that DIALux already computed
2. **World placement when reading.** The composition rule
   `world = apply(arrangementCS, elementCS.o)` and the emission direction
   `-rot(arrangementCS, elementCS.za)`. Both are written out as formulas, with the
   frame each one uses
3. **A reader-side pitfall.** Treating the arrangement frame as a child offset
   doubles or stacks positions (113 of 142 wrong in one project). The notes give
   the symptom and the rule that fixed it
4. **Photometry to Radiance.** The record chain from a luminaire to its candela
   table and lamp flux, the LM-63 output, and placement of `ies2rad` sources with
   `xform`. evo-builder converts in the other direction (LDT/IES files into the
   project record)
5. **Room shells to Radiance.** Two room record variants turned into floor, wall
   and ceiling polygons
6. **Result files.** These notes hold structure observations only and **do not
   extract illuminance values**. evo-reader recovers candidate values
   statistically. evo-builder documents `.rsl` files and includes analysis
   scripts. The three descriptions have not been compared in detail

## Code

No source code from the cited projects is included here. The Python
converter is maintained separately in ctlux-core.
