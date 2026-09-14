# LLVM 21 tier-2 arithmetic validation

All eleven entries in slots 46–56 compile and numerically pass on Apple M5
at CuMetal `0c9d89b`. No compiler changes were needed.
[Per-entry logs, timings, and hashes](llvm21-tier2-arithmetic-checks.json).

| Slot | Check | Result | Import (s) | Runtime attempt (s) | GPU (µs) |
| --- | --- | --- | --- | --- | --- |
| 46 | Overflowing addition | Pass | 68.0 | 0.340 | 1.499 |
| 47 | Overflowing subtraction | Pass | 67.4 | 0.236 | 1.541 |
| 48 | Three-limb carry chain | Pass | 61.5 | 0.296 | 75.958 |
| 49 | Paired widening product | Pass | 61.4 | 0.299 | 1.458 |
| 50 | Low-product multiply-add | Pass | 63.3 | 0.243 | 1.625 |
| 51 | High-product multiply-add | Pass | 63.1 | 0.184 | 1.708 |
| 52 | 32×32→64 multiplication | Pass | 61.8 | 0.241 | 1.624 |
| 53 | True mask blend | Pass | 62.4 | 0.238 | 34.666 |
| 54 | False mask blend | Pass | 81.3 | 0.341 | 1.500 |
| 55 | Variable right shift | Pass | 84.0 | 0.253 | 1.708 |
| 56 | Variable left shift | Pass | 72.2 | 0.183 | 1.500 |

Every selected result slot equals 1, while the other 117 slots and 16 guard words
remain untouched. These are separate one-thread launches through the generic PTX
path, with workload specializations disabled. All traces report compilation-cache
misses. No PTX or generated Metal was edited or instrumented.

## Coverage and limits

Addition and subtraction each check three cases, including normal arithmetic,
wraparound boundaries, and overflow/borrow flags. The carry-chain fixture checks
propagation across three 64-bit limbs. Product fixtures check paired high/low
results, low/high-product addition, and 32-bit inputs widened to 64 bits. The mask
fixtures check both boolean outcomes. Shift fixtures use black-boxed operands and
shift counts.

These are fixed Rust fixtures, not exhaustive arithmetic or randomized-input
coverage. A source-level multiply-add check does not by itself prove that a
particular PTX opcode was exercised. Historical comments about broken arithmetic
in the Rust source are not current findings. Full-suite and mining validation
remain incomplete; LLVM 7/19 totals are unchanged.

## Provenance and reproduction

The input remains the full 25,161,784-byte LLVM 21 artifact: CUDA 13.3 / NVVM 23,
PTX 9.3, sm_100, SHA-256
`2840485fe193d39cfdbb2e5babd25f6772a08a2550843c20b3beba5a26185f4b`.
The compiler, runtime, PTX hashes and branch commit were checked again after the
batch and match startup values. Two workers compiled entries; GPU launches were
sequential. Host runtime-attempt times include Metal compilation/setup; GPU times
come from launch traces.

For each entry/slot from the ledger:

```sh
build-rust-ptx-apple/cumetalc "$PTX" --backend=cumetal-ir --ptx-strict \
  --overwrite --entry "$ENTRY" --emit=msl -o "$OUT.metal"
python3 demos/rust-ptx/run_self_test.py "$OUT.metal" \
  --build-dir build-rust-ptx-apple --kernel "$ENTRY" --slot "$SLOT"
```

Imports were bounded at 600 seconds and runtime attempts at 300 seconds.
Local modules, ABI sidecars and logs: `/tmp/llvm21-tier2-validation`.
Next bounded batch: identity and Base58/iteration checks, slots 57–62.
