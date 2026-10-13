# ZNano on ESP32-C3 — ESP-IDF / QEMU benchmark

**Run date:** 2026-10-11 · **Environment:** ESP-IDF 5.5.1, Espressif QEMU 9.0.0

**270/270 exact round trips** over six synthetic workloads, five batch sizes, three formats (raw, ZNano, zlib), and three repetitions. A separate automated test confirmed identical compressed sizes on host and emulated target.

> **Scope:** Synthetic test fixtures, not customer-data averages. QEMU does not establish performance on physical silicon, Wi-Fi/BLE bandwidth savings, or power/battery improvement. Regular structured data benefits most; irregular data and especially single-record batches can expand.

## On-wire payload sizes

Each sample is 20 binary bytes. Every size includes a shared 16-byte envelope and codec-specific bytes. Reduction (%) = `100 × (1 − encoded / raw)`; a negative value indicates expansion.

**zlib baseline:** 1.3.1, level 6, 4 KiB window (`wbits=12`), `memLevel=5`, zlib wrapper with Adler-32. These embedded settings differ from conventional 15/8 settings.

| Synthetic workload | Records | Raw (B) | ZNano (B) | ZNano reduction | zlib (B) | zlib reduction |
|---|---:|---:|---:|---:|---:|---:|
| Regular meter | 1 | 36 | 43 | -19.44% | 43 | -19.44% |
| Regular meter | 4 | 96 | 55 | 42.71% | 79 | 17.71% |
| Regular meter | 16 | 336 | 126 | 62.50% | 211 | 37.20% |
| Regular meter | 64 | 1296 | 516 | 60.19% | 630 | 51.39% |
| Regular meter | 256 | 5136 | 1957 | 61.90% | 2174 | 57.67% |
| Irregular meter | 1 | 36 | 43 | -19.44% | 44 | -22.22% |
| Irregular meter | 4 | 96 | 87 | 9.38% | 107 | -11.46% |
| Irregular meter | 16 | 336 | 261 | 22.32% | 347 | -3.27% |
| Irregular meter | 64 | 1296 | 1052 | 18.83% | 1249 | 3.63% |
| Irregular meter | 256 | 5136 | 4124 | 19.70% | 4892 | 4.75% |
| Regular industrial | 1 | 36 | 43 | -19.44% | 44 | -22.22% |
| Regular industrial | 4 | 96 | 52 | 45.83% | 74 | 22.92% |
| Regular industrial | 16 | 336 | 96 | 71.43% | 172 | 48.81% |
| Regular industrial | 64 | 1296 | 327 | 74.77% | 478 | 63.12% |
| Regular industrial | 256 | 5136 | 1192 | 76.79% | 1560 | 69.63% |
| Irregular industrial | 1 | 36 | 43 | -19.44% | 45 | -25.00% |
| Irregular industrial | 4 | 96 | 99 | -3.12% | 107 | -11.46% |
| Irregular industrial | 16 | 336 | 321 | 4.46% | 347 | -3.27% |
| Irregular industrial | 64 | 1296 | 1209 | 6.71% | 1284 | 0.93% |
| Irregular industrial | 256 | 5136 | 4762 | 7.28% | 4965 | 3.33% |
| Regular GNSS | 1 | 36 | 43 | -19.44% | 45 | -25.00% |
| Regular GNSS | 4 | 96 | 55 | 42.71% | 86 | 10.42% |
| Regular GNSS | 16 | 336 | 126 | 62.50% | 215 | 36.01% |
| Regular GNSS | 64 | 1296 | 485 | 62.58% | 662 | 48.92% |
| Regular GNSS | 256 | 5136 | 1957 | 61.90% | 2267 | 55.86% |
| Irregular GNSS | 1 | 36 | 43 | -19.44% | 45 | -25.00% |
| Irregular GNSS | 4 | 96 | 96 | 0.00% | 107 | -11.46% |
| Irregular GNSS | 16 | 336 | 306 | 8.93% | 347 | -3.27% |
| Irregular GNSS | 64 | 1296 | 1209 | 6.71% | 1299 | -0.23% |
| Irregular GNSS | 256 | 5136 | 4762 | 7.28% | 5081 | 1.07% |

