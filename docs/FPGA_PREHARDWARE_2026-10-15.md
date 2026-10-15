# ZNano FPGA — Pre-Hardware Engineering Results (October 2026)

**Status:** RTL simulation and exploratory technology mapping. **Not yet validated on a physical FPGA.**

ZNano's hardware work extends its structured-telemetry lossless compression technology toward FPGA implementation and, potentially, embedded hardware IP integration. The current evidence concerns the **encoder RTL**, without an RTL decoder, board-level interfaces, physical timing closure, or power measurement.

## Highlights

| Evidence | Result | Interpretation |
|---|---:|---|
| Original application batches | 17 | Five payload-size profiles across four named device families |
| Original input / encoded output | 22,132 / 6,052 bytes | 72.65% weighted byte reduction on the supplied batches |
| Reduction across original batches | 54.61%–90.85% | Workload-specific; not a universal compression guarantee |
| Application transactions | 156, zero failures | Original batches and exact prefixes across three flow-control scenarios |
| Output-stability checks | 51,964 passed | Simulation checks, not a formal proof |
| Additional golden transactions | 10,000 no-stall + 2,000 stalled, zero failures | Separate regression campaign |
| Technology-mapping targets | Artix-7 and Gowin GW2A | Exploratory synthesis; no placed/routed implementation |

The RTL encoded outputs matched the C reference for the tested transactions. The C reference also checked byte-exact restoration for the original batches; **an RTL decoder was not tested**. The 17 original batches may overlap and should not be interpreted as 22,132 unique bytes of field observations. Device labels identify supplied files, not independently established field provenance.

## Workload evidence

| Device/profile | Payload size | Original batch sizes | Observed reduction |
|---|---:|---|---:|
| HYDRUS water meter | 36 B | 6, 20, 40 | 65.28%–77.71% |
| HYDRUS water meter | 82 B | 10, 40 | 80.61%–90.85% |
| LANDIS+GYR E450 | 54 B | 6, 10, 20, 40 | 60.80%–71.48% |
| NEO-M8-FW3 | 76 B | 6, 10, 20, 40 | 61.62%–73.68% |
| SHARKY-775-159 | 76 B | 6, 10, 20, 40 | 54.61%–65.36% |

The 72.65% aggregate includes the ZNano stream's framing/alignment but excludes additional transport and container overhead. Six messages is the chosen application minimum, not a universal codec limit. Batch accumulation may introduce latency.

## FPGA resource mapping — encoder core only

| Target/configuration | LUT primitives | Flip-flops | DSP multipliers | Block RAM |
|---|---:|---:|---:|---|
| Artix-7, DSP enabled | 406 | 222 | 3 | 8 × 36 Kbit |
| Artix-7, DSP disabled | 1,533 | 222 | 0 | 8 × 36 Kbit |
| Gowin GW2A, DSP enabled | 586 | 250 | 2 | 16 × 18 Kbit |
| Gowin GW2A, DSP disabled | 2,805 | 250 | 0 | 16 × 18 Kbit |

These are Yosys post-mapping primitive counts, **not placed-and-routed device utilization**. LUT primitives are not directly comparable across vendors. The core currently reserves 32 KiB of logical frame memory (36 KiB of physical block RAM in these mappings). The measured application batches are much smaller; compact memory configurations remain an engineering opportunity, not a completed optimization.

## Cycle-level simulation

For the 17 original transactions, the weighted input throughput calculated from measured RTL cycles would be **5.194 MB/s at an assumed 50 MHz** (2.597 MB/s at 25 MHz; 10.389 MB/s at 100 MHz). These are **conditional clock conversions**, not achieved board throughput, Fmax, interface throughput, or power results.

Flow-control simulations included no stalls, input/output stalls, and prolonged output backpressure. Reset/recovery and protocol checks also passed in the reported test benches.

## Readiness and next validation stage

**Completed:** RTL encoder development; matching C-reference encoded output on the reported test suite; flow-control and reset testing; exploratory mapping to two FPGA families.

**Not yet completed:** vendor place-and-route, timing closure, bitstream generation, board demonstration, power/energy measurements, end-to-end hardware decoder, sustained I/O throughput, or independent industrial validation.

The next practical milestone is an external FPGA implementation and measurement campaign: timing/Fmax, LUT/FF/DSP/BRAM utilization after place-and-route, throughput and latency under real I/O, energy per payload, and byte-exact verification. Zetako is exploring collaborations with European FPGA engineering teams and embedded hardware partners.

## Scope of publication

This page reports **selected public-facing results** from Zetako's internal October 15, 2026 engineering reports. It does not publish RTL source, proprietary algorithms, private test vectors, internal scripts, or implementation-sensitive architecture details. The results are internal engineering evidence, not third-party certification.

For evaluation and collaboration: **contact@zetako.ai** · https://zetako.ai/
