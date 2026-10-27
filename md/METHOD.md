# Method

Notation: positions are `(x, y, z)` in metres. Axis vectors are dimensionless. A
coordinate frame is `CS = (o, xa, ya, za)`: an origin and three axis vectors, all
given in the parent frame. The formulas assume finite, mutually perpendicular unit
axes.

## 1. Container

- A `.evo` file is a standard ZIP archive (`PK\x03\x04`). Entries are a mix of
  *stored* and *deflate* compression, so read entries through a ZIP library rather
  than seeking into the raw file
- The project record is the entry `Project/ProjectData/ProjectData.dat`. It is plain
  text in the ISO 10303-21 ("STEP physical file") syntax. Its header names a DIALux
  schema. No public EXPRESS schema for it is known to us
- Each data line is a record: `#N = Type(field, field, ...);`
- The companion entry `Project/ProjectData/ProjectData.xml` describes record types
  and field names and is useful when a field position is unclear

## 2. Record grammar

**Records.** Match `#N = Type( ... );` where the body may contain quoted strings
with semicolons or parentheses. Record ids are unique. A repeated id is an error.

**Field splitting.** Split the body on commas that are at parenthesis depth 0 and
outside quotes. Inside a quoted string, `''` is an escaped apostrophe and does not
end the string. Unbalanced parentheses or an unterminated quote make a record invalid.
The current reference parser warns and skips such records rather than always
stopping the conversion.

**References.** A reference is `#N` outside quoted strings. Remove quoted strings
before collecting references, because a `#123` inside text is not a link.

**Text.** Strip the outer quotes, replace `''` with `'`, and decode the standard
ISO 10303-21 escapes `\X2\hhhh...\X0\` (UTF-16BE) and `\X4\hhhhhhhh...\X0\`
(UTF-32BE). The other escapes of the standard (`\X\hh`, `\S\`) are not decoded by
this method. Such text is kept as it is stored. A stricter reader should report
them as an error instead.

**Frames.** A `CoordSys3D` record has exactly four fields, each a 3-tuple:
`(o, xa, ya, za)`.

**Two operations** are used throughout:

```
apply(CS, p) = o + p.x * xa + p.y * ya + p.z * za     (point: rotate and translate)
rot(CS, v)   =     v.x * xa + v.y * ya + v.z * za     (direction: rotate only)
```

**Frame composition.** Parent links come from relationship records
`RelAggregates` and `RelContainedInSpatialStructure`: the leading reference is
the parent, the remaining references are its children. If a child's local frame is
`L = (lo, lx, ly, lz)` and its parent's world frame is `P`, then

```
world(child) = ( apply(P, lo), rot(P, lx), rot(P, ly), rot(P, lz) )
```

applied recursively from the root. An object without a parent uses its local
frame as its world frame. A cycle in the parent chain is an error.

## 3. Luminaire chain (position and aim)

**Records involved.** Each luminaire is a `LuminaireElement`. Its fourth field
references its own `CoordSys3D`. Luminaires placed as an array (line, field,
circle) are children of a `LuminaireArrangement` through `RelAggregates`. The
arrangement has its own `CoordSys3D` in the world (or in its parent).

**Formula.**

For an arrangement member, `arrangementCS` is the arrangement's world frame after
its own parents have been applied. `elementCS` is the member's stored frame
relative to the arrangement. The arrangement frame is applied exactly once.

```
arrangement member:   world_position = apply(arrangementCS, elementCS.o)
                      axis_z         = rot(arrangementCS, elementCS.za)
single luminaire:     world_position = elementCS.o          (after parent composition)
                      axis_z         = elementCS.za
