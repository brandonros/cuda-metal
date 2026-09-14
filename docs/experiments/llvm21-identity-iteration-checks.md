# LLVM 21 identity and iteration validation

All six entries in slots 57–62 compile and numerically pass on Apple M5 at
CuMetal `e0f1698`, without compiler changes.
[Per-entry logs, timings, and hashes](llvm21-identity-iteration-checks.json).

| Slot | Check | Result | Import (s) | Runtime attempt (s) | GPU (µs) |
| --- | --- | --- | --- | --- | --- |
| 57 | `arith_blackbox_identity_u64` | Pass | 71.4 | 0.404 | 1.708 |
| 58 | `arith_blackbox_identity_u32` | Pass | 70.3 | 0.241 | 1.625 |
| 59 | `base58_div_by_58` | Pass | 112.9 | 0.243 | 1.541 |
| 60 | `iter_static_table_lookup` | Pass | 112.2 | 0.242 | 1.999 |
| 61 | `iter_mut_slice_partial` | Pass | 70.4 | 0.338 | 4.000 |
| 62 | `iter_mut_alphabet_lookup` | Pass | 72.0 | 0.335 | 5.000 |

Each selected slot equals 1; the other 117 slots and all 16 guard words remain
untouched. The original full LLVM 21 PTX and generated Metal are unchanged and
uninstrumented. Generic PTX launches use one thread and disabled workload
specializations. All report compilation-cache misses. Compiler/runtime/PTX hashes
and the branch commit matched startup values at completion.

The fixtures check u64/u32 black-box identity, isolated Base58 division arithmetic,
a fixed table index, partial mutable-slice iteration, and iteration with alphabet
lookup. The latter two check representative modified and untouched buffer
positions, not every possible index or input. These are fixed fixtures rather
than exhaustive or concurrent mining coverage.

Input: CUDA 13.3 / NVVM 23, PTX 9.3, sm_100, 25,161,784 bytes;
SHA-256 `2840485fe193d39cfdbb2e5babd25f6772a08a2550843c20b3beba5a26185f4b`.

## Reproduce

For each entry/slot from the JSON ledger:

```sh
build-rust-ptx-apple/cumetalc "$PTX" --backend=cumetal-ir --ptx-strict \
  --overwrite --entry "$ENTRY" --emit=msl -o "$OUT.metal"
python3 demos/rust-ptx/run_self_test.py "$OUT.metal" \
  --build-dir build-rust-ptx-apple --kernel "$ENTRY" --slot "$SLOT"
```

Imports used two workers with 600-second timeouts; runtime attempts were sequential
with 300-second timeouts. Host runtime timings include Metal compilation/setup;
GPU timings are separate. Local output: `/tmp/llvm21-identity-iteration-validation`.

[Complete cumulative LLVM 21 inventory and all remaining work](llvm21-validation-inventory.md).
