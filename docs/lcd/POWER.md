# LCD power research

Researched 2026-09-05. LCD power and RGB driver circuitry is now added to the schematic. PCB wiring has not changed.

## Recommendation

Keep U4, the existing TPS62933 12 V-to-3.3 V buck. The module vendor specifies 3.3 V nominal LCD power and supplies a 3.3 V reference circuit using the controller's internal boost/regulator/bias circuitry. A separate 9 V regulator is not required. The OLED's 2.8 V LDO can be removed when that circuit is retired. The 3.0 V LDO is unnecessary for LCD logic and is retained to power U5 touch excitation/readout. User bench measurements support powering the RGB backlight from the existing 3V3 rail with independent current limiting and PWM, targeting approximately 10 mA per channel. Final component selection still needs tolerance and temperature margins.

## Existing board rails

Verified from a fresh KiCad netlist of `bonanzaDisplay.kicad_sch`:

| Rail | Source / present use | Proposed disposition |
| --- | --- | --- |
| 12V | BAP connector J5 and J2; feeds U4 and OLED J1 pins 3/29 | Keep as board input; do not connect directly to LCD logic or V0 |
| 3V3 | U4 TPS62933DRLR; RP2354A and both OLED LDO inputs | Keep; preferred nominal LCD supply |
| 3V0 | U2 XC6206P302MR; OLED circuitry, J1 pin 17 | Retain for U5 TSC2007 and local touch signal pull-ups |
| 2V8 | U3 XC6206P282MR; OLED J1 pin 24 | Not needed for this LCD |

