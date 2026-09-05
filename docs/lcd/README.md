# ERC12864F7-4 LCD conversion: connector research

Verified against EastRising documentation downloaded from the [requested product page](https://www.buydisplay.com/1-8-inch-128x64-lcd-module-display-w-touch-panel-rgb-backlight-7-colors) on 2026-09-05. Copies are in `datasheets/`.

Three reusable connector symbols are in `bonanzaDisplay.kicad_sym` and instantiated as J16/J17 on `lcd-touch.kicad_sch` (sheet 2) and J18 on `touch-control.kicad_sch` (sheet 3). Open both through `bonanzaDisplay.kicad_sch`. The existing OLED circuit and PCB are retained. The LCD sheet now includes supply/bias circuitry and RGB current limiting/PWM switches. Touch readout is wired through U5 (TSC2007) to RP2354A I2C1 and IRQ. LCD MCU wiring, connector footprints, and PCB integration remain pending; the conversion is **not ready for manufacture**.

See [POWER.md](POWER.md) for the subsequent supply-rail research, internal LCD bias generator, sample backlight measurements, and implemented power circuitry.

## Connector selection

| Ref | Interface | Manufacturer connector | Contacts | Contact arrangement |
| --- | --- | --- | --- | --- |
| J16 | ERC12864F7-4 LCD | ER-CON30HB-1 | 30, 0.5 mm pitch | Horizontal SMT ZIF, bottom contact; 0.30 ± 0.05 mm FPC |
| J17 | ERC12864F7-4 RGB backlight | ER-CON2.0-4P-SMD | 4, 2.0 mm pitch | Horizontal SMD wire connector; numeric pad mapping unresolved |
| J18 | ER-TP018-1 resistive touch | ER-CON04HB-2 | 4, 1.0 mm pitch | Horizontal SMT ZIF, bottom contact; 0.30 ± 0.05 mm FPC |

The touch option must be selected when ordering; the product page offers both “No Touch Panel” and the resistive panel with connector. This is ER-TP018-1, not the differently shaped ER-TP018-2 intended for a 128×160 TFT.

Symbols model connector contacts, so their electrical pin types are passive. They contain connector part-number and datasheet properties. Footprints are deliberately blank: matching pitch alone does not establish compatibility with the existing OLED connector footprint. Confirm contact side, insertion direction, pin 1, mating cable, and mechanical layout before assigning footprints.

## LCD pinout (J16)

Source: [module datasheet](https://www.buydisplay.com/download/manual/ERC12864-4_Series_Datasheet.pdf), §4.1, pp. 9–10; cross-checked with the [interfacing schematic](https://www.buydisplay.com/download/interfacing/ERC12864-4_Interfacing.pdf).

| Pin | Symbol name | Function |
| --- | --- | --- |
| 1 | IRS | Internal/external voltage-regulator resistor selection |
| 2 | P/S | High = parallel, low = serial |
| 3 | C86 | Parallel interface selection: high = 6800, low = 8080 |
| 4 | VR | External contrast-divider input |
| 5 | V0 | LCD drive voltage |
| 6 | V4 | LCD bias voltage |
| 7 | V3 | LCD bias voltage |
| 8 | V2 | LCD bias voltage |
| 9 | V1 | LCD bias voltage |
| 10 | CAP2− | Charge-pump capacitor terminal |
| 11 | CAP2+ | Charge-pump capacitor terminal |
| 12 | CAP1+ | Charge-pump capacitor terminal |
| 13 | CAP1− | Charge-pump capacitor terminal |
| 14 | CAP3+ | Charge-pump capacitor terminal |
| 15 | VOUT | Charge-pump output |
| 16 | VSS | Ground |
| 17 | VDD | Logic supply |
| 18 | DB7/SI | Parallel DB7 / serial data input |
| 19 | DB6/SCL | Parallel DB6 / serial clock input |
| 20 | DB5 | Parallel data |
| 21 | DB4 | Parallel data |
| 22 | DB3 | Parallel data |
| 23 | DB2 | Parallel data |
| 24 | DB1 | Parallel data |
| 25 | DB0 | Parallel data |
| 26 | E / /RD | 6800 enable / 8080 active-low read |
| 27 | R/W / /WR | 6800 read/write / 8080 active-low write |
| 28 | A0 | High = display data, low = command |
| 29 | /RES | Active-low reset |
| 30 | /CS1 | Active-low chip select |

The manufacturer PDF has editorial errors: C86 low is described as “6800” on p. 9, whereas the interface reference explicitly identifies it as 8080. Pin 29 is printed “/RET” but described as /RES; the symbol uses /RES. The paragraph beside CAP pins on p. 9 describes serial data, not capacitor function. Capacitor pin names and the reference circuit were used to interpret these terminals. Older mirrored manuals can contain different or incorrectly extracted pin names; use the saved source revision.

The SPI reference uses P/S low, IRS high, VR open, DB0–DB5 grounded, and pins 26/27 high. It includes external bias and charge-pump capacitors plus supply decoupling; these are not included in the connector symbol. Read the reference circuit and controller documentation when designing that network. This is not a drop-in pin-compatible replacement for J1's SSD1322 OLED.

## RGB backlight contacts (J17)

Source: module datasheet p. 8, RGB-specific drawing dated Dec-15-2015. The cable contacts are labeled **R, G, B, A**. They are **not numbered** in that drawing. The [connector drawing](https://www.buydisplay.com/download/connector/ER-CON2.0-4P-SMD.pdf) also omits contact numbers and uses a generic illustration with more contacts than the four-contact dimensional table.

J17 therefore uses literal **R/G/B/A pin identifiers**, not an invented 1–4 mapping. On the module drawing, the order is A–B–G–R from the LCD FPC side toward the outside edge of the backlight cable. Viewing the mating face can reverse the apparent order. Before PCB integration, establish a numeric footprint mapping from an actual mating connector or a numbered manufacturer drawing; alternatively create a footprint with matching letter pad identifiers after checking orientation.

“A” suggests a common anode and R/G/B the color returns, but the saved module drawing does not include an RGB LED circuit or individual channel electrical ratings. Subsequent user bench tests confirmed common-anode polarity and the color mapping on the sample; see POWER.md. The product listing's 45 mA typical backlight current does not establish whether that is per-channel or combined. Do not derive RGB resistor values from the white-backlight example in the interfacing PDF or the generic backlight specification. Sample terminal voltages were measured at 1, 2, 5, and 10 mA; the user accepted 10 mA brightness. Manufacturer current limits remain unconfirmed.

See [TOUCH.md](TOUCH.md) for the implemented 3V0 TSC2007 interface on a separate I2C1 bus, measured electrode resistances, and remaining qualification checks.

## Touch pinout (J18)

Source: [ER-TP018-1 drawing](https://www.buydisplay.com/download/manual/ER-TP018-1_Drawing.pdf), Rev. A, Jul-31-2014. The table and circuit diagram agree:

| Pin | Electrode | Physical position, front view |
| --- | --- | --- |
| 1 | XL | Left |
| 2 | YD | Down |
| 3 | XR | Right |
| 4 | YU | Up |

These are raw resistive electrodes, not SPI or I²C signals. Preserve the manufacturer's direction names rather than assuming X+/X− conventions. A readout circuit must alternately drive one axis and measure the other. The drawing specifies a 3.0 V operating voltage. U5 now supplies electrode excitation from 3V0; the panel current note and touch protection still require qualification. GPIO26/27 now carry I2C1 to U5; GPIO24 receives touch IRQ. GPIO28/29 remain free.

## Saved source documents

| Document | Relevant information |
| --- | --- |
| [ERC12864-4_Series_Datasheet.pdf](datasheets/ERC12864-4_Series_Datasheet.pdf) | p. 8 RGB mechanical drawing; pp. 9–10 LCD pinout |
| [ERC12864-4_Interfacing.pdf](datasheets/ERC12864-4_Interfacing.pdf) | SPI/parallel reference circuits and boost/bias network |
| [ER-TP018-1_Drawing.pdf](datasheets/ER-TP018-1_Drawing.pdf) | Touch pin order and contact-side drawing |
| [ER-CON30HB-1.pdf](datasheets/ER-CON30HB-1.pdf) | LCD connector dimensions and bottom-contact construction |
| [ER-CON04HB-2.pdf](datasheets/ER-CON04HB-2.pdf) | Touch connector dimensions, pad 1 and bottom-contact construction |
| [ER-CON2.0-4P-SMD.pdf](datasheets/ER-CON2.0-4P-SMD.pdf) | Four-contact backlight connector dimensions; no numbering |

Connector PDFs are linked directly from the product page under its datasheet sections. Original URLs use `https://www.buydisplay.com/download/connector/` followed by the corresponding filename; touch and module PDFs use `/download/manual/`.

## Validation and next step

The supply and backlight implementation and its connectivity/ERC validation are documented in [POWER.md](POWER.md#schematic-implementation). Original root-sheet component connectivity remains unchanged. The LCD sheet now has C42-C53, R40-R48, and Q1-Q3, with one hierarchical supply connection to the existing 3V3 rail.

Next: choose SPI or parallel MCU wiring, assign RGB PWM GPIOs, resolve connector footprints/contact orientation, and retire the OLED circuit before updating the PCB. LCD supply-limit reconciliation and temperature/tolerance checks remain required before release.
