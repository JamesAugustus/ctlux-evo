# EVO lighting layout recovery for Radiance: method notes

[Turkish](Turkish/README.md)

These notes describe how to recover the lighting layout of a DIALux
evo `.evo` project (luminaire positions, aiming directions, photometry and room
shells) from the project file itself, and how to rebuild that layout as a
Radiance scene. The work is interoperability-oriented file format analysis: only saved project files were
inspected, and the program itself was not examined.
The notes state what could be read, what could not, and how this relates to other
public work.

Independent work, not affiliated with, endorsed by or supported by DIAL GmbH.
DIALux is a trademark of DIAL GmbH. The name identifies the application
that saved the source files.

## Scope

- Input: a DIALux evo `.evo` project file (observed with evo 5.13)
- Output: luminaire world positions and emission directions, photometry as IES
  LM-63 files (limits in METHOD.md, section 8), and room shells (floor, walls,
  ceiling), mapped to Radiance
- Not covered: calculation results (`.rsl`), the binary scene graph, furniture,
  textures, daylight. The converter also reads furniture. These notes are
  deliberately limited to the lighting layout and the room shell

## Summary

| Topic | Status |
| --- | --- |
| Container (ZIP) and project record (ISO 10303-21 text) | Described. Also described by earlier public work (see RELATED_WORK.md) |
| Record grammar: field splitting, references, text escapes | Described. Decoded escapes: `''`, `\X2\`, `\X4\` |
| Luminaire world position and emission direction | Described, including a known pitfall with arrays. The arrangement structure is also documented by evo-builder |
| Luminaire photometry (candela table + lumens -> IES -> Radiance) | Described as a data path, with limits (METHOD.md, section 8): the cd/klm scaling is an interpretation, the opening size is a placeholder, rotation about the light axis and the closing C plane are not written. The candela record layout is also documented by evo-builder. Not compared with DIALux results |
| Room shell (outline + height -> floor, walls, ceiling) | Described for two record variants |
| Result files (`.rsl`) | Structure observations only, **not decoded, no illuminance values extracted** |
| Binary scene graph (`ScenegraphScene`) | Not decoded |
| Light colour / colour temperature | Not validated. Neutral white is used |

## Documents

- [METHOD.md](METHOD.md): the method, with the formulas
- [RELATED_WORK.md](RELATED_WORK.md): other public work on the `.evo` format and
  how these notes differ
- [PROVENANCE.md](PROVENANCE.md): method scope and evidence limits
- [LICENSE](LICENSE): MIT OR Apache-2.0
- [CITATION.md](CITATION.md): how to cite these notes

## Status

Method notes. The working Python converter is in
[ctlux-core](https://github.com/JamesAugustus/ctlux-core) (MIT OR Apache-2.0).
From the core repository root, its EVO command is
`python3 -B -m core input.evo output`. No sample project files, photometric
data or DIALux application files are included here.

## Development note

The analysis and the findings are the author's. AI tools helped with wording and
with keeping the text organised.
The main derivation used one real project. Selected photometry checks
and `.rsl` header observations were repeated on a second project. The
underlying project files are not distributed (see METHOD.md, section 8).

## Licence scope

Copyright (C) 2026 James Augustus

All original content in this repository, including method notes, documentation and any example code, is offered under **MIT OR Apache-2.0**. You may choose either [MIT](LICENSE-MIT) or [Apache 2.0](LICENSE-APACHE). The conditions of your chosen licence apply. Compliance with both is not required

Both options permit commercial use and distribution in closed products. Neither requires publication of source code or private modifications

- Under MIT, include the copyright and permission notice in all copies or substantial portions of the Software
- Under Apache 2.0, give recipients the licence, mark modified files with prominent change notices and preserve the relevant source notices as required by section 4
- Under Apache 2.0, include the relevant [NOTICE](NOTICE) attribution in a location allowed by section 4(d), such as a distributed NOTICE file or documentation
- Apache NOTICE requirements do not apply when MIT is chosen

Both options preserve applicable copyright notices. Neither requires advertising credit, a dedicated user interface credit or academic citation

Third party quotations, code and trademarks retain their own terms and are not relicensed by this notice. Only rights held in the original material are granted. Ideas, methods and file format facts do not become exclusive copyright under these licences. For independent implementations using only those ideas or facts, scholarly citation is voluntary and requested in [CITATION.md](CITATION.md)
