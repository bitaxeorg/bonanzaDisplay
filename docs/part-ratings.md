# Selected-part ratings review

Reviewed 2026-10-19 for the v3.1 design, including the 750 kΩ R35 change.
Manufacturer, PARTNO and MPN fields now agree with the selected LCSC codes
for all 55 fitted components. Capacitor Voltage fields contain the actual
selected-part ratings; displaced design targets are retained in hidden
`Previous voltage requirement` fields. Resistor power, voltage and tolerance,
and capacitor dielectric and tolerance, are also recorded in the design.
No LCSC selections, footprints or circuit values were changed by this metadata
cleanup. The earlier R35 substitution is preserved.

**The capacitor voltage ratings pass the steady-state checks below, but the
power-component review is not an unconditional pass.** C8/C36 and C14 need
attention for DC-bias loss; L1 needs saturation and magnetic-orientation
qualification. L2 and the output capacitor need validation against the actual
load and fault conditions.

## Assumptions

- 12 V nominal input; 13.2 V (+10%) used as an engineering check, not a verified
  specification for the host supply. Hot-plug/transient peaks are not included.
- Logic-capacitor checks use 3.3 V nominal and 3.6 V for margin. Core capacitors
  use 1.1 V nominal. Firmware-selected core overvoltage needs a separate check.
- Samsung DC-bias results below are interpolated from manufacturer typical
  curves, at 25 °C and the stated measurement conditions. They are not guaranteed
  minimum capacitances; tolerance, temperature, aging and AC amplitude also matter.
- X5R capacitors have an 85 °C upper temperature classification. Check local
  component temperatures in the assembled product; resistor power derates at
  elevated temperature too.

## Capacitors

| References | Selected MPN / LCSC | Actual ratings | Circuit check |
|---|---|---|---|
| C8, C36 | Samsung CL21A106KAYNNNE / C15850 | 10 µF ±10%, 25 V, X5R, 0805 | 13.2 V is 53% of rated voltage. **Only about 1.88 µF typical at 12 V and 1.67 µF at 13.2 V.** |
| C14, C15, C18, C21, C24 | Samsung CL05A475MP5NRNC / C23733 | 4.7 µF ±20%, 10 V, X5R, 0402 | Voltage passes. About 2.43 µF typical at 3.3 V, 2.24 µF at 3.6 V, and 4.25 µF at 1.1 V. |
| C16, C17 | Fenghua 0402CG150J500NT / C1548 | 15 pF ±5%, 50 V, C0G, 0402 | Ample oscillator voltage margin; C0G avoids the substantial DC-bias loss of high-capacitance X5R parts. Crystal startup remains a hardware check. |
| C19, C20, C23, C25, C27, C28, C30–C33, C35, C37, C41 | Murata GRM155R71H104KE14D / C77020 | 100 nF ±10%, 50 V, X7R, 0402 | Ample voltage margin on logic/core and 12 V rails. C41's rating applies across BST–SW, not BST–ground. Exact DC-bias curves were not evaluated for this group. |
| C22 | Samsung CL05A105KA5NQNC / C52923 | 1 µF ±10%, 25 V, X5R, 0402 | OLED internal VDD is about 2.5 V; approximately 0.82 µF typical at 2.5 V. Voltage passes; nominal capacitance follows the existing application circuit. |
| C26 | Samsung CL31B475KBHNNNE / C51205 | 4.7 µF ±10%, 50 V, X7R, 1206 | Using VCOMH ≤0.86 × VCC gives 11.35 V at 13.2 V input; approximately 3.65 µF typical. Ample voltage margin. |
| C29 | Yageo CC0402KRX7R9BB333 / C106862 | 33 nF ±10%, 50 V, X7R, 0402 | Low-voltage soft-start node; voltage passes and nominal value exceeds TI's 6.8 nF minimum. |
| C34 | Samsung CL21A226MAQNNNE / C45783 | 22 µF ±20%, 25 V, X5R, 0805 | About 14.90 µF typical at 3.3 V and 14.03 µF at 3.6 V. Voltage passes; assess effective total output capacitance and load transients. Manufacturer lists this part as NRND. |

