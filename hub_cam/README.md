# hub_cam: USB hub carrier for two cameras plus the hub_imu module

**Status:** schematic drafted (see `schematic.png`); PCB layout not started.

| Ref | Part | JLCPCB # | Role |
|---|---|---|---|
| U1 | FE1.1S-BQFN24B (WQFN-24, 4x4 mm) | C6776948 (extended) | USB 2.0 high-speed 4-port hub |
| Y1 | 12 MHz crystal, 3225 package | C9002 | Hub clock. The FE1.1S has internal load capacitors, so none are fitted. |
| R1 | 2.7k 1%, 0603 | C13167 | REXT bias resistor |
| R2, R3 | 10k, 0402 | C25744 | XRSTJ pull-up, DRV pull-down (normal hub mode, charging off) |
| C1, C2 | 10 uF 0603, 100 nF 0402 | C96446, C1525 | 5 V input to the hub |
| C3, C4 | 4.7 uF, 100 nF 0402 | C23733, C1525 | Output of the hub's internal 3.3 V regulator (VD33F) |
| C5 | 10 uF 0603 | C96446 | 5 V bulk capacitor at the camera connectors |
| J2, J3 | JST GH 4-pin, side entry (SM04B-GHS-TB) | C189895 | Camera ports. Pinout: 1 VBUS, 2 D-, 3 D+, 4 GND |
| J1 | 1x4 2.54 mm holes | - | Stacking header for hub_imu. Same footprint and pin order as hub_imu's J1. |
| J4-J7 | Wire solder holes | - | Host cable: 1.7 mm holes for 16-18 AWG power, 0.65 mm holes for D-/D+ |

## Port map

| Hub port | Connected to |
|---|---|
| 1 | CAM1 (J2) |
| 2 | CAM2 (J3) |
| 3 | IMU (J1) |
| 4 | Unused |

## Hub configuration

- **Self-powered (BUSJ tied high).** The host then doesn't limit the cameras to 100 mA each. Nothing on the board limits or switches the camera power.
- **VBUSM and OVCJ tied to 3.3 V.** There is no over-current sensing.
- **No ESD protection** (open question).

## Planned layout

- Board 1.0 mm thick, sized for 90 ohm differential pairs.
- About 16.5 x 23 mm, to fit within the 3/4" strap.
- The GH connectors, hub IC and crystal go on the bottom.
- The host-cable holes and cable-tie slots go at the far end, under the stacked IMU module.
