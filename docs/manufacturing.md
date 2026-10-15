# Manufacturing bonanzaDisplay v3.1

[Back to the project README](../README.md)

## Fabrication rules

The project rules target a six-layer board with 1 oz outer copper and a
routed outline. The existing 0.1016 mm track/space limits are retained.
Do not select 2 oz copper or V-cut edges without reviewing these rules.

The project and `bonanzaDisplay.kicad_dru` enforce:

- Copper to routed edge: 0.20 mm.
- Hole to copper: 0.20 mm generally; component PTH holes: 0.28 mm,
  increased to 0.30 mm on inner layers.
- Hole to hole: 0.20 mm generally; component holes: 0.45 mm.
- Component PTH annular ring: 0.15 mm; different-net SMD pad clearance:
  0.15 mm.
- NPTH round holes: 0.50 mm minimum; NPTH slots: 1.00 mm minimum;
  plated slots: 0.35 mm minimum.
- Silkscreen clearance: 0.15 mm; text height/stroke: 1.00/0.15 mm.

Vias use 0.30 mm drills with 0.45 or 0.50 mm pads, or 0.25 mm drills
with 0.45 mm pads. The latter combination avoids JLCPCB's published
small-via surcharge; smaller pads with that drill do not. No microvias
or blind/buried vias are required.

Limits were checked against [JLCPCB's capabilities](https://jlcpcb.com/capabilities/Capabilities)
on 2026-10-14. The CAD stackup is nominal; confirm the standard fabrication
stackup and order options before purchase. These checks do not replace
the remaining via-in-pad, solder-mask, and assembly-output reviews.
See [issue #11](https://github.com/bitaxeorg/bonanzaDisplay/issues/11).

### Via-in-pad and assembly

Order this six-layer board with **epoxy-filled and copper-capped vias
(POFV)**. When checked on 2026-10-14, JLCPCB included this process without an extra
charge on its six-layer product; confirm the current quote and options. See
its [via-covering guidance](https://jlcpcb.com/help/article/pcb-via-covering)
and [free POFV announcement](https://jlcpcb.com/news/free-via-in-pad-6-20-layer-pcbs-pofv).
The intended process uses epoxy fill with a copper cap, not a requirement
for the separate copper-fill option. Tenting alone is not a substitute.

The board's through vias have 0.25 or 0.30 mm drills. Apply the filling
process to those vias, including vias within solder pads; leave component
insertion holes and mechanical holes open. Retain the component-pad
solder-mask openings over capped vias.

The 2026-10-14 local paste/mask review found no change needed at the
locations tracked in [issue #12](https://github.com/bitaxeorg/bonanzaDisplay/issues/12):

- U1 pad 61: five vias; nine rounded 0.91 × 0.91 mm paste openings,
  approximately 61% coverage of the 3.4 × 3.4 mm exposed pad, with
  0.22 mm webs between openings.
- J1 pads 31/32 and J5 pads 13/14: anchor-pad paste openings are present.
- R40 pad 2: one centered via; both resistor terminals retain matching
  paste openings.

Before releasing the order, confirm the filled-and-capped process in the
order details and review JLCPCB's final stencil/CAM and assembly previews
for these locations. The local review does not verify supplier changes
to stencil apertures or the final soldering result.

## Assembly outputs

Generate Gerbers, plated/non-plated drill files, a BOM, and a component
placement file from the same saved design revision. Keep their coordinate
origins consistent and check that the BOM and placement file contain the
same fitted references.

The current assembly selection, including encoder pull-ups R41–R43, contains
55 fitted components in 28 LCSC part groups, all placed on the bottom side.
The pull-ups use JLCPCB Basic part C25900 / UNI-ROYAL 0402WGF4701TCE
(4.7 kΩ, ±1%, 0402). Packages generated at hardware commit `0fab1ca` predate
these resistors and must be regenerated before ordering.
It excludes J2/J15 headers, test pads, and fiducials from the assembly tables.
The OLED module and host FFC cable are sourced separately. USB parts are
included; omitting them requires an explicit assembly-variant change.

Preserve and verify the project's Fabrication Toolkit placement corrections,
especially J1's pickup offset and U4's rotation offset. Inspect pin 1,
polarity, cable entrances, and component alignment in the supplier preview.
SW3 has SMD electrical contacts and NPTH mechanical holes but is marked as a
through-hole footprint; an SMD-only placement export would incorrectly omit it.

Some source manufacturer/MPN fields still need reconciliation with the
selected LCSC parts; see
[issue #13](https://github.com/bitaxeorg/bonanzaDisplay/issues/13).
Resolve conflicting metadata and verify the exact supplier selection before
ordering. Stock snapshots and local placement checks do not establish
order-time availability or supplier model alignment.

The final supplier preview and hardware/firmware validation remain tracked in
[issue #17](https://github.com/bitaxeorg/bonanzaDisplay/issues/17).
Regenerate the manufacturing package after any design, sourcing, or DNP change.
