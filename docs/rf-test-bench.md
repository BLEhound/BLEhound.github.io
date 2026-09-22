# RF test bench (bonus use)

Because BLEhound is a full BLE endpoint with a controllable radio and a passive
capture path, the same hardware can double as a **low-cost, relative BLE RF test
bench** — useful for firmware RF regression, batch consistency, and production
go/no-go screening.

!!! warning "What this is — and isn't"
    This is a **relative / golden-referenced** measurement, good for R&D and production
    sorting. It is **not** a calibrated, traceable, certification-grade measurement.
    Absolute power (dBm), frequency accuracy (ppm), modulation quality, and adjacent
    channel power to spec need instruments (SDR, spectrum analyzer, or a real BLE
    tester). See the limits below.

## The idea: signaling, not DTM

A "signaling" tester behaves as the peer in a **real** BLE connection, so the device
under test (DUT) runs its normal production firmware — no Direct Test Mode required.

- **Transmit-side metrics** come from analyzing the DUT's real packets: relative power
  (from RSSI), packet timing, and error rate.
- **Receiver sensitivity** uses the BLE protocol's own acknowledgment: a companion
  central steps its transmit power down while BLEhound watches the air for the DUT's
  `NESN` acknowledgments. The fraction of un-acked downlink packets is the DUT's
  receive PER — no test firmware on the DUT needed.

## What you can measure

| Metric | With BLEhound alone | Notes |
|---|---|---|
| Relative TX power | ✅ | RSSI per channel; catches PA/antenna/matching faults vs a golden unit |
| PER / CRC error rate | ✅ | Link quality on real traffic |
| Receive PER → relative sensitivity | ✅ | Companion central + ACK observation |
| Absolute TX power (dBm) | ⚠️ | Only semi-absolute after a one-time golden calibration |
| Frequency offset / drift | ❌ | Needs an SDR (e.g. HackRF) + a disciplined reference |
| Modulation quality (Δf) | ❌ | Needs IQ — an SDR |
| Adjacent channel power (deep limits) | ❌ | Needs a calibrated spectrum analyzer |

## Building it out

For frequency offset, drift, and modulation you add an SDR with a GPS-disciplined
reference; for calibrated absolute power and ACP you add a spectrum analyzer. A
practical production bench combines: a companion nRF central (protocol + sensitivity),
BLEhound (air observation + ACK-based PER), an SDR + GPSDO (frequency/modulation), and
— for calibrated power/ACP — a spectrum analyzer, all inside an RF-shielded fixture
with a golden-unit transfer calibration.

!!! note
    A shielded enclosure and a fixed fixture are essential — ambient 2.4 GHz traffic
    otherwise swamps the measurement. Absolute radiated figures (TRP/TIS) for a sealed
    device still require an OTA calibration chamber.