## Memory measured under FreeRTOS

Maximum observed over **90 executions per format**. Each test used a fresh task. The round-trip stack column includes the harness and the encoding phase; it is **not decoder-only stack usage**.

| Format | Peak encode allocations (B) | Peak decode allocations (B) | Stack after encode (B) | Stack after round trip (B) |
|---|---:|---:|---:|---:|
| raw | 0 | 0 | 304 | 304 |
| ZNano | 0 | 0 | 432 | 528 |
| zlib (6/w12/m5) | 38,596 | 7,120 | 576 | 576 |

Allocation values refer to codec-requested bytes, excluding allocator metadata and integration buffers. **ZNano requests no dynamic allocation in this test.** The zlib decoder receives a full output buffer in one call, possibly avoiding a streaming window allocation.

The full harness additionally requires **30,928 B of static buffers**: 20,608 B for ZNano hexadecimal adaptation, 5,200 B for shared transport, and 5,120 B for zlib reconstruction. The task reserves 16,384 B of stack; 30,720 B of test fixtures reside in flash. Control variables and FreeRTOS have additional footprints. Memory-fill stack measurements reflect touched bytes and are **not an absolute safe product stack bound**.

## Compiled footprint

ESP-IDF 5.5.1, RISC-V GCC 14.2.0 (`esp-14.2.0_20241119`), optimized `-Os`. Application sizes include FreeRTOS, fixtures, harness, and enabled codecs, but exclude bootloader and partition table.

| Build variant | Application image (B) | Increase versus raw (B) |
|---|---:|---:|
| raw | 166,048 | 0 |
| ZNano | 170,624 | 4,576 |
| zlib | 217,488 | 51,440 |
| all | 221,984 | 55,936 |

These size differences include variant-specific harness code and linker-retained dependencies. In the combined firmware, the attributed linked contributions are:

| Component | Flash contribution (B) | Static RAM data (B) |
|---|---:|---:|
| `libznano.a` | 3,757 | 45 |
| `libzlib.a` | 31,169 | 0 |

The ZNano value covers encoder and decoder, excluding ASCII adapters. Linked contributions do not represent complete standalone library sizes or runtime heap/stack usage.

## Evidence and reproducibility

The original benchmark references an evidence directory named `results/esp32c3-qemu-2026-10-11/` containing `measurements.csv` (270 runs), `summary.json`, `qemu.log`, `firmware.json` and `provenance.json`. It also references a separate reproduction `README.md` and `data/manifest.json` containing fixture SHA-256 checksums.

**Publication status:** The underlying evidence files and reproduction guide have **not been added to this public repository with this report**. Those filenames document the original test package and are intentionally *not* presented as working public links. Full independent reproduction requires publication of the referenced assets and tooling or access to the controlled evaluation package.

The original ZNano codec implementation was not changed for these benchmark runs.

## Emulation boundaries and next validation

Espressif QEMU 9.0.0 ran with `-icount 3`. Virtual elapsed-time observations are diagnostic only, and **must not be used to rank codec throughput**. Physical-board tests are still required for actual runtime speed, Wi-Fi/BLE traffic, power consumption, and battery impact. These radios are not emulated by this fork ([Espressif QEMU reference](https://github.com/espressif/esp-toolchain-docs/blob/main/qemu/README.md)).

**Headline in context:** 270/270 lossless restores, up to 76.79% byte reduction *on the tested regular industrial workload*, zero codec-requested dynamic allocation, and 3,757 B of linked encoder+decoder flash contribution. None of these establishes universal compression or real-device energy gains.
