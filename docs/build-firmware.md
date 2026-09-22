# Build the firmware

BLEhound's firmware is a Zephyr / nRF Connect SDK application. It drives the radio
directly (see `firmware/src/radio_nrf54l.c` / `radio_nrf52.c`).

- **Targets:** nRF54LM20A (primary, with nRF21540 FEM) and nRF52840
- **SDK:** nRF Connect SDK **v3.4.0** (pinned in `west.yml`)

## Toolchain (west)

```bash
mkdir blehound-ws && cd blehound-ws     # path must contain no spaces
git clone https://github.com/BLEhound/BLEhound
west init -l BLEhound                    # BLEhound/west.yml is the manifest
west update                              # pulls NCS from official Nordic GitHub
west zephyr-export
```

You also need a Zephyr SDK compatible with NCS v3.4.0 (1.0.1). If it is not
auto-detected, set `ZEPHYR_SDK_INSTALL_DIR`.

## Build

=== "nRF54LM20A dongle"

    ```bash
    west build -b nrf54lm20dk/nrf54lm20a/cpuapp -s BLEhound/firmware \
      -- -DEXTRA_CONF_FILE=boards/blehound.conf \
         -DEXTRA_DTC_OVERLAY_FILE=boards/blehound.overlay
    ```

=== "nRF52840 DK / Dongle"

    ```bash
    west build -b nrf52840dk/nrf52840 -s BLEhound/firmware
    ```

=== "Wrapper script"

    ```bash
    BLEhound/tools/build.sh                           # nRF54LM20A dongle → build_dongle/
    BLEHOUND=0 BLEhound/tools/build.sh nrf52840dk/nrf52840 # nRF52840 → build/
    ```

## Flash

```bash
west flash
# or via nrfutil + J-Link (handles APPROTECT recovery):
BLEhound/tools/flash.sh
RECOVER=1 BLEhound/tools/flash.sh    # unlock APPROTECT first
```

## Configuration

| Kconfig | Meaning |
|---|---|
| `SNIFFER_DEFAULT_TARGET_MAC` | Boot-time target MAC (empty = no filter); host overrides it once connected |
| `TRI_STRATEGY_*` | Three-board follow strategy after a `CONNECT_IND` |

At runtime, the host (extcap) controls channel, target MAC, single/multi-target mode,
and the multi-channel relay — so you rarely need to rebuild to change behavior.
