# BLEhound

**An open, FEM-boosted Bluetooth LE sniffer on nRF54 — with synchronized
multi-channel capture.**

BLEhound is a from-scratch BLE sniffer: firmware, a Wireshark `extcap` plugin, and
open hardware. The radio is driven by our own firmware (not Nordic's sniffer
firmware), which is what enables connection following, encrypted-link capture, and
**three-radio synchronized multi-channel** capture.

Source code: [github.com/BLEhound/BLEhound](https://github.com/BLEhound/BLEhound)

## Why BLEhound

- **Synchronized multi-channel capture (three boards).** One radio can only listen on
  one channel at a time, so a lone sniffer can miss a `CONNECT_IND` on another
  advertising channel. BLEhound runs three boards guarding 37 / 38 / 39, time-aligned
  over a hardware SYNC line + inter-board SPI, and merged by the host into **one**
  Wireshark interface. Validated end-to-end on real boards.
- **FEM (nRF21540 PA/LNA)** for better sensitivity and range.
- **New silicon, current features** — nRF54LM20A (also nRF52840): CSA #1/#2 connection
  following, 1M / 2M / Coded PHY with in-connection PHY updates, extended advertising
  (`AUX_CONNECT_REQ`), BLE 5.x/6.x link-layer coverage.
- **Complete and reproducible** — JLCEDA Pro source, Gerbers, BOM, and a 3D-printed case.
- **Doubles as a low-cost BLE RF test bench** — see [RF test bench](rf-test-bench.md).

## Honest scope

- As a single board it sits alongside [Sniffle](https://github.com/nccgroup/Sniffle)
  and the Nordic nRF Sniffer; its edge there is the FEM, newer silicon, and integrated
  hardware.
- Multi-board redundancy helps most on lossy/marginal links; a clean link is already
  fully captured by one board.
- The three-board relay does not apply to extended-advertising connections and cannot
  recover encrypted control PDUs.

## Get started

<div class="grid cards" markdown>

- :material-rocket-launch: **[Quick start](quickstart.md)** — from zero to packets in Wireshark
- :material-chip: **[Build the firmware](build-firmware.md)** — west / nRF Connect SDK
- :material-shark: **[Use with Wireshark](usage-wireshark.md)** — the extcap plugin
- :material-access-point-network: **[Multi-channel](usage-multichannel.md)** — three-board setup
- :material-developer-board: **[Hardware](hardware.md)** — order & assemble the dongle
- :material-cog: **[How it works](architecture.md)** — the design

</div>

## License & ethics

Code: Apache-2.0. Hardware: CERN-OHL-S-2.0. BLEhound includes legacy-pairing analysis
tooling; use it only on devices you own or are authorized to test. LE Secure
Connections (BLE 4.2+) cannot be passively broken by any sniffer.
