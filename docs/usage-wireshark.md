# Use with Wireshark

BLEhound talks to Wireshark through an [extcap](https://www.wireshark.org/docs/man-pages/extcap.html)
plugin. Wireshark calls the plugin to list interfaces and options, then streams
packets from it as PCAP (`LINKTYPE_BLUETOOTH_LE_LL_WITH_PHDR`).

## Install

```bash
pip install -r BLEhound/host/requirements.txt
BLEhound/tools/install_extcap.sh
```

Or manually:

```bash
mkdir -p ~/.local/lib/wireshark/extcap
cp BLEhound/host/nrf_sniffer_extcap.py BLEhound/host/tri_aggregator.py \
   ~/.local/lib/wireshark/extcap/
chmod +x ~/.local/lib/wireshark/extcap/nrf_sniffer_extcap.py
```

Restart Wireshark. Two interfaces appear:

- **nRF BLE Sniffer** — a single board
- **nRF BLE Sniffer (3ch aggregated)** — three boards merged into one stream

The firmware's USB identity: VID `0x1915`, PID `0x520F`, product `nRF BLE Sniffer`.

## Options

On the single-board interface you can set:

- **Scan channel** — which advertising channel to guard (37 / 38 / 39)
- **Target MAC** — only follow connections to this device (format `aa:bb:cc:dd:ee:ff`)
- **Include CRC errors** — pass malformed packets through for debugging
- **Multi-target mode** — track several concurrent connections (evaluation)

## Display-filter bookmarks

`host/wireshark_dfilters` contains ready-made display filters. Useful ones:

| Filter | Shows |
|---|---|
| `btle.access_address != 0x8e89bed6` | Connection data only (all connections) |
| `btle.advertising_header.pdu_type == 0x05 && btle_rf.flags.crc_valid == 1` | Valid `CONNECT_IND` (real connection starts) |
| `btatt` | ATT/GATT application data |
| `btle_rf.flags.crc_valid == 0` | CRC-error packets (link quality) |

!!! tip
    If connection data seems to "disappear", check the **display filter bar first** —
    a leftover bookmark filter (e.g. a hard-coded access address) is the usual cause,
    not lost packets.

## Decryption

For encrypted links, provide the LTK to Wireshark (Preferences → Protocols → Bluetooth
Low Energy), or recover a Legacy-pairing key with the [security tooling](https://github.com/BLEhound/BLEhound/tree/main/host/security).
LE Secure Connections cannot be passively decrypted.
