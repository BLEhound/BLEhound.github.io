# Multi-channel capture (three boards)

A single BLE radio can only listen on one channel at a time. So a lone sniffer, sitting
on one advertising channel, can miss a `CONNECT_IND` that arrives on one of the other
two — and then never sees that connection.

BLEhound solves this with **three boards**, each permanently guarding one advertising
channel (37 / 38 / 39), time-aligned and merged by the host into a single Wireshark
stream.

## How it fits together

```
 board 0 (ch 37) ─┐
 board 1 (ch 38) ─┤─ USB ─►  host aggregator ─► one Wireshark interface
 board 2 (ch 39) ─┘             (time-align + de-duplicate)
        │
        └── SYNC line + inter-board SPI  (shared time base)
```

- Each board stamps packets with its `board_id` and the latest SYNC-edge tick.
- The host `SyncClock` aligns the three timelines; the `Aggregator` merges and
  de-duplicates by `(access address, CRC, PDU)`.
- Optionally, when one board catches a `CONNECT_IND`, the host relays the connection
  parameters to the other two so all three follow the same connection in parallel
  (`FollowRelay`).

## Wiring

Connect the boards' SYNC line together and the inter-board SPI per the hardware
notes. Flash the same firmware to all three; each board's role (which channel it
guards) is assigned by strap/role config.

## Capture

1. Install the extcap (see [Use with Wireshark](usage-wireshark.md)).
2. In Wireshark, choose the **nRF BLE Sniffer (3ch aggregated)** interface.
3. (Optional) enable the three-board follow relay, and set a target MAC.
4. Start capturing — you get one merged, de-duplicated, time-consistent stream.

From the command line (e.g. for scripted runs):

```bash
python3 host/tri_aggregator.py --selftest      # verify the merge logic
```

## What it does and doesn't buy you

- **Advertising coverage:** with three boards you never miss a `CONNECT_IND`
  regardless of which channel it lands on.
- **Loss resilience:** the boards have independent antenna positions/noise, so if one
  drops packets another may still have them — the merged stream stays continuous. This
  helps most on lossy/marginal links (measured: higher survival, zero early
  disconnects vs a single board).
- **Limits:** on a clean link a single board already captures everything; the relay
  does **not** apply to connections established via extended advertising
  (`AUX_CONNECT_REQ`); and it cannot recover encrypted control PDUs (all boards miss
  the same `instant` together).
