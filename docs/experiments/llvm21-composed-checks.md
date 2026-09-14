# LLVM 21 composed-primitive validation

At CuMetal `1e3c01e`, three entries numerically pass on Apple M5 and two fail
register-definedness import before GPU execution.
[Per-entry logs, timings, and hashes](llvm21-composed-checks.json).

| Slot | Check | Result |
| --- | --- | --- |
| 41 | Variable-length Base58 | Import failure: `%r100` undefined |
| 42 | Variable-length Base58 with leading zero | Import failure: `%r116` undefined |
| 43 | Base58 of 32 zero bytes | Numerical pass |
| 44 | Xoroshiro/Base64 nonce | Numerical pass |
| 45 | Bech32 P2WPKH address | Numerical pass |

Each passing entry returns 1 in its selected slot; the other 117 slots and all
16 guards remain intact. Base58 and Bech32 fixtures compare complete output and
length; the nonce fixture compares all 21 bytes for seed 12345 and logical thread
0. These are fixed fixtures, not exhaustive runtime-input or mining validation.

## Shared blocker

Slot 41 contains:

```ptx
ld.local.v4.b32 {%r96, %r97, %r98, %r99}, [%rd1];
mov.b64 %rd180, {%r100, %r99};
// intervening instructions
mov.b64 {_, %r5}, %rd180;
```

`%r100` has no definition. Its only textual use is the low half of this pack;
`%rd180` is subsequently used only to extract the high half, discarding the low
half. Slot 42 has the same shape with `%r116`, `%r115`, `%rd214`, and `%r6`.
The errors propagate to incoming edges of `$L__BB37_1` and `$L__BB38_1`.
This evidence points to missing reasoning about unused halves of packed values,
not a demonstrated numerical Base58 failure.

Next fix: prove the discarded half is unobservable before register SSA, retaining
rejection when that half is read. Add positive and negative regressions and retry
both original entries. Do not initialize the missing register or edit the PTX.

## Provenance and reproduction

CuMetal stayed on `poc/rust-ptx-harness` at `1e3c01e` throughout the run. Its
working tree was clean when the concurrent-agent concern was checked. Compiler,
runtime, and input PTX hashes match their recorded startup values, including a
second check after the batch. No current branch change explains these failures.
Changes to another Rust-CUDA checkout do not change this prebuilt input artifact.

Original input: full LLVM 21 PTX, CUDA 13.3 / NVVM 23, PTX 9.3, sm_100,
25,161,784 bytes, SHA-256
`2840485fe193d39cfdbb2e5babd25f6772a08a2550843c20b3beba5a26185f4b`.
PTX and generated Metal remain unchanged and uninstrumented. Workload
specializations are disabled. Imports used two workers; GPU launches were
sequential, with one thread each. Host runtime timings include Metal compilation
and setup; GPU timings are recorded separately in the JSON ledger.

For each entry and slot from that ledger:

```sh
build-rust-ptx-apple/cumetalc "$PTX" --backend=cumetal-ir --ptx-strict \
  --overwrite --entry "$ENTRY" --emit=msl -o "$OUT.metal"
# Only after successful compilation:
python3 demos/rust-ptx/run_self_test.py "$OUT.metal" \
  --build-dir build-rust-ptx-apple --kernel "$ENTRY" --slot "$SLOT"
```

Imports were bounded at 600 seconds; runtime attempts at 300 seconds.
Local output: `/tmp/llvm21-composed-validation`. The full LLVM 21 suite and mining
kernels remain unvalidated; LLVM 7/19 historical totals are unchanged.
