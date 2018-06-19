# Version 0.0 (WIP)

All pointers represent offsets relative to the beginning of the file, and *must not* be 0, unless they are marked with (opt).

Areas referenced by pointers *must not* overlap each other or the File Header.

## File Header

| Name | Data | Value |
| - | - | - |
| magic_number | [u8; 4] | b"nmdl" |
| version | Version |
| vertex_count | u32 |
| positions_offset | *[Position; vertex_count] |
| normals_offset | (opt) *[Normal; vertex_count] |
| texcoords_offset | (opt) *[TexCoord; vertex_count] |
| index_count | u32 |
| indices_offset | [u32; index_count] |

Everything else in this specification is subject to change, except for:
* The value of `magic_number`
* The position and size of `version`

Any changes to this specification will be accompanied by a change to the value of `version`.

## Version
| Name | Data |
| - | - |
| major | u16 |
| minor | u16 |

The version number *may* be ignored, but it is provided to allow parsers to easily maintain compatibility with older files when breaking changes are made to the format.

The major number represents changes that require adding a new branch the parser. The minor number represents changes that only require adding to the existing branch, and will still allow the branch to parse files from an earlier version.

Example 1: If a field is defined as "any u32 except 0", and then the standard is changed to allow 0, only the minor version should be incremented, because the updated parser still be able to parse a file that never uses the value 0.

Opposite of example 1: If a field is defined as "any u32", and then the standard is changed to disallow 0, the major version should be incremented, because the updated parser should fail when the value 0 is used, which causes it to be incompatible with older files. A new branch will need to be created to avoid losing support for the older files.

## Position
`[f32; 3]`

## Normal
`[f32; 3]`

Normal values *should* be normalized. That is, `sqrt(normal[0]^2 * normal[1]^2 * normal[2]^2)` should be roughly equal to 1.

## TexCoord
`[f32; 2]`
