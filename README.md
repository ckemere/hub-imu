# hub_imu: USB IMU module for FE1.1S USB 2.0 hubs

A small 2-layer module that stacks onto a USB 2.0 hub board built around the
Terminus **FE1.1S** hub IC, such as
[these hubs on Amazon](https://www.amazon.com/dp/B0GTVPVPV9). It plugs into one of the
hub's downstream ports through a 0.1" 1x4 header and appears on the host as a
USB device that bridges to a Bosch BNO055 9-axis IMU over I2C.

- **Board:** 21.0 x 14.4 mm, 2 layers, 1.6 mm FR4, 1 oz copper (JLCPCB standard rules)
- **Tools:** KiCad 9.0

## Contents

| File | What it is |
|---|---|
| `hub_imu.kicad_pro` / `.kicad_sch` / `.kicad_pcb` | KiCad project, schematic, layout |
| `hub_imu.kicad_dru` | Custom design rules (see below) |
| `stage1.svg` ... `stage5.svg` | Layout renders (F.Cu, B.Cu, F.SilkS, Edge.Cuts) after placement, power, USB, signals, pours |
| `drc_stage1.rpt` ... `drc_stage5.rpt` | DRC report after each stage (`drc_stage5.rpt` is the final one) |

## Circuit

| Ref | Part | Role |
|---|---|---|
| U1 | STM32F042F6P6 (TSSOP-20) | Crystal-less USB FS device, USB-to-I2C bridge |
| U2 | BNO055 (LGA-28) | 9-axis IMU, I2C mode (PS0 = PS1 = GND), address 0x28 (COM3 = GND) |
| U3 | AP2112K-3.3 (SOT-23-5) | 5 V to 3.3 V LDO, the only heat source on the board |
| J1 | 1x4 2.54 mm header | Connection to the hub |
| J2 | 2x3 1.27 mm header | SWD programming/debug |
| D1 + R5 | Red 0402 LED + 1k | Status LED on PA3 |
| R3, R4 | 4.7k | I2C pull-ups |
| R2 | 10k | BNO055 nRESET pull-up |
| R1 / C3 | 10k / 100 nF | BOOT0 pull-down / NRST filter |
| C1 / C2 | 1 uF / 4.7 uF | LDO input / output |
| C4-C8 | 100 nF | Decoupling (U1 VDD, U1 VDDA, U2 VDD, U2 VDDIO, U2 CAP) |

### J1 - hub connector (0.1" pitch)

| Pin | Signal |
|---|---|
| 1 | +5V (VBUS) |
| 2 | USB D- |
| 3 | USB D+ |
| 4 | GND |

**Check this pin order against the hub's downstream-port header before ordering.**
It has not been verified against the specific hub listed above.

### J2 - SWD (2x3, 1.27 mm)

| Pin | Signal | Pin | Signal |
|---|---|---|---|
| 1 | +3V3 | 2 | BOOT0 |
| 3 | SWDIO | 4 | SWCLK |
| 5 | NRST | 6 | GND |

This is a custom pinout, not a standard ARM SWD connector, so an adapter or cable is needed.

## Firmware notes (STM32F042F6P6)

| Function | Pin |
|---|---|
| USB D- / D+ | PA11 / PA12 on pins 17/18. **Set `SYSCFG_CFGR1.PA11_PA12_RMP`**, or USB will not enumerate (these pins default to PA9/PA10). |
| I2C1 SDA / SCL | PF0 / PF1 (AF1) |
| BNO055 INT | PA5 (pin 11), EXTI |
| BNO055 nRESET | PA6 (pin 12), drive low to reset (10k pull-up on board) |
| Status LED | PA3 (pin 9), active high, ~1.3 mA |
| SWD | PA13 / PA14 |

Use the internal HSI48 clock with CRS synced to USB SOF. There is no crystal, and the BNO055's 32 kHz crystal pins are left open (internal clock).

## Layout notes

- All components are on the top side.
- The bottom layer is a GND plane that also carries +3V3 distribution. Signals are on top, apart from four short jumpers: SDA under SCL, and BOOT0 / SWDIO to J2's through-hole pins.
- The USB D+/D- pair is 0.25 mm wide with a 0.2 mm gap, has no vias, and is length-matched (0 mm skew). Solid ground runs underneath it (bottom-layer track/via keepout `USB_BCu_keepout`).
- There are no bottom-layer tracks or vias under the IMU (`U2_BCu_keepout`), and no top copper inside U2's pad ring (`U2_FCu_interior`).
- Every decoupling cap is within 2 mm of the pin it serves and has an adjacent GND via.

### Custom design rules (`hub_imu.kicad_dru`)

- 0.2 mm minimum track and clearance; vias 0.6 mm pad / 0.3 mm drill.
- +5V / +3V3 tracks are 0.4 mm, necking to 0.25 mm only inside U2's courtyard (its pads are 0.254 mm wide).
- USB tracks are exactly 0.25 mm.
- No bottom-layer tracks or vias intersecting U2's courtyard.
- R1 and U1 GND pads may have one thermal spoke; both also have a direct track to a GND via.

## Verification status

The final DRC (`drc_stage5.rpt`) shows **0 errors and 0 unconnected pads**. The remaining items are
`lib_footprint_mismatch` warnings on the 0402 and LGA footprints. They come from KiCad re-save
rounding on rotated footprints, not from real geometry changes.

ERC reports only `lib_symbol_issues`, because the `hubimu` symbols are embedded in the schematic
rather than installed as a library.

## Open items

- Confirm J1's pin order against the hub's header (see above).
- Confirm the BNO055's PIN1 and reserved pins (12, 13, 15, 16, 21-24) against Bosch's pin table. They are currently unconnected.
- MCU decoupling is 100 nF on VDD and VDDA, plus 4.7 uF at the LDO output. ST's usual guidance also suggests 1 uF + 10 nF on VDDA.
