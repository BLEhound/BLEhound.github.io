# 构建固件

BLEhound 的固件是一个 Zephyr / nRF Connect SDK 应用。它直接驱动射频（参见 `firmware/src/radio_nrf54l.c` / `radio_nrf52.c`）。

- **目标平台：** nRF54LM20A（主要平台，搭配 nRF21540 FEM）及 nRF52840
- **SDK：** nRF Connect SDK **v3.4.0**（在 `west.yml` 中固定版本）

## 工具链（west）

```bash
mkdir blehound-ws && cd blehound-ws     # path must contain no spaces
git clone https://github.com/BLEhound/BLEhound
west init -l BLEhound                    # BLEhound/west.yml is the manifest
west update                              # pulls NCS from official Nordic GitHub
west zephyr-export
```

你还需要一个与 NCS v3.4.0 兼容的 Zephyr SDK（1.0.1）。若未被自动检测到，请设置 `ZEPHYR_SDK_INSTALL_DIR`。

## 构建

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

=== "封装脚本"

    ```bash
    BLEhound/tools/build.sh                           # nRF54LM20A dongle → build_dongle/
    BLEHOUND=0 BLEhound/tools/build.sh nrf52840dk/nrf52840 # nRF52840 → build/
    ```

## 烧录

```bash
west flash
# or via nrfutil + J-Link (handles APPROTECT recovery):
BLEhound/tools/flash.sh
RECOVER=1 BLEhound/tools/flash.sh    # unlock APPROTECT first
```

## 配置

| Kconfig | 含义 |
|---|---|
| `SNIFFER_DEFAULT_TARGET_MAC` | 启动时的目标 MAC（留空 = 不过滤）；一旦连接后由上位机覆盖 |
| `TRI_STRATEGY_*` | 收到 `CONNECT_IND` 之后的三板跟踪策略 |

在运行时，上位机（extcap）控制信道、目标 MAC、单／多目标模式以及多信道接力 —— 因此你很少需要重新构建来改变行为。
