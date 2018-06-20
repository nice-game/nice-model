# Version 0.0 (WIP)

All pointers represent offsets relative to the beginning of the file.

Pointers *must not* be 0, unless they are marked with (opt), in which case 0 represents a null value.

Areas referenced by pointers *must not* overlap each other or the File Header.

All structures are tightly packed, so there can be no confusion about alignment.

## File Header

| Name | Data | Value |
| - | - | - |
| magic_number | [u8; 4] | b"nmdl" |
| version | Version | { major: 0, minor: 0 } |
| vertex_count | u32 |
| positions_offset | *[Position; vertex_count] |
| normals_offset | (opt) *[Normal; vertex_count] |
| texcoords_offset | (opt) *[TexCoord; vertex_count] |
| index_count | u32 |
| indices_offset | *[u32; index_count] |
| material_count | u8 |
| materials_offset | *[Material; material_count] |

Everything in this specification is subject to change, except for:
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

## Material
| Name | Data |
| - | - |
| index_count | u32 |
| texture1 | (opt) *Path |
| texture2 | (opt) *Path |
| light_penetration | u8 |
| subsurface_scattering | u8 |
| emissive_brightness | u16 |
| base_color | [u8; 3] |

`index_count` represents the number of indices this material will be applied to. It is cumulative, so if the first material has an `index_count` of 12, and the second material has an `index_count` of 9, the second material will be applied to indices 12..21.

`texture1` is interpreted as a texture where the first 3 channels define an RGB color value, and the 4th channel is a flag to enable or disable emissiveness. For the sake of performance, this *should* reference a DXT1 file.

`texture2` is interpreted as a texture where the first 2 channels represent a tangent-space normal encoded as `normal.xy / normal.z`, the 3rd channel represents a "polish" factor, and the 4th channel represents a "metallic" factor. For the sake of performance, this *should* reference a DXT3 file.

`light_penetration` represents the percentage of light that makes it through one unit of material. 0 represents an opaque object, and 255 represents a completely invisible object.

`subsurface_scattering` represents how wide of a cone each incoming ray spreads out into per unit distance traveled through the material. 0 represents no scattering, and 255 represents maximum scattering. The units here are arbitrary.

## Path
`String`

A path *must* be a valid path on any system where the file will be loaded. It *should* be cross-platform compatible, and relative instead of absolute.

## String
| Name | Data |
| - | - |
| length | u16 |
| bytes | [u8; length] |

`bytes` *must* be UTF-8 encoded, and does not have a null terminator. However, null characters are acceptable and treated as part of the string.
