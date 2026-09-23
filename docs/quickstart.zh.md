# 快速上手

从一个组装好的 BLEhound dongle（或一块 nRF52840 DK/Dongle）到在 Wireshark 中看到实时数据包。

## 前置条件

- 一个 BLEhound dongle（nRF54LM20A）—— 或一块 nRF52840 DK / Dongle 用于试用固件
- [nRF Connect SDK v3.4.0](https://www.nordicsemi.com/Products/Development-software/nrf-connect-sdk) 工具链（`west`）
- [Wireshark](https://www.wireshark.org/download.html)（官方构建版本）
- Python 3.9+

## 1. 构建并烧录固件

```bash
mkdir blehound-ws && cd blehound-ws
git clone https://github.com/BLEhound/BLEhound
west init -l BLEhound && west update && west zephyr-export

west build -b nrf54lm20dk/nrf54lm20a/cpuapp -s BLEhound/firmware \
  -- -DEXTRA_CONF_FILE=boards/blehound.conf \
     -DEXTRA_DTC_OVERLAY_FILE=boards/blehound.overlay
west flash
```

对于 nRF52840 板：`west build -b nrf52840dk/nrf52840 -s BLEhound/firmware && west flash`。

更多细节：[构建固件](build-firmware.md)。

## 2. 安装 Wireshark 插件

```bash
pip install -r BLEhound/host/requirements.txt
BLEhound/tools/install_extcap.sh
```

!!! tip "Windows"
    Windows 上 Wireshark 需要随附的 `.bat` 包装器 —— 把 `nrf_sniffer_extcap.py`、
    `nrf_sniffer_extcap.bat`、`tri_aggregator.py` 复制进 Wireshark 的 extcap 目录
    (`%APPDATA%\Wireshark\extcap`)。详见 [配合 Wireshark → Windows](usage-wireshark.md#windows)。

## 3. 抓包

1. 重启 Wireshark。
2. 在接口列表中，选择 **nRF BLE Sniffer**。
3. （可选）在接口选项中，设置扫描信道或目标 MAC。
4. 开始抓包 —— 你应当能看到广播包；一旦连接建立，嗅探器便会跟踪它。

从 `host/wireshark_dfilters` 加载显示过滤器书签，可快速隔离某个设备、连接数据或 CRC 错误。

下一步：用 [三块板](usage-multichannel.md) 同时抓取全部三个广播信道。
