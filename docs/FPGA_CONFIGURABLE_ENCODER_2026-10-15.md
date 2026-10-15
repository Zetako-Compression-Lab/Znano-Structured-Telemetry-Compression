# ZNano FPGA — Configurable Encoder: October 2026 Update

**Research status:** pre-hardware RTL simulation, bounded formal verification, and exploratory FPGA technology mapping. No physical FPGA validation is claimed.

Zetako has evaluated a configurable hardware-oriented ZNano encoder alongside the preserved baseline. The work targets structured telemetry batches, including water and energy metering and GNSS payloads.

## Key measured findings

| Measurement | Result | Scope |
|---|---:|---|
| Compression on supplied original lots | **72.65% weighted reduction** | 22,132 input bytes → 6,052 encoded bytes; 17 original lots, potentially overlapping |
| Aggregate cycle reduction | **11.66%** | Optimised candidate vs baseline, same 17 lots, RTL simulation without pauses |
| Simulation transactions | **74,412 passed** | Six evaluated configurations; zero reported failures |
| C-reference checks | **10,000 round trips + 30,000 selected-line checks passed** | Software oracle |
| Logical frame memory | **4 KiB vs 32 KiB** | Compact vs general configuration |
| Mapped frame RAM blocks | **1 vs 8 Artix-7 RAMB36E1; 2 vs 16 Gowin DPB** | 87.5% fewer frame-memory blocks in these Yosys mappings |

The optimised encoder preserved the canonical encoded bytes on the executed regressions. The memory improvement is **not** a claim of 87.5% smaller total silicon area or lower power. The cycle reduction is **not** a measured improvement in physical board throughput.

### Compression results on supplied telemetry

| Profile | Original batch sizes | Measured reduction range |
|---|---|---:|
| HYDRUS water, 36-byte payloads | 6, 20, 40 messages | 65.28%–77.71% |
| HYDRUS water, 82-byte payloads | 10, 40 messages | 80.61%–90.85% |
| LANDIS+GYR E450, 54-byte payloads | 6, 10, 20, 40 messages | 60.80%–71.48% |
| NEO-M8-FW3, 76-byte payloads | 6, 10, 20, 40 messages | 61.62%–73.68% |
| SHARKY-775-159, 76-byte payloads | 6, 10, 20, 40 messages | 54.61%–65.36% |

Device identifiers reflect labels in supplied input files; field provenance and statistical representativeness are not independently established. The encoded sizes include codec framing and alignment, but not any additional transport/container overhead. Batching introduces application-dependent waiting time.

## Pre-hardware resource comparison

| Configuration | Target | LUT primitives | Flip-flops | DSP | Frame RAM blocks |
|---|---|---:|---:|---:|---:|
| Baseline | Artix-7 | 406 | 222 | 3 | 8 |
| Optimised compact | Artix-7 | 454 | 254 | 1 | 1 |
| Baseline | Gowin GW2A | 586 | 250 | 2 | 16 |
| Optimised compact | Gowin GW2A | 550 | 268 | 1 | 2 |

Figures are technology-mapping estimates **before place-and-route**; cross-vendor LUT primitives are not directly comparable. The compact configuration accepts the supplied application batches and rejects oversized transactions. The general-capacity configuration remains available for larger batches.

## Verification coverage and limitations

- RTL output checked against the preserved C oracle on the reported regression cases, including flow-control pauses and reset/error scenarios.
- Scoped interface/reset safety proved by induction under stated reduced-configuration assumptions.
- Several bounded symbolic transaction checks passed, including a six-message case; other formal jobs **timed out** and remain incomplete.
- No claim of full parameterised formal equivalence, post-synthesis equivalence, or universal liveness.
- No physical board tests, timing closure/Fmax, power/energy, vendor place-and-route, or sustained I/O throughput.
- The RTL implementation currently covers the **encoder**, not a hardware decoder. C software decoding was used for restoration checks.
- Vendor-specific RAM-mapping warnings remain to be assessed in vendor tools.

## Next industrial milestone

Zetako is seeking technical evaluation of the encoder on a real FPGA platform: vendor synthesis and memory inference review, place-and-route, measured frequency and latency, resource usage, power per payload, and independent replay of representative telemetry workloads.

This public update intentionally contains **engineering outcomes and validation boundaries only**. It does not include proprietary RTL, algorithms, test vectors, implementation scripts, or internal architecture details.

**Contact:** contact@zetako.ai · https://zetako.ai/