Primary curve sources: Samsung pages for
[C8/C36](https://product.samsungsem.com/mlcc/CL21A106KAYNNN.do),
[4.7 µF logic/core capacitors](https://product.samsungsem.com/mlcc/CL05A475MP5NRN.do),
[C22](https://product.samsungsem.com/mlcc/CL05A105KA5NQN.do),
[C26](https://product.samsungsem.com/mlcc/CL31B475KBHNNN.do), and
[C34](https://product.samsungsem.com/mlcc/CL21A226MAQNNN.do).
The local review evidence includes the manufacturer's embedded DC-bias data.

C8 is the TPS62933 local input capacitor; C36 is the OLED 12 V reservoir.
Their voltage rating alone does not establish adequate ripple filtering or
transient response. Evaluate larger-capacitance or less bias-sensitive parts
against required effective capacitance and available footprint space.

C14 supplies VREG_VIN. The RP2350 guide calls for at least 4.7 µF input
capacitance, so its roughly 2.4 µF typical at operating bias deserves attention.
C15/C24 operate at the lower core voltage and retain much more capacitance.
Nominal component recommendations and guaranteed effective minimums are not
interchangeable; confirm the capacitor choice against the regulator guidance.
[RP2350 datasheet, §6.3.8](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)

C34's nominal 22 µF does not establish a 22 µF effective output bank. TI's
selection tables specify effective capacitance and require stability/transient
checks when departing from the example combinations. The selected L2 and
approximately 2.2 MHz operating frequency need to be considered together.
[TPS62933 datasheet](https://www.ti.com/lit/ds/symlink/tps62933.pdf)

## Inductors

| Reference | Selected part | Verified ratings | Assessment |
|---|---|---|---|
| L1 | TDK MLP2016H3R3MT0S1 / C383379 | 3.3 µH ±20%; DCR 120 mΩ typical, 150 mΩ maximum; 1.2 A at a 40 °C temperature rise | DCR passes the RP2350 limit of 250 mΩ. **1.2 A is a thermal rating, not proof of saturation performance.** ≥1.5 A saturation and the required magnetic orientation remain unverified. |
| L2 | CJIANG FTC201612S1R5MBCA / C5832350 | 1.5 µH ±20%; DCR 40 mΩ typical, 50 mΩ maximum; Irms and Isat each 4 A typical, with 3.5 A in the limiting column | Use 3.5 A conservatively, not the catalog's headline 4 A. Isat corresponds to approximately 30% inductance loss. Normal-load and fault margin depend on actual peak current. |

TDK's [product page](https://product.tdk.com/en/search/inductor/inductor/smd/info?part_no=MLP2016H3R3MT0S1)
does not specify a saturation-current value. The
[RP2350 guide](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)
requires a fully shielded, polarity-marked inductor and recommends
AOTA-B201610S3R3-101-T. An inductor symbol's dot does not establish that the
ordered component has a usable orientation marking. Qualify L1 or replace it
with a documented suitable part; this review does not demonstrate that it fails
at the existing board's load.

The [CJIANG manufacturer specification](https://atta.szlcsc.com/upload/public/pdf/source/20240918/5AD8A1A0D9C2859F32750FF2CC514AFA.pdf)
labels the lower 3.5 A column “Max” despite listing a 4 A typical current; retain
that distinction when selecting margin. At 13.2 V input, 3.27 V output, 1.5 µH
and 2.2 MHz, ideal ripple is about 0.75 A peak-to-peak. A hypothetical 3 A load
therefore gives about 3.37 A peak before tolerances. This does not establish a
robust 3 A board rating, and TI's maximum high-side current limit is 5.8 A.
Measure expected worst-case load and assess startup/short-circuit behavior.

## Resistors

All fitted resistors are 0402, ±1%, **62.5 mW and 50 V maximum working voltage**.
The power rating also limits allowable continuous voltage to √(P × R); 50 V
does not imply that 50 V is safe across every resistance value.

| References | Resistance | Selected MPN / LCSC |
|---|---|---|
| R2, R4 | 5.1 kΩ | UNI-ROYAL 0402WGF5101TCE / C25905 |
| R12 | 30.9 kΩ | UNI-ROYAL 0402WGF3092TCE / C270615 |
| R13, R33 | 10 kΩ | UNI-ROYAL 0402WGF1002TCE / C25744 |
| R14 | 13.7 kΩ | Yageo RC0402FR-0713K7L / C274888 |
| R31, R37, R38 | 27 Ω | Yageo RC0402FR-0727RL / C138021 |
| R32, R40 | 1 kΩ | UNI-ROYAL 0402WGF1001TCE / C11702 |
| R34 | 51 Ω | Yageo RC0402FR-0751RL / C137960 |
| R35 | 750 kΩ | UNI-ROYAL 0402WGF7503TCE / C122545 |
| R41, R42, R43 | 4.7 kΩ | UNI-ROYAL 0402WGF4701TCE / C25900 |

At 3.63 V and −1% resistance, R40 dissipates at most 13.31 mW if the full
rail is across it, and each encoder pull-up dissipates 2.83 mW when closed.
R35 dissipates approximately 70 µW at the 13.2 V input check. These have ample
room-temperature power margin. Feedback/reset/CC resistor dissipation is also
well below 62.5 mW in normal operation.

R31's power rating permits approximately 48 mA RMS and R34's approximately
35 mA RMS before temperature derating. Their actual VREG_AVDD/VSL currents
were not established here. USB series resistors carry dynamic signal current;
applying the entire 3.3 V rail as a continuous DC drop is not the correct model.
Confirm these currents/waveforms in hardware if a guaranteed thermal margin
is required. The 51 Ω VSL resistor must not be assumed to carry the full OLED
panel supply current.

## Other selected parts and scope limits

| Part | Selected rating or operating specification | Assessment |
|---|---|---|
| U4 TPS62933DRLR | 3.8–30 V operating input; 3 A regulator IC | 13.2 V input passes. IC current rating does not establish the assembled board's continuous output capability. |
| U1 RP2354A | 3.3 V supply domains: relevant recommended range 3.135–3.63 V | Divider nominal output is 3.272 V. Combining ±1% resistors and ±2% feedback reference gives about 3.16–3.39 V before ripple/transients; verify dynamic margin. |
| D2/D3 1N4148WT / C917006 | Catalog: 75 V reverse, 150 mA forward, 150 mW | Voltage is ample. Actual VSL network current and thermal conditions remain to be verified. |
| J1 AFC07-S30ECA-00 / C262645 | Catalog: 50 V, 0.5 A/contact | Voltage passes. Verify worst-case OLED current per contact; parallel pins do not guarantee equal sharing. |
| J5 AFA01-S12FCA-00 / C262757 | Catalog: 50 V, 0.4 A/contact | Voltage passes. Check total input and cable/contact current at maximum brightness/load. |
| J6 TYPE-C-31-M-12 / C165948 | Catalog: 20 V, 5 A power-contact rating | Development data port; VBUS is not used to power the board. Existing documented USB limitations remain. |
| SW1/SW2 TS-1088-AR02016 / C720477 | Catalog: 12 V, 50 mA | 3.3 V reset/BOOT switching currents are well below these ratings. |
| SW3 SIQ-02FVS3 / C41430892 | Supplier catalog returned no electrical ratings | Pull-ups produce about 0.70 mA/contact nominal. This review does not certify contact life or a manufacturer electrical rating. |
| Y1 ABM8-272-T3 / C20625731 | 12 MHz, CL 10 pF, ESR 50 Ω, ±30 ppm initial tolerance and stability, −40…85 °C | Crystal startup, drive level and board stray capacitance remain hardware validation items. |

The connector, diode, switch and crystal entries above record catalog data;
they are not a fresh end-to-end datasheet qualification. Exact selected part
identities for all 28 unique LCSC groups were reconciled. Catalog snapshots
do not guarantee availability or a supplier's assembly-model orientation.

## Verification and release status

After the metadata changes, KiCad 10.0.6 checks produced zero ERC findings,
physical DRC violations, unconnected items and schematic/PCB parity findings
with the project's configured rules. All 253 exported pin-to-net assignments,
component values, footprints and LCSC codes match the pre-cleanup baseline;
all 72 PCB footprints retain their geometry and standard fields. The rendered
schematic's updated voltage labels were inspected.

Local evidence is under `review/bom-metadata/`, including catalog responses,
curve data, before/after exports, native IPC captures, and `verification.json`.
These generated review files are not part of the manufacturing package.

Keep [#13](https://github.com/bitaxeorg/bonanzaDisplay/issues/13) open for final
BOM/CPL regeneration and supplier assembly-preview checks. Resolve the
capacitance and inductor qualification items above before treating this
ratings review as complete. Existing manufacturing exports are superseded
by subsequent design changes.
