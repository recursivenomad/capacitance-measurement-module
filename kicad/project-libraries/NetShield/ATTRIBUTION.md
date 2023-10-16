# Net Shield library attributions




## Peter S. Hollander

Library created by Peter S. Hollander in 2023, and made freely available under the [*MIT-0*][URL-MIT-0] license.

*Contact: <recursivenomad@protonmail.com>*  
*Repository: <https://gitlab.com/recursivenomad/capacitance-measurement-module/-/tree/main/kicad/project-libraries/NetShield>*


## KiCad source

The creation of bezier curves within the schematic symbol through the direct editing of [`./Device_NetShield.kicad_sym`](./Device_NetShield.kicad_sym) was identified within KiCad's [`.../sch_sexpr_parser.cpp`][URL-KiCad-bezier], and the values for a bezier ellipse were derived from KiCad's [`.../test_ellipse_to_bezier.cpp`][URL-KiCad-ellipse].
The built-in Net Tie symbols & footprints were referenced for similarity.

&nbsp;




# Tools utilized

- [KiCad][URL-KiCad] - Schematic symbol & footprint design




[URL-KiCad]: <https://www.kicad.org/>
[URL-KiCad-bezier]: <https://gitlab.com/kicad/code/kicad/-/blob/00bf2ca36f8795f38905f3869fbc078b43cc07e6/eeschema/sch_plugins/kicad/sch_sexpr_parser.cpp>
[URL-KiCad-ellipse]: <https://gitlab.com/kicad/code/kicad/-/blob/00bf2ca36f8795f38905f3869fbc078b43cc07e6/qa/unittests/libs/kimath/geometry/test_ellipse_to_bezier.cpp>
[URL-MIT-0]: <https://opensource.org/license/mit-0/>
