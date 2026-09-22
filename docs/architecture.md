# How it works

BLEhound is deliberately built on a **direct radio driver** rather than Nordic's
sniffer firmware. Owning the radio configuration is what makes the deeper features
possible.

## Pipeline

```
 nRF radio ─► firmware (de-whiten, CRC, timestamp) ─► COBS frames over USB
     ─► host extcap (de-frame, build PCAP) ─► Wireshark dissectors
```

Each captured packet carries channel, RSSI, a microsecond hardware timestamp, CRC
result, PHY, and access address. The host wraps it as
`LINKTYPE_BLUETOOTH_LE_LL_WITH_PHDR` so Wireshark's own Bluetooth dissectors decode
the full stack (advertising PDUs, LL control, L2CAP, ATT/GATT, SMP).

## Connection following

After a valid `CONNECT_IND`, the firmware computes the hop sequence (CSA #1 or #2) and
follows the connection:

- Re-anchors on actually-received packets to resist clock drift (anchor error stays
  within ~1 µs over long runs).
- Applies channel-map and connection-parameter updates at the correct `instant`,
  including `WinOffset` re-anchoring.
- Follows in-connection PHY updates (1M / 2M / Coded), including asymmetric per-event
  PHY.
- Follows connections established via extended advertising (`AUX_CONNECT_REQ`).
- Keeps following encrypted links (encryption does not change the hop sequence);
  ciphertext is captured and can be decrypted in Wireshark with the key.

## Three-board synchronization

A single radio hears one channel at a time. Three boards, each guarding 37 / 38 / 39,
give simultaneous advertising-channel coverage. Two mechanisms keep them coherent:

- **Shared time base.** One board periodically drives a hardware **SYNC** edge; every
  board captures that same physical edge on its own timer and reports the tick with
  each packet. The host `SyncClock` computes per-board offsets and re-bases every
  timestamp onto a reference board.
- **Merge & de-duplicate.** The `Aggregator` orders the three streams on the aligned
  time base and removes duplicates by `(access address, CRC, PDU)` within a window
  smaller than the inter-frame spacing, so genuine master/slave packets in the same
  event are kept while cross-board copies of one packet are collapsed.

Optionally, `FollowRelay` takes a `CONNECT_IND` seen by one board, converts its anchor
to each other board's time base, and injects a follow command so all three track the
same connection in parallel.

## Front-end (FEM)

The hardware uses an nRF21540 PA/LNA. The LNA lifts weak signals above the noise floor
for better sensitivity/range; `src/fem_ctrl.c` handles the FEM control lines.

## Host modules

- `nrf_sniffer_extcap.py` — extcap protocol, serial I/O, COBS de-framing, PCAP output.
- `tri_aggregator.py` — pure logic (no I/O): `SyncClock`, `Aggregator`, `FollowRelay`.
  Runnable self-test: `python3 tri_aggregator.py --selftest`.

## Scope & limits

- Multi-board redundancy helps on lossy/marginal links; a clean link is already fully
  captured by one board.
- The follow relay does not apply to extended-advertising connections.
- Encrypted control PDUs cannot be recovered by adding boards — all boards miss the
  same `instant` together.
- LE Secure Connections (BLE 4.2+) cannot be passively decrypted by any sniffer.