emission direction:   direction = -axis_z / |axis_z|
```

The local `+z` axis of a luminaire is the mounting normal (towards the ceiling),
so light leaves along `-z`: a downlight has `direction = (0, 0, -1)`. Tilted
accent and wall-washer luminaires keep their own tilt, because every element
carries its own frame. Their rotation about the emission axis is not carried into
Radiance (section 4), so an asymmetric distribution can end up turned the wrong
way. See section 8.

Each member's own frame already holds its unique offset inside the array (for
example, a 6 x 10 ceiling grid gives 60 different origins). The parametric array
records (line, field, circle position data) are therefore not needed to place the
members.

The origin of the element frame was observed to coincide with the centre of the
light-emitting surface to within about 1 cm (one project).

**Known pitfall and fix.** An earlier version of this method treated the
arrangement's frame as a *child offset* and added it to the member positions.
Symptoms: the position of one member doubles (it appears far above the ceiling),
the other members collapse onto the arrangement origin, and many luminaires share
one point. In one real project, 113 of 142 positions were wrong before the fix.
The correct rule is the one above: the arrangement frame is the parent frame, and
it is applied once to each member's own origin. After the fix, all positions in
that project were distinct and inside the room outlines.

A second pitfall: a luminaire subtree also contains other `CoordSys3D` records
(for example material anchor points on faces). Taking "any non-zero frame found
below the element" picks one of these and gives wrong positions. Use only
the frame in the element's fourth field.

## 4. Photometry chain

```
LuminaireElement
  --RelDefinesByPrototype-->            LuminairePrototype
  --PrototypeGeometricRepresentation--> LuminairePrototypeRepresentationData   ("equipment")
LightDistributionConnection(equipment, ..., LightDistribution)
  LightDistribution --> LightDistributionData   (C angles, gamma angles, candela values)
LampTypeChannel(equipment, ..., ..., lumens)    (luminous flux, fourth field)
```

- Verified photometry requires a unique element-to-prototype link and a unique
  prototype-to-equipment link. Multiple or conflicting known links are errors.
  Missing links can still permit luminaire placement, but without verified
  photometry. The converter reports unmatched photometry and approximate power
- `LightDistributionData`, fields 2, 3 and 4: C-plane angles (ascending, starting
  at 0), gamma angles (ascending, 0 to 180), and `nC * nG` candela values ordered by
  C plane. In the projects examined the last C plane was below 360 (for example
  345 or 357.5), and rotationally symmetric products had a single plane
- The method treats the table as candela per 1000 lm and scales it by the lamp
  flux: `cd_out = cd * lumens / 1000`. evo-builder's documentation states the
  same unit (cd/klm)
- **Flux check.** Here I is scaled `cd_out` in candela, rather than the
  stored cd/klm value. The flux of a table is the integral of I(C, gamma) sin(gamma)
  over gamma from 0 to pi and C from 0 to 2 pi (angles in radians), including the
  interval from the last C plane back to 0. A single plane is multiplied by 2 pi.
  Divided by the lamp flux, this gave, in the first project, a ratio of 1.00 for 7
  of 9 products (the other two were not explained).
  In a second project, measured on 2026-09-27, 6 of 11 products were within 1 % of
  1.00 and 5 of 11 were between 0.80 and 0.81. A ratio below 1 is what a light
  output ratio below 100 % would give, but this was not confirmed. The tables and
  the numerical steps of these measurements are not part of this package, so the
  ratios are reported observations, not checks a reader can repeat. They are
  consistent with the cd/klm reading. This is **not** a comparison with a DIALux
  calculation
- Missing lumens are an error. No default flux is assumed

**IES LM-63-2002 output** (one file per equipment record):

```
IESNA:LM-63-2002
[TEST] not available
[TESTLAB] not available
[ISSUEDATE] not available
[MANUFAC] not read from the project record
[_NOTICE] Photometric data belongs to its owner, normally the luminaire
[MORE] manufacturer. Written from a lighting project file to rebuild
[MORE] that project.
TILT=NONE
1 <lumens> 1 <nG> <nC> 1 2 <w> <l> <h>     lamps, lm/lamp, multiplier, counts, type C, metres
1.0 1.0 0.0
<gamma angles>
<C angles>
<one line of nG scaled candela values per C plane>
```

The luminous opening size `<w> <l> <h>` is not read from the project. The method
uses a fixed placeholder (0.3 x 0.3 x 0.1 m).

The first field of `LightDistributionData` (a symmetry marker) is not used, and
tables are not expanded: whatever planes are stored are written. A single stored
plane is read by LM-63 readers as a rotationally symmetric distribution. Tables
ending at 90 or 180 were not seen in the projects examined.

The C planes are written as stored. A closing plane at 360 is not added, although
LM-63 type C files normally end at 0, 90, 180 or 360. `ies2rad` accepted the files
written this way. Stricter readers may not.

**To Radiance.** `ies2rad` turns each IES file into a Radiance light source. Each
instance is placed with `xform`. `ies2rad` writes the source with its nadir on -Z
and its C0 plane on +X. The rotation turns the nadir to the emission direction and
the C0 plane to world +X projected perpendicular to the emission direction, or to
world +Y when the light is horizontal along X. It is written as three angles:

```
!xform -rx a -ry b -rz c -t x y z  <ies2rad output>
```

A downward luminaire therefore keeps its C0 plane on world +X. An earlier converter
version used `-ry beta -rz (phi - 180)`, which turned the C0 plane of every downward
luminaire to world -X. This was measured with an asymmetric test distribution in
Radiance. Rotation about the emission axis from the EVO frame is not carried over.

## 5. Room chain

**Variant A: polygon-based space.**

```
Space (fourth field: CoordSys3D)
  --> ... --> PolygonBasedSpaceRepresentationDataPart
                (owner, n, frame, height?, material, material, .Bottom., (PolyPoint2D refs), .Storey.)
