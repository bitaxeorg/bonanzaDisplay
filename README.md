# bonanzaDisplay v3.1

## Development USB port

J6 is a development-only USB interface. Supply the board with 12 V through
J2 (pin 1: 12 V; pin 2: GND) when using USB for programming or debugging.
USB does not power the board.

The following limitations are intentionally accepted for this revision:

- **No VBUS sensing.** VBUS connects only to J6 and test point TP64. The
  RP2354A cannot detect when a connected host or hub removes VBUS. While
  the board remains powered from 12 V, its USB D+ attachment pull-up can
  remain enabled. This does not meet the USB requirement to remove the
  attachment pull-up when VBUS is absent and can source current into an
  unpowered host's data input. Disconnect the USB cable before shutting
  down the host/hub or disabling its port power while the board stays on.
  Normal USB suspend, with VBUS still present, is a different condition.
- **No external USB ESD protection.** The port relies on the MCU's internal
  protection. Use normal ESD precautions during development; the port has
  not been validated for exposed production use or system-level ESD
  immunity.

No USB power circuit, VBUS-sensing circuit, or external USB ESD array is
being added for this revision. This is an accepted development-port
limitation, not a claim of full USB compliance. Revisit these choices if
USB becomes a user-facing production interface.

## Field programming and assembly options

The intended field-update path is the ESP32 on bitaxeBonanza-1004x
programming the RP2354A through the 12-pin host connector J5:

| Display connector pin | Signal |
| --- | --- |
| J5 pin 11 | SWDIO |
| J5 pin 12 | SWCLK |

ESP32 SWD programming support and the complete field-update workflow still
require firmware implementation/validation.

A future production assembly variant may omit J6 and its USB-only
resistors R2, R4, R37, and R38. This is an option, not a change to the
current BOM or component DNP flags.

Decision recorded in [issue #14](https://github.com/bitaxeorg/bonanzaDisplay/issues/14).

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
(POFV)**. JLCPCB includes this process free on its six-layer product;
see its [via-covering guidance](https://jlcpcb.com/help/article/pcb-via-covering)
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