The [TPS62933](https://www.ti.com/lit/ds/symlink/tps62933.pdf) is a 3 A-class buck IC. That rating is not a measured spare-current budget for this PCB: inductor limits, layout, thermal performance, and other loads still matter. The LCD logic's specified current is sufficiently small that there is no apparent need for a larger regulator. R12=30.9 kohm and R13=10 kohm imply approximately 3.272 V from the nominal 0.8 V feedback reference, before tolerances.

## LCD logic and glass drive

The [module datasheet](https://www.buydisplay.com/download/manual/ERC12864-4_Series_Datasheet.pdf), p. 11, specifies:

| Parameter | Operating specification |
| --- | --- |
| VDD relative to VSS | 3.0 V minimum, 3.3 V typical, 3.5 V maximum |
| V0 relative to VSS | 8.8–9.2 V, 9.0 V typical |
| LCD supply current at VDD=3.3 V | 230 microamps maximum as listed; excludes backlight |
| Logic absolute maximum | 3.6 V; not an operating target |

Power enters J16 pin 17 (VDD); pin 16 is ground. The approximately 9 V glass-drive voltage is generated inside the ST7565R using a switched-capacitor booster, internal regulator, and voltage followers. The followers create V1–V4. These are display bias nodes, not additional PCB supply rails.

The [vendor interfacing circuit](https://www.buydisplay.com/download/interfacing/ERC12864-4_Interfacing.pdf) shows a four-times boost arrangement at 3.3 V with:

- Three 1 uF flying capacitors: pins 10–11, 12–13, and 13–14.
- One 1 uF reservoir capacitor from VOUT (pin 15) to ground.
- Five 1 uF capacitors from V0–V4 (pins 5–9) to ground.
- 0.1 uF plus 10 uF supply decoupling.
- IRS high and VR open for internal contrast adjustment.

These reference values are now placed as C42-C52 on the LCD sheet. Select effective capacitance accounting for ceramic DC bias; 25 V-rated parts are a reasonable starting choice for boost/bias nodes. Keep loops short near the FPC connector. The controller must be initialized to enable booster/regulator/followers and establish the correct contrast voltage; fitting capacitors alone does not enable the supply. Do not supply the RGB LEDs from VOUT or V0.

The [ST7565R controller datasheet](https://www.buydisplay.com/download/ic/ST7565R.pdf), pp. 31–34, 45, 48 and 57, confirms this architecture. A copy is saved as `datasheets/ST7565R.pdf`.

### Voltage tolerances to resolve before wiring is finalized

There is a real documentation discrepancy: the module allows VDD up to 3.5 V, but the linked ST7565R v1.5 controller specifies an operating maximum of 3.3 V for VDD/VDD2 and 13.5 V for VOUT. Four-times boosting at 3.3 V gives an ideal 13.2 V; at 3.5 V it would reach 14.0 V. Thus the full module VDD range cannot simply be combined with four-times boosting without checking the controller limits.

Direct 3V3 power follows the module vendor's reference design and is the preferred baseline. Before release, check regulator tolerance and ripple, measure unloaded/startup VOUT, and confirm the supplied controller limits with EastRising. The board's nominal 3.272 V is close enough to 3.3 V that tolerances matter. If the stricter controller limit must be met without vendor clarification, consider a tightly controlled supply around 3.1–3.2 V and verify MCU signal levels, or adjust the shared rail after checking all its loads. This is a fallback, not evidence that a new regulator is inherently required.

Reusing the existing nominal 3.0 V LDO for LCD logic is less attractive: negative tolerance can put it below the module's 3.0 V minimum, and it reduces margin to the MCU's 3.3 V output levels. Three-times boosting is another possible configuration, but its approximately 9.9 V ideal output at 3.3 V leaves less headroom above the 9 V target; losses and cold-temperature contrast requirements must be checked before selecting it.

## RGB backlight

### Sample measurements and accepted brightness target

The user tested individual channels with a bench supply limited to 3.3 V and measured voltage across the backlight with a multimeter. Every measurement below reached the requested current in constant-current mode, and every channel lit. The tests confirmed the common-anode connection and color mapping on this sample.

| Channel current | Red voltage | Green voltage | Blue voltage |
| --- | --- | --- | --- |
| 1 mA | 1.828 V | 2.291 V | 2.530 V |
| 2 mA | 1.893 V | 2.395 V | 2.575 V |
| 5 mA | 2.057 V | 2.525 V | 2.667 V |
| 10 mA | 2.307 V | 2.748 V | 2.789 V |

The user accepted 10 mA as sufficient brightness; red is visually the dimmest but acceptable for now. Use approximately 10 mA per channel as the design target and independent PWM for dimming. Do not increase red current solely to match perceived brightness without confirming its rating. Three channels at that target imply approximately 30 mA total; simultaneous operation has not yet been reported as tested.

At 10 mA, nominal 3.3 V leaves 0.993 V (R), 0.552 V (G), and 0.511 V (B) for external current limiting and switching. At the board's calculated nominal 3.272 V, those margins become 0.965 V, 0.524 V, and 0.483 V. This supports the existing rail with resistors and low-side MOSFETs; an active current sink would need suitably low dropout. Final resistor values must account for the actual rail, switch drop, component tolerances, and temperature. No additional backlight regulator is presently indicated.

These are measured terminal voltages for one sample, not guaranteed LED junction forward voltages or manufacturer current ratings. They do not establish maximum safe current, lifetime, internal resistor values, or temperature/production variation.

### Additional source found in the follow-up search

An [older EastRising manual hosted by Display Future](https://www.displayfuture.com/Display/datasheet/monographic/ERC12864-4.pdf), Rev. 1.0 dated Jun-17-2012, explicitly includes ERC12864F7-4. Its p. 11 specifies **RGB backlight VLED = 3.3 V typical**, with no minimum/maximum voltage. The series-wide backlight current row gives **30/45/60 mA minimum/typical/maximum**; the absolute maximum is **75 mA**. It does not identify per-channel versus combined current. These rows were visually verified and the original saved as `datasheets/ERC12864-4_Series_Manual_2012.pdf`.

This supports evaluating the existing 3V3 rail first. It does not establish a direct connection without current limiting, built-in resistor values, or current-regulator headroom. The separate voltage rows for single-color product variants are not verified RGB channel specifications. This older copy also predates the 2015 RGB cable drawing revision in the current PDF, so sample validation remains appropriate.

The product page lists 45 mA typical backlight current. The series datasheet lists 75 mA absolute maximum without allocating current among RGB channels. Neither establishes per-color forward voltage/current, simultaneous-channel limits, or onboard current limiting. A/R/G/B labels suggest common anode, but the module-specific drawing lacks an LED circuit.

The manufacturer [8051 evaluation-board schematic](https://www.buydisplay.com/download/manual/8051_MCU_Board%20Schematic.pdf) contains a three-channel low-side transistor driver block. However, it is a generic board, labels that block “Not Use,” and does not provide module-specific LED ratings. It does not settle the question. A copy is saved as `datasheets/8051_MCU_Board_Schematic.pdf`.

| Option | When it works | Consequence |
| --- | --- | --- |
| Existing 3V3 plus independent RGB current limiting and PWM switches | Every channel has adequate voltage headroom at desired brightness across tolerances | No additional voltage regulator |
| New 5 V rail from 12 V plus RGB drivers | Blue/green need more headroom than 3V3 provides | Adds a regulator; a small buck is preferable if efficiency matters |
| Existing 12 V plus suitably rated current sinks or series resistors/switches | Verified LED topology and current limits permit it | No extra voltage rail, but considerably more heat and wasted power |

Do not select final resistors or a 5 V rail solely from typical LED-color assumptions. For a resistor-driven channel, use `R = (Vsupply - Vf - Vswitch) / I`, checking maximum current at highest supply/lowest Vf and brightness at lowest supply/highest Vf. Each color needs its own current limiting. PWM switches should default off during reset.

For scale only, if total LED current were 45 mA and average forward voltage 3 V, a linear drive from 12 V would dissipate approximately 0.405 W outside the LEDs, versus approximately 0.090 W from 5 V. These are illustrative assumptions, not confirmed ERC12864F7-4 ratings.

## Touch

The ER-TP018-1 is a passive four-wire resistive panel; its drawing specifies 3.0 V operation. It does not require an LCD-style bias regulator. U2 now supplies the TSC2007 (U5) on sheet 3 and its local signal pull-ups. U5 switches the electrodes and reports readings over I2C1 on GPIO26/27, with IRQ on GPIO24. See [TOUCH.md](TOUCH.md) for the measured 678/326 ohm axis loads and the unresolved current/POR qualification checks.

## Information still needed

Obtain from EastRising: guaranteed minimum/typical/maximum Vf and operating current for each color, maximum combined current, and whether any current limiting is built into the backlight. Sample common polarity and terminal voltages at 1–10 mA have now been checked, but guaranteed limits remain unknown. Also reconcile the module/controller supply limits. No messages have been sent to the manufacturer.

The research and sample measurements support retaining the main buck and using 3V3 for the backlight at approximately 10 mA per channel. The driver BOM remains to be selected and verified across operating margins.

## Schematic implementation

The LCD sheet now takes `3V3` through a hierarchical input connected to the existing root-sheet `3V3` net. J16 VDD (17), IRS (1), and J17 common anode A use this rail; J16 VSS (16) uses the existing global GND. VR (4) is intentionally marked no-connect for internal contrast regulation. Data/interface-mode pins remain pending.

| Components | Function / values |
| --- | --- |
| C42-C46 | 1 uF from V0, V4, V3, V2, V1 respectively to GND |
| C47 | 1 uF from VOUT to GND |
| C48 | 1 uF between J16 pins 10 and 11 |
| C49 | 1 uF between J16 pins 12 and 13 |
| C50 | 1 uF between J16 pins 14 and 13 |
| C51, C52 | VDD decoupling: 0.1 uF / 16 V and 10 uF / 10 V |
| C53 | 0.1 uF / 16 V local backlight supply decoupling |
| Q1-Q3 | AO3400A low-side switches, red / green / blue |
| R40-R42 | Red 100 ohm, green 56 ohm, blue 51 ohm; 1%, 0402, 0.0625 W |
| R43-R45 | 100 ohm series gate resistors |
| R46-R48 | 100 kohm gate-to-ground pulldowns |

C42-C50 specify 25 V X7R in 0805 footprints; exact capacitor MPNs must preserve adequate effective capacitance under DC bias. Supply capacitor footprints are 0402 for 0.1 uF and 0805 for 10 uF. Connector footprints remain unresolved.

The [AO3400A manufacturer specifications](https://www.aosmd.com/products/mosfets/low-voltage-mosfets-12v-30v/ao3400a) give maximum RDS(on) of 48 milliohms at VGS=2.5 V. Its drop at 10 mA is negligible relative to the series resistors. The symbol uses gate 1, source 2, drain 3 in SOT-23. `BL_R_PWM`, `BL_G_PWM`, and `BL_B_PWM` are local input-net labels awaiting MCU integration. High turns a color on; pulldowns establish off with the MCU undriven.

Using the measured 10 mA terminal voltages as fixed approximations, the selected resistors give R/G/B currents of 9.65/9.36/9.47 mA at the calculated 3.272 V rail, or 9.93/9.86/10.02 mA at 3.3 V. Actual current depends on the backlight I-V curve and temperature; these are design estimates, not guaranteed bounds. At 10 mA, resistor dissipation is 10/5.6/5.1 mW. Before release, verify supply extremes, brightness/current over temperature, and simultaneous RGB operation. Resistor limiting does not establish the manufacturer's unknown absolute current rating.

Validation: KiCad 10.0.4 exported both sheets and the netlist. Connectivity checks verified the shared supply, all boost/bias capacitors, independent RGB resistor/switch paths, gate pulldowns, and unique references. All original root-sheet component connectivity is preserved. Both sheet renders were inspected. ERC changed from 63 findings to 46: 24 retained existing findings, 19 unfinished LCD/touch contacts, and three PWM-label warnings awaiting MCU connections. The prior single-global-GND-label warning disappeared because the new sheet now uses GND. No new library mismatches were introduced.