PolyPoint2D --> (x, y)   floor outline in the space frame
```

The reference converter does not yet identify a verified height field for this
variant. It scans the top-level fields from left to right and uses the first
scalar number strictly between 1.5 and 30 as the height in metres. If none is
found, it assumes 3 m. An earlier numeric field, such as `n` when its value is in
that range, can therefore be mistaken for height. Every such room produces a warning identifying the inferred or default
height. Check this assumption against the source project before using results.

**Variant B: storey contour.** `RelAssociatesStoreyContourBasedSpace` links a
`Space` to one `StoreyContour`, whose `StoreyContourRepresentationData` holds the
2D outline. The height comes from the part record, or, when the part is marked
`.Storey.`, from the parent `Storey` (its `StoreyRepresentationData`). A missing
or non-identity sub-frame is rejected, because its meaning was not established.

**Formula** (both variants), with `S` the world frame of the space or contour and
`(x_i, y_i)` the outline, closing point removed:

```
floor_i   = apply(S, (x_i, y_i, 0))
ceiling_i = apply(S, (x_i, y_i, h))
floor     = polygon(floor_0 ... floor_n-1)
ceiling   = polygon(ceiling_n-1 ... ceiling_0)          (reversed order)
wall_i    = polygon(floor_i+1, floor_i, ceiling_i, ceiling_i+1)
```

**Face orientation.** In the project measured for this (nine rooms), the stored
outline ran counter-clockwise seen from above: with the order above every floor
faced up and every ceiling faced down, that is, into the room. The wall order
given here also faces into the room for such an outline. If an outline is stored
clockwise, all three face outwards. Check the sign of the outline area and reverse
the outline first.

An earlier converter version wrote walls in the other order,
`(floor_i, floor_i+1, ceiling_i+1, ceiling_i)`, so its walls did not face the same
way as its floors and ceilings. The reference code published with these notes
uses the order given above and turns a clockwise outline first. Radiance `plastic` surfaces are rendered from both
sides, so images were not affected. Mesh export, back-face culling and any
calculation that uses the surface normal are.

`PolyPoint3D` records in the same file are *not* room walls: they belong to
suspended ceilings and prismatic volumes.

**Default reflectances.** Surface colours are not read from the project. Radiance
`plastic` materials with photopic reflectance
`rho_v = 0.265 R + 0.670 G + 0.065 B` are used:

| Surface | R G B | rho_v |
| --- | --- | --- |
| floor | 0.33 0.22 0.13 | about 0.24 |
| wall | 0.52 0.49 0.44 | about 0.50 |
| ceiling | 0.75 0.75 0.73 | about 0.75 |

## 6. Result files (`.rsl`): structure observations, not decoded

Calculation results are stored under `Project/Results/<group>/` as `.rsl` files.
**These files are not decoded. No illuminance values are extracted by this
method.** What was observed:

- **Header.** Every `.rsl` file starts with a Boost.Serialization native binary
  archive header: an 8-byte little-endian length `22`, the text
  `serialization::archive`, then a 2-byte library version `20` (`0x14`), followed
  by type-size flags. Layout of the native archive was taken from the public Boost
  source code
- **Inventory per result group.** `Datameshcolorillums0.rsl` (false-colour
  surface mesh), `Dataillumqt0.rsl` (quantitative illuminance values)
  `...Toc.rsl` files (tables of contents), `Datavislightemsurf0.rsl` (visible
  luminous surfaces of luminaires), `Dataenvillum0.rsl` (environment
  illuminance), small id and string tables
- **Coloured mesh vs quantitative values.** The coloured mesh is a display layer
  (geometry plus colour). The quantitative values are in a separate file
- **Per-object framing.** After the header, the mesh file holds unaligned float32
  coordinates in metres, but runs of vertices are interrupted by per-object
  framing (version, tracking and class-id bytes). It is not a flat array
- **Same size, different content.** Mesh files of different result groups can have
  exactly the same size but different hashes, consistent with the same geometry
  carrying different colours
- **Candidate anchor.** In the native binary archive, a run of numbers may be
  written as an 8-byte count followed by the raw values. This was considered as an
  anchor for a schema-less scan. On a second project this signature was not found
  at the start of the candidate value runs, so it remains unconfirmed

On that second project the header was present in all 408 result files, but a 24-byte
vertex record stride was not confirmed.

## 7. Scene assembly in Radiance

| Part | Produced as | Radiance tools |
| --- | --- | --- |
| Room shells | `polygon` surfaces with `plastic` materials | (scene text) |
| Luminaires | `ies2rad` sources, placed and aimed with `xform` | `ies2rad`, `xform` |
| Scene | octree | `oconv` |
| Images and illuminance | rendering. Irradiance mode, then lux = 179 x (0.265 R + 0.670 G + 0.065 B) of the RGB irradiance | `rpict`, `rtrace`, `falsecolor`, `pcond` |

For interiors, use several ambient bounces (for example `-ab 5`). One bounce
underestimates interreflection.

## 8. Limits

- Observed with DIALux evo 5.13 only. The main derivation used one real project.
  Selected photometry checks and `.rsl` header observations were repeated on a
  second project
- Results have **not** been compared with DIALux calculation results
- Polygon-based room height is inferred from the first scalar between 1.5 and
  30, or defaults to 3 m. The field interpretation is unverified and reported
  as a warning. Check the resulting room geometry before calculation
- Light colour and colour temperature are not validated. A neutral white is used
- The candela scaling (per 1000 lm) is an interpretation, not a confirmed field
  definition
- The luminous opening size in the IES output is a placeholder. Illuminance far
  from the luminaire is hardly affected. Luminance in a viewing direction is
  intensity divided by the projected emitting area, so with a placeholder area it
  is wrong, and glare metrics (UGR, DGP) computed from these sources are not
  reliable
- Rotation about the emission axis is not carried into Radiance. The C0 plane is
  placed by the fixed rule in section 4. For asymmetric luminaires (wall washers,
  linear luminaires) the distribution can therefore be turned the wrong way. The element frame's x axis is a candidate for the C0
  direction. This was not implemented or verified
- IES files are written without a closing C plane at 360
- Wall faces written by an earlier converter version were not oriented consistently
  with floors and ceilings (section 5). The published reference code corrects this
- The record scan matches complete records only. In an earlier converter version a
  record cut off at the end of the text was skipped without an error, and the
  default limit of 4000 luminaires was applied without a report. The reference
  code published with these notes reports both
- Room surface reflectances are defaults, not project values
- `.rsl` result files and the binary scene graph (`Project/ScenegraphScene`) are
  not decoded
- Field positions were established on the observed version. Other versions may
  differ. Unexpected records should stop the conversion rather than be guessed

## Independence and trademarks

Independent work, not affiliated with, endorsed by or supported by DIAL GmbH.
DIALux is a trademark of DIAL GmbH. Product names identify only the
formats and programs discussed.
