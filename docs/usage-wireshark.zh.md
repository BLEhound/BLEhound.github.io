# 配合 Wireshark 使用

BLEhound 通过一个 [extcap](https://www.wireshark.org/docs/man-pages/extcap.html) 插件与 Wireshark 通信。Wireshark 调用该插件来列出接口和选项，然后从中以 PCAP（`LINKTYPE_BLUETOOTH_LE_LL_WITH_PHDR`）形式串流数据包。

## 安装

```bash
pip install -r BLEhound/host/requirements.txt
BLEhound/tools/install_extcap.sh
```

或手动安装：

```bash
mkdir -p ~/.local/lib/wireshark/extcap
cp BLEhound/host/nrf_sniffer_extcap.py BLEhound/host/tri_aggregator.py \
   ~/.local/lib/wireshark/extcap/
chmod +x ~/.local/lib/wireshark/extcap/nrf_sniffer_extcap.py
```

重启 Wireshark。会出现两个接口：

- **nRF BLE Sniffer** —— 单块板
- **nRF BLE Sniffer (3ch aggregated)** —— 三块板合并为一路数据流

固件的 USB 标识：VID `0x1915`，PID `0x520F`，产品名 `nRF BLE Sniffer`。

### Windows

Windows 上 Wireshark 只跑 `.exe` / `.bat` 形式的 extcap,不直接跑 `.py`,所以用随附的 `.bat` 包装器:

1. 装 Python 3(勾选 **Add python.exe to PATH**),执行 `pip install pyserial`。
2. 找 extcap 目录 —— Wireshark **帮助 → 关于 → 文件夹 → Personal Extcap path**(通常 `%APPDATA%\Wireshark\extcap`)。
3. 把 `BLEhound/host/` 里的 `nrf_sniffer_extcap.py`、`nrf_sniffer_extcap.bat`、`tri_aggregator.py` 复制进去,重启 Wireshark。

串口在 Windows 上显示为 `COMx`,其余用法相同。`tools/*.sh` 脚本仅限 macOS/Linux。

## 选项

在单板接口上，你可以设置：

- **Scan channel（扫描信道）** —— 守护哪个广播信道（37 / 38 / 39）
- **Target MAC（目标 MAC）** —— 仅跟踪到此设备的连接（格式 `aa:bb:cc:dd:ee:ff`）
- **Include CRC errors（包含 CRC 错误）** —— 放行畸形数据包以便调试
- **Multi-target mode（多目标模式）** —— 同时跟踪多个并发连接（评估阶段）

## 显示过滤器书签

`host/wireshark_dfilters` 包含现成的显示过滤器。常用的有：

| 过滤器 | 显示内容 |
|---|---|
| `btle.access_address != 0x8e89bed6` | 仅连接数据（所有连接） |
| `btle.advertising_header.pdu_type == 0x05 && btle_rf.flags.crc_valid == 1` | 有效的 `CONNECT_IND`（真正的连接起点） |
| `btatt` | ATT/GATT 应用层数据 |
| `btle_rf.flags.crc_valid == 0` | CRC 错误的数据包（链路质量） |

!!! tip "提示"
    如果连接数据看起来「消失」了，请**先检查显示过滤器栏** —— 通常是残留的书签过滤器（例如某个写死的接入地址）所致，而不是数据包丢失。

## 解密

对于加密链路，请向 Wireshark 提供 LTK（Preferences → Protocols → Bluetooth Low Energy），或用[安全工具](https://github.com/BLEhound/BLEhound/tree/main/host/security)恢复传统配对（Legacy pairing）密钥。LE Secure Connections 无法被被动解密。
