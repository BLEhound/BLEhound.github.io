# Quick start

From an assembled BLEhound dongle (or an nRF52840 DK/Dongle) to live packets in
Wireshark.

## Prerequisites

- A BLEhound dongle (nRF54LM20A) — or an nRF52840 DK / Dongle to try the firmware
- [nRF Connect SDK v3.4.0](https://www.nordicsemi.com/Products/Development-software/nrf-connect-sdk) toolchain (`west`)
- [Wireshark](https://www.wireshark.org/download.html) (official build)
- Python 3.9+

## 1. Build & flash the firmware

```bash
mkdir blehound-ws && cd blehound-ws
git clone https://github.com/BLEhound/BLEhound
west init -l BLEhound && west update && west zephyr-export

west build -b nrf54lm20dk/nrf54lm20a/cpuapp -s BLEhound/firmware \
  -- -DEXTRA_CONF_FILE=boards/blehound.conf \
     -DEXTRA_DTC_OVERLAY_FILE=boards/blehound.overlay
west flash
```

For an nRF52840 board: `west build -b nrf52840dk/nrf52840 -s BLEhound/firmware && west flash`.

More detail: [Build the firmware](build-firmware.md).

## 2. Install the Wireshark plugin

```bash
pip install -r BLEhound/host/requirements.txt
BLEhound/tools/install_extcap.sh
```

## 3. Capture

1. Restart Wireshark.
2. In the interface list, pick **nRF BLE Sniffer**.
3. (Optional) In the interface options, set a scan channel or a target MAC.
4. Start capturing — you should see advertising packets; once a connection starts,
   the sniffer follows it.

Load the display-filter bookmarks from `host/wireshark_dfilters` to quickly isolate
a device, connection data, or CRC errors.

Next: capture all three advertising channels at once with
[three boards](usage-multichannel.md).
