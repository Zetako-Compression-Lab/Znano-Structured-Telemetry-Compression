# ZNano FPGA — Formal Extension to Real Telemetry Frames

**15 October 2026 · Public engineering summary · Pre-hardware validation**

## Executive result

The latest internal verification campaign extends the FPGA-oriented ZNano encoder's formal evidence to the **actual message widths of all five supplied telemetry profiles** and frame sizes of **6–40 messages**. Its selected matrix completed **144/144 positive proof jobs**: **68 unbounded scoped safety jobs** (induction with base checks) and **76 bounded universal local lemmas**. Two deliberately injected faults yielded the expected counterexamples.

A complementary replay ran **3,150 RTL transactions** on **175 actual-data prefixes** across six implementations and three traffic schedules. Every emitted nibble matched the unchanged C oracle. These results add coverage to earlier campaigns; they are **not** a single top-level proof of C-to-RTL or whole-stream equivalence.

## Workload and capacity

| Supplied telemetry profile | Nibbles/message | Bytes/frame at 6 messages | Bytes/frame at 40 messages |
|---|---:|---:|---:|
| HYDRUS water, short | 72 | 216 | 1,440 |
| HYDRUS water, long | 164 | 492 | 3,280 |
| LANDIS+GYR E450 | 108 | 324 | 2,160 |
| NEO-M8-FW3 | 152 | 456 | 3,040 |
| SHARKY-775-159 | 152 | 456 | 3,040 |

The five profiles originate from four supplied source files; the two HYDRUS widths are separate and are not mixed within a frame. The maximum 3,280-byte frame fits in the 4 KiB compact RAM configuration. File labels do not independently establish device provenance or field representativeness.

## Formal verification scope

The 144 selected jobs are decomposed into memory, column, framing/ordering, address bridge and controller progress obligations. They use arbitrary symbolic payload nibbles rather than fixed test values. The contracts span **700 width/row/configuration combinations**, which is a parameter-domain count, **not 700 separate full-frame proofs**.

- **Memory:** watched-address proofs for concrete RAM, synchronous read behavior, post-load correctness and address bounds.
- **Column classification:** arbitrary watched column, constant/variable decisions and correct value emission.
- **Stream structure:** header ordering, group masks, column-major data ordering, completion and zero padding.
- **Bridge:** checked substitution from logical word address to row/column coordinates.
- **Progress:** local controller-state lemmas under explicit assumptions about input/output handshakes and absence of reset abort.

The memory, column and stream-structure contracts cover five configurable implementations. The progress composition is for **compact-fast** only. The bridge is machine checked, but its use to assemble cross-job contracts is documented mathematically rather than replayed as one automated theorem.

An integrated concrete-RAM classification induction still **timed out at 45 seconds**. Earlier broad formal obligations and full parameterized equivalence remain open. No claim is made that the entire codec is formally proved.

## Real-payload replay and cycle results

The test set consists of **5 profiles × 35 complete prefixes (6–40 messages) = 175 cases**. Across the original RTL and five configurable implementations, under continuous traffic, paused I/O and periodic output backpressure, the campaign ran **3,150 passing transactions** with exact C-reference encoded-output agreement.

| 175 continuous-traffic cases | Total simulated cycles |
|---|---:|
| Original reference RTL | 2,482,057 |
| Compact-fast RTL | 2,203,621 |
| Reduction | **11.218%** |

Encoded output and compression sizes were identical between the compared implementations. These are **simulation cycles**, not achieved FPGA throughput, clock frequency or energy measurements.

The earlier 74,412-transaction regression and 936-transaction follow-up remain separate preserved evidence; they were not rerun in this campaign. The new replay includes overlapping payload prefixes and must not be presented as a count of independent datasets.

## Evidence preservation and next steps

The internal audit checked **2,719 historical file hashes**, including earlier reports and archives. Reference C, released RTL, the wire format and historical deliverables remained unchanged. The new work added verification artifacts only.

Remaining tasks include an automated top-level equivalence/composition certificate, resolution of the integrated RAM/classification timeout, wider message widths and partial groups, RTL decoder validation, post-mapping checks, and real hardware measurement.

**No FPGA board demonstration, vendor place-and-route, physical Fmax, power, energy, or timing closure has been completed.**

This public summary intentionally excludes proprietary RTL, source code, proof harnesses, private telemetry samples, internal logs, and implementation-sensitive verification artifacts.

**Contact:** contact@zetako.ai · https://zetako.ai/
