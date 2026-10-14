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
