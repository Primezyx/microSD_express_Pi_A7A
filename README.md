# microSD-express Pi Adapter

![demo](demo.jpg)

- This adapter allows you to connect a MicroSD Express card directly to the PCIe connector on a Raspberry Pi 5
- Very fast speeds (800MB/s sequential read) can be achieved this way
- NOTE: controlled impedance is critical for good PCIe performance. The files are designed for 50μm PI dielectric and 0.5oz copper.

---

## Fork: Radxa Cubie A7A compatibility

This fork adds a pin-by-pin comparison between this adapter's J2 and the
Radxa Cubie A7A PCIe FPC connector (J3 on schematic page 16,
`16_PCIE_40PIN_CONNECT`, `radxa_cubie_a7a_v1.10_schematic.pdf`).

**Conclusion: no board modification is required.** The 16-pin connector
matches and every signal lines up pin for pin.

Full comparison (Chinese): [A7A_适配对照表.md](A7A_适配对照表.md)

### What was checked

| Item | Result |
|---|---|
| Pin count | 16 vs 16 — match |
| Pitch | 0.5 mm vs 0.5 mm — match |
| Pin 1–2 | 5V — match |
| Pin 3, 6, 9, 12 | GND — match |
| Pin 4 / 5 | CLK_P / CLK_N — match, no polarity swap |
| Pin 7 / 8 | RX_P / RX_N — match, no polarity swap |
| Pin 10 / 11 | TX_P / TX_N — match, no polarity swap |
| Pin 13 | POWER_EN — match |
| Pin 14 | WAKE — match |
| Pin 15 | CLKREQ — match |
| Pin 16 | RESET / PERST — match |

The A7A symbol `FPC_2X8P` is drawn as 2×8, but a dual-row 16-pin connector
uses the same pin numbering as the single-row 16-contact PCIe FPC standard,
so signals transfer directly.

### Power

A7A feeds 5V1 / 5V2 through FB4 (60R @ 4A ferrite bead) from VCC5V0_SYS,
decoupled by C87 22uF + C88 100nF + C89 22uF. That is more headroom than the
two 500 mA-rated 5V pins on the Pi 5 and is ample for the ~1.8 W peak of a
microSD Express card.

### Impedance and stackup — do not change

- Differential trace width: **0.12192 mm**
- Reference plane: B.Cu ground
- Dielectric: **50 μm polyimide (PI)**
- Copper: **0.5 oz**

These values target 90 Ω differential. Changing the stackup or the fab house
means recomputing the impedance, otherwise the PCIe 3.0 eye diagram collapses.

### Still unverified on A7A

1. **Whether the A733 PCIe controller enumerates microSD Express cards.**
   Radxa's own docs state NVMe SSD support is still being adapted, so the
   A733 PCIe peripheral ecosystem is immature. The protocol is standard PCIe,
   but there is no tested precedent for this card type.
2. **Mechanical retention.** This adapter has only 16 SMD pads and no
   SH1 / SH2 ground mounting tabs like J3. Cable retention depends on the
   FPC itself.

### Suggested bring-up order

1. Check the A7A device tree for `PCIE_PERSTN` and `PCIE_CLKREQ`
2. Validate the A733 PCIe lane with a plain M.2 NVMe adapter
3. Insert this adapter and check `lspci`
4. Only then benchmark

---

## Credits

Original project: <https://github.com/vapetrov/microSD_express_Pi>
by Vassily Petrov.

This fork adds documentation only — the schematic, PCB, footprints, gerbers
and 3D models are unchanged from upstream.

## License

MIT, see [LICENSE](LICENSE). Original copyright (c) 2025 Vassily Petrov
is retained.

## Data sources

- Upstream project: <https://github.com/vapetrov/microSD_express_Pi>
- Radxa Cubie A7A schematic: <https://dl.radxa.com/cubie/a7a/docs/hw/radxa_cubie_a7a_v1.10_schematic.pdf>
- Raspberry Pi PCIe connector standard: <https://datasheets.raspberrypi.com/pcie/pcie-connector-standard.pdf>
