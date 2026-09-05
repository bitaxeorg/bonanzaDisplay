# Resistive touch interface

Implemented in `touch-control.kicad_sch` (sheet 3), with RP2354A and supply connections on the root sheet. The touch sensor is separate from the ST7565R LCD and RGB backlight. Sources reviewed: the saved ER-TP018-1 drawing, [RP2350 datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf) (GPIO function table, digital IO specifications, ADC and E9 erratum), [TI TSC2007 product page](https://www.ti.com/product/TSC2007), and [TSC2007 datasheet](https://www.ti.com/lit/ds/symlink/tsc2007.pdf), saved as `datasheets/TSC2007.pdf`.

## Implemented architecture

U5 is a TSC2007IPWR (TSSOP-16) between J18 and the RP2354A. Its electrode drivers, ADC and touch detection replace discrete electrode switching and MCU ADC sequencing. Retain U2's 3V0 supply for the touch controller and local bus pull-ups. This avoids assuming that the panel's documented 3.0 V operating point permits direct excitation by 3.3 V GPIOs. The drawing does not establish 3.0 V as an absolute maximum either.

Before this change, a schematic netlist confirmed that GPIO26/27 and GPIO24 were unused. They now connect to U5. GPIO20/21 connect to DISP_SDA/DISP_SCL on J5 and correspond to I2C0. Allocate I2C1 to the local touch interface, leaving the host bus independent:

| Signal | U1 GPIO | U1 package pin | TSC2007 TSSOP pin |
| --- | --- | --- | --- |
| TOUCH_SDA | GPIO26, I2C1 SDA | 40 | 11 SDA |
| TOUCH_SCL | GPIO27, I2C1 SCL | 41 | 12 SCL |
| TOUCH_IRQ_N | GPIO24, input | 36 | 10 PENIRQ |

This consumes ADC-capable GPIO26/27 as digital pins; GPIO28/29 remain available for analog work. GPIO18/19 remain available for a later LCD SPI0 clock/data assignment. The touch assignments are now wired; the LCD SPI assignment remains provisional.

## Sensor and support circuit

| J18 pin / electrode | Controller connection |
| --- | --- |
| 1 XL | X−, pin 4 |
| 2 YD | Y+, pin 3 |
| 3 XR | X+, pin 2 |
| 4 YU | Y−, pin 5 |

The chosen polarities aim for increasing X to the right and Y downward. Final screen orientation is set by calibration; verify it with corner presses.

Connect controller VDD/REF pin 1 to 3V0 and GND pin 6 to ground. Tie address pins A0 (14) and A1 (13) low for address 0x48. Leave NC pins 7/8/9/15 unconnected; ground unused AUX (16). C54/C55 provide local 0.1 uF / 1 uF decoupling (10 V X7R). R49/R50 are 4.7 kohm SDA/SCL pull-ups to 3V0; verify rise time at the selected bus speed. R51 adds a 10 kohm pull-up on the buffered interrupt output to the same 3V0 rail, without loading the sensor electrodes directly. PENIRQ is buffered; do not treat its internal sensor pull-up as an external open-drain bus pull-up.

The RP2354A recognizes 3.0 V as high (VIH minimum 2.0 V at 3.3 V IOVDD). Disable its internal 3.3 V pull-ups on this local I2C bus; use open-drain I2C operation, never push-pull highs. This keeps the controller's signals within its own supply domain without level shifters. Verify startup states and U2 output tolerance. Check U2's capacitor/stability requirements when adding local capacitance.

Keep electrode routes short and away from LCD charge-pump nodes and PWM switching. Assess connector ESD protection with low-leakage parts; avoid arbitrary large filter capacitors that slow electrode settling.

## Direct ADC alternative

GPIO26-29 can alternatively alternate between electrode drive and ADC measurement. To read X, drive XL low/XR high and sense a Y electrode with the other Y connection high impedance. To read Y, drive YU low/YD high and sense an X electrode. Disable unused pulls and digital input buffers while sampling; allow settling after changing drive direction. Add touch detection, filtering and calibration in firmware.

However, with the MCU on 3V3, U2 alone does not provide 3V0 GPIO drive. Discrete switches or an analog switch network would be needed to drive from 3V0 unless the vendor confirms 3.3 V operation. The panel's end-to-end resistance is also needed to assess GPIO loading. On affected A2 silicon, E9 requires particular care with digital input enables during analog use; A3 fixes this issue. This does not prevent direct ADC use, but makes it less attractive than a dedicated controller for this board.

## Firmware outline

Bring up local I2C1 at 100 kHz, then consider 400 kHz after checking bus timing. Use 12-bit coordinates, read X/Y and optional contact-resistance data while touched, debounce/filter samples, calibrate against displayed targets, and map into 128×64 coordinates. A provisional 50–100 coordinate updates per second is ample for buttons and dragging. Restore pen-detect mode after sampling and suppress conversion-related IRQ transitions. Touch is single-point; do not infer multitouch gestures.

## Measurements and remaining qualification

The panel drawing gives sheet resistance (500 ohms/square on the glass), not guaranteed end-to-end electrode resistances. It also includes an unexplained “DC3V 1mA” note. Do not treat that as a clear typical or absolute maximum drive-current specification.

The user measured 678 ohms across XL–XR (J18 1–3) and 326 ohms across YD–YU (2–4). At 3.0 V these imply approximately 4.42 mA and 9.20 mA respectively while each axis is driven, before controller switch resistance. These exceed the unexplained 1 mA note, so the user authorized provisional schematic implementation while that specification remains unresolved. Brief sampling lowers average current, but does not establish compliance with an instantaneous limit. Cross-layer isolation has not been reported as measured.

TSC2007 also specifies power ramp and off-time conditions for reliable POR. Check U2 startup and rapid power cycling against datasheet p. 31; if these cannot be met, use a controller with an appropriate reset mechanism or add controlled power cycling. This is a qualification item for TSC2007, not proof that the existing regulator satisfies its POR requirements.

## Schematic changes and validation

J18 moved from sheet 2 to the new sheet 3, retaining its reference, symbol UUID, connector metadata and electrode pin identifiers. U5 uses the standard `Driver_Display:TSC2007xPW` symbol with the TSC2007IPWR value and TSSOP-16 footprint. C54 is 0402, C55 is 0603, and R49-R51 are 0402. The exact passive MPNs remain to be selected; J18's connector footprint remains to be verified.

Hierarchical ports carry 3V0, TOUCH_SDA, TOUCH_SCL and TOUCH_IRQ_N between sheet 3 and the root. The existing U2 output supplies 3V0. The three relevant U1 no-connect markers were removed. All other existing component connectivity, including the LCD/backlight power circuit, OLED circuit and BAP host bus, is preserved.

KiCad 10.0.4 exports all three sheets and the netlist. Netlist assertions verified every electrode, U5 supply/address/AUX connection, three MCU signals, signal pull-ups, unconnected NC pins, unique references, and unchanged unrelated nets. ERC fell from 46 to 42 findings, removing exactly the four former J18 unconnected-pin errors and adding none. The touch sheet has no ERC findings. Remaining project findings include unfinished LCD data/mode and RGB PWM connections plus pre-existing issues. Sheet renders were inspected. PCB layout and firmware are not changed.
