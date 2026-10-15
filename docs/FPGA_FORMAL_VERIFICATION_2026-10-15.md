# ZNano FPGA — Decomposed Formal Verification Update

**Date:** 15 October 2026  
**Evidence:** Internal RTL formal verification and simulation; no physical FPGA hardware was used.

## Summary

A follow-up verification campaign for ZNano's configurable FPGA encoder completed **174 positive proof jobs**, all passing: **132 unbounded safety jobs** established through induction with matching bounded base cases, and **42 bounded complete-transaction proof partitions**. Two deliberately corrupted variants produced expected counterexamples and are not counted as positive jobs.

The new proof strategy decomposes previously monolithic verification problems into protocol/reset safety, control/address safety, synchronous RAM behavior, and small-frame transaction correctness. All selected final positive jobs completed within their assigned solver budgets (at most 45 seconds per invocation; separate loading covers used 15 seconds). This **does not** mean every previously timed-out broader property has been proven.

## What was proved

| Verification area | Evidence and scope |
|---|---|
| Interface and reset safety | Unbounded induction with concrete RAM across five compact/general configurations, arbitrary interface pauses and resets after an initial reset |
| Loading/write safety | Unbounded control proofs with arbitrary 16-bit dimensions and input activity, under stated initial reset assumptions |
| Read/write bounds | 96 fixed-shape unbounded jobs for 32 dimension pairs across three compact configurations, plus 20 variable-row jobs for four telemetry widths across five configurations |
| RAM behavior | Watched-address synchronous read refinement for the concrete 8,192 × 4-bit (4 KiB) RAM |
| Transaction correctness | 42 bounded proof partitions, including all 32 constant/variable classification masks for five columns × two rows on compact-fast, plus 10 one-column/six-row partitions |
| Negative controls | Two intentionally faulty designs correctly produced counterexamples |

The transaction jobs retain symbolic payload bits and compare the encoded output against an independent small-frame wire-format oracle. Their assumptions include an initial reset, one valid start, contiguous input, and an always-ready output. They **do not** establish full parameterized end-to-end equivalence for realistic telemetry frame widths, or arbitrary-stall/reset transaction correctness.

## Regression and preservation

- The prior **74,412 passing RTL simulation transactions** remain a separate earlier result; they were not rerun in full during this formal follow-up.
- A fresh replay of **936 customer transactions** across six configurations matched the retained output and cycle measurements exactly.
- **800 protected files** were confirmed unchanged by SHA-256, including the existing RTL, C reference, encoded format and earlier engineering artifacts.
- No new hardware mapping was performed. Earlier Artix-7/Gowin resource estimates remain applicable to unchanged source designs.

## Remaining obligations

Universal read-address proof for all arbitrary dimension pairs, symbolic full-width transaction equivalence with multi-nibble headers, arbitrary-stall/reset end-to-end correctness, and hardware-vendor netlist/primitive verification remain open. Some earlier combined proof attempts timed out; those records remain explicitly incomplete.

**No board execution, vendor place-and-route, physical Fmax, energy/power measurement or hardware decoder validation has been completed.**

## Context

This update supplements the [October configurable FPGA encoder evaluation](FPGA_CONFIGURABLE_ENCODER_2026-10-15.md) and the [earlier pre-hardware report](FPGA_PREHARDWARE_2026-10-15.md). It publishes verification outcomes and scope only, without proprietary RTL, proof harnesses, internal datasets or implementation-sensitive details.

For technical and industrial evaluation enquiries: **contact@zetako.ai** · https://zetako.ai/
