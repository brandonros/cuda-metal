# LLVM 21 discarded tuple-half proof

The two Base58 entries previously rejected before execution now compile and
numerically pass on Apple M5 with `da434ab`. The original PTX is unchanged.
[Logs, timings, and hashes](llvm21-discarded-half.json).

| Slot | Entry suffix | Result | GPU time |
| --- | --- | --- | --- |
| 41 | `base58_var_len` | Pass, slot = 1 | 147.625 µs |
| 42 | `base58_var_len_leading_zero` | Pass, slot = 1 | 142.000 µs |

Both retain all other 117 result slots and 16 guard words. Together with the
[earlier passes for slots 43–45](llvm21-composed-checks.md), all five composed
fixtures have now passed individually with this LLVM 21 artifact. This is not a
full-suite or mining validation claim.

## Proof and implementation

These programs pack an undefined low 32-bit half and a defined high half into
one 64-bit register, then extract only the high half. The importer previously
required the unobserved half to be defined too.

Before register SSA, the new bounded normalization accepts only:

- An unpredicated, well-formed scalar `mov.b64` pack of two 32-bit registers.
- Exactly one definition and one source occurrence of the packed register across
  the function.
- A later unpredicated `mov.b64` extraction in the same raw block, with exactly
  one 32-bit destination and one discard (`_`) lane.
- No intervening write to the selected source, including predicated writes, and
  no intervening call.

It removes the pack and replaces the extraction with a `mov.b32` of the selected
source. Every intervening instruction stays in place. The defined source still
must pass normal SSA validation; no undefined bit is initialized or materialized.
The proof handles either discarded half, but deliberately does not handle
cross-block extraction, multiple consumers, or changed sources. Those retain
normal validation. Definition/use counts are collected once per function rather
than rescanning the whole function for each candidate.

## Regression evidence

The complete typed PTX/IR/MSL unit executable passes. Positive cases cover both
halves. Negative cases cover reading the undefined half, extracting both halves,
an extra full-width consumer, changed sources (including conditional writes),
cross-block extraction, predicated pack/extraction, and malformed tuple widths
or arity.

Two new Apple-GPU tests pass 65,541 runtime inputs each (131,082 total), including
zero, all-one, sign-bit, and mixed-bit values, with 16 guard words per launch.
Existing tuple-move, guarded-load, guarded-self-select, and bounded-self-select
GPU regressions also pass: six targeted GPU tests total.

```sh
cmake --build build-rust-ptx-apple --target cumetalc cumetal_ptx_ir_msl_test -j 4
build-rust-ptx-apple/tests/unit/cumetal_ptx_ir_msl_test
ctest --test-dir build-rust-ptx-apple \
  -R '^functional_ptx_(discarded_half(_low)?|guarded_load|guarded_self_select|bounded_self_select|tuple_move)$' \
  --output-on-failure
```

## Original artifact reproduction

Input: CUDA 13.3 / NVVM 23, PTX 9.3, sm_100, 25,161,784 bytes;
SHA-256 `2840485fe193d39cfdbb2e5babd25f6772a08a2550843c20b3beba5a26185f4b`.
Neither PTX nor generated Metal was edited or instrumented. Runs use the generic
PTX path, disabled workload specializations, and one GPU thread. Both report
compilation-cache misses. The ledger records the working-tree base commit and
the subsequent commit containing the tested fix separately.

For each entry/slot in the ledger:

```sh
build-rust-ptx-apple/cumetalc "$PTX" --backend=cumetal-ir --ptx-strict \
  --overwrite --entry "$ENTRY" --emit=msl -o "$OUT.metal"
python3 demos/rust-ptx/run_self_test.py "$OUT.metal" \
  --build-dir build-rust-ptx-apple --kernel "$ENTRY" --slot "$SLOT"
```

Local logs/modules: `/tmp/llvm21-discarded-half-validation`. Imports used two
workers with 600-second timeouts; runtime attempts were sequential with
300-second timeouts. Host timings include compilation/setup and are distinct
from GPU timings.
Next bounded batch: tier-2 arithmetic checks, slots 46–56.
