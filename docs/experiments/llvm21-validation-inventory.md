# Complete LLVM 21 validation inventory

For artifact SHA-256 `2840485fe193d39cfdbb2e5babd25f6772a08a2550843c20b3beba5a26185f4b`:

- **54 / 118 numerical self-tests have recorded passes.**
- **64 / 118 numerical self-tests remain unvalidated.**
- The separate launch probe passes. It also writes slot 0, but does not validate
  the xoroshiro numerical entry that uses that slot.
- All four mining kernels remain unvalidated end to end.
- All recorded LLVM 21 compiler failures have later passing evidence; there is
  no currently unresolved observed failure in this ledger. Unvalidated entries
  may still reveal failures.

These are cumulative passes across compiler revisions, **not** a fresh full-suite
pass on the current compiler. Earlier LLVM 7/19 evidence is excluded. After the
remaining entries are tested and fixes are made, rerun the complete inventory on
one final compiler/runtime build. Fixed known-answer fixtures do not replace
runtime-input and concurrent workload validation.

## Every unvalidated numerical entry

Names below omit the common `kernel_self_test_` prefix.

| Slot | Entry suffix |
| --- | --- |
| 0 | `primitive_xoroshiro` |
| 3 | `primitive_base58` |
| 6 | `primitive_keccak256` |
| 7 | `primitive_ripemd160` |
| 8 | `primitive_sha256_32` |
| 9 | `primitive_sha256_variable` |
| 10 | `solana_priv` |
| 11 | `solana_pub` |
| 12 | `solana_encoded` |
| 63 | `iter_static_slice_lookup` |
| 64 | `arith_divrem_by_58_pow_5` |
| 65 | `arith_i128_chain_add` |
| 66 | `base58_limb_divrem` |
| 67 | `dynamic_index_write` |
| 68 | `arith_widening_mul_chain_3term` |
| 69 | `base58_inner_mutate_phase` |
| 70 | `dalek_clamp_integer` |
| 71 | `dalek_scalar_round_trip_one` |
| 72 | `dalek_mul_base_scalar_one` |
| 73 | `k256_secret_from_bytes_one` |
| 74 | `k256_derive_scalar_one` |
| 75 | `k256_derive_scalar_two` |
| 76 | `static_u64_array_lookup` |
| 77 | `static_struct_wrapped_u64_lookup` |
| 78 | `k256_encode_generator` |
| 79 | `k256_double_generator` |
| 80 | `k256_scalar_one_round_trip` |
| 81 | `arith_u128_imm_shr_52` |
| 82 | `static_depth4_newtype_nesting` |
| 83 | `reverse_range_write` |
| 84 | `dalek_scalar52_from_bytes` |
| 85 | `dalek_scalar52_montgomery_reduce_r` |
| 86 | `dalek_scalar52_mul_internal_then_reduce_one_r` |
| 87 | `dalek_scalar52_as_bytes_one` |
| 88 | `dalek_scalar52_sub_no_underflow` |
| 89 | `dalek_scalar52_sub_with_underflow` |
| 90 | `dalek_scalar52_montgomery_reduce_with_sub` |
| 91 | `index_trait_dispatch` |
| 92 | `dalek_scalar_one_to_bytes_direct` |
| 93 | `k256_affine_generator_encode` |
| 94 | `subtle_choice_u8_into_bool` |
| 95 | `subtle_conditional_select_u64` |
| 96 | `k256_encoded_point_from_affine_coords` |
| 97 | `index_trait_const_indices` |
| 98 | `generic_array_basic_index` |
| 99 | `generic_array_copy_from_slice` |
| 100 | `from_affine_coords_replica` |
| 101 | `generic_array_as_slice_last` |
| 102 | `dalek_scalar_round_trip_zero` |
| 103 | `dalek_scalar_from_bytes_wide_zero` |
| 104 | `field_bytes_into_conversion` |
| 105 | `base58_min_nonzero` |
| 106 | `named_field_struct_return` |
| 107 | `base58_handrolled_no_seq` |
| 108 | `slice_reverse_partial` |
| 109 | `dalek_scalar_eq_zero` |
| 110 | `generic_array_copy_from_ga_source` |
| 111 | `dalek_zero_eq_zero` |
| 112 | `dalek_from_canonical_zero` |
| 113 | `dalek_scalar52_from_bytes_zero` |
| 114 | `dalek_scalar52_mul_internal_zero` |
| 115 | `dalek_scalar52_montgomery_reduce_zero` |
| 116 | `dalek_scalar52_as_bytes_zero` |
| 117 | `dalek_reduce_pipeline_zero` |

## Mining kernels and remaining execution coverage

All four require deterministic CPU/GPU comparison and end-to-end result handling:

- `kernel_find_better_shallenge_nonce`
- `kernel_find_bitcoin_vanity_private_key`
- `kernel_find_ethereum_vanity_private_key`
- `kernel_find_solana_vanity_private_key`

The existing mining checklist also calls for positive and negative matches,
result-buffer bounds, no-result behavior, multiple threads, repeated launches,
and deterministic repeatability. Validate returned key/address/nonce bytes as
applicable, not merely successful launch. Fixed-size self-test launches do not
establish this coverage.

Additional outstanding work from the existing checklist:

- Audit each entry and reachable helpers for constant folding; Rust `black_box`
  alone does not prove the intended arithmetic survived compilation.
- Add host-input primitive kernels returning complete digests/public keys and
  compare them with independent CPU references.
- Broaden Ed25519 inputs and clamping/scalar boundaries; cover hash block/padding
  boundaries, encoding byte patterns, and leading zeros.
- Exercise arithmetic boundary inputs for carry/borrow, high-bit products,
  shifts, signed comparisons, and narrow/wide conversions.
- Test launch boundaries such as 1/31/32/33/257 threads with per-thread guards,
  and repeated launches/supported configurations.
- Automate the regression set on a suitable Apple GPU runner; registered CTests
  alone do not constitute a GitHub GPU CI gate.
- Validate each mining mode through the intended host/CLI integration after its
  standalone harness passes. PTX launch success is not CLI compatibility.
- Benchmark only after correctness gates pass, reporting performance separately.
- Independently review and test the extracted upstream issue patches against
  current upstream; passing this fork is not upstream acceptance.

Suggested next batch: fill the early gaps—slots 0, 3, and 6–12—then continue with
63–117 in bounded groups.

## All slots and evidence

[Machine-readable inventory](llvm21-validation-inventory.json). Passes require
an explicit numerical-pass log for the exact entry name. Original failures remain
in their historical reports; later passes resolve them in this aggregate view.

| Slot | Entry suffix | Status / evidence |
| --- | --- | --- |
| 0 | `primitive_xoroshiro` | Unvalidated |
| 1 | `primitive_sha512` | [llvm21-miner-smoke.json](llvm21-miner-smoke.json) |
| 2 | `primitive_ed25519` | [llvm21-loop-definedness.json](llvm21-loop-definedness.json) |
| 3 | `primitive_base58` | Unvalidated |
| 4 | `primitive_secp256k1_compressed` | [llvm21-guarded-load.json](llvm21-guarded-load.json) |
| 5 | `primitive_secp256k1_uncompressed` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 6 | `primitive_keccak256` | Unvalidated |
| 7 | `primitive_ripemd160` | Unvalidated |
| 8 | `primitive_sha256_32` | Unvalidated |
| 9 | `primitive_sha256_variable` | Unvalidated |
| 10 | `solana_priv` | Unvalidated |
| 11 | `solana_pub` | Unvalidated |
| 12 | `solana_encoded` | Unvalidated |
| 13 | `ethereum_priv` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 14 | `ethereum_pub` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 15 | `ethereum_address` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 16 | `bitcoin_priv` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 17 | `bitcoin_pub` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 18 | `bitcoin_pkh` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 19 | `bitcoin_encoded` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 20 | `bitcoin_matches` | [llvm21-address-checks.json](llvm21-address-checks.json) |
| 21 | `wif_compressed_mainnet` | [llvm21-wif-checks.json](llvm21-wif-checks.json) |
| 22 | `wif_uncompressed_mainnet` | [llvm21-wif-checks.json](llvm21-wif-checks.json) |
| 23 | `wif_compressed_testnet` | [llvm21-wif-checks.json](llvm21-wif-checks.json) |
| 24 | `wif_uncompressed_testnet` | [llvm21-wif-checks.json](llvm21-wif-checks.json) |
| 25 | `shallenge_hash` | [llvm21-shallenge-checks.json](llvm21-shallenge-checks.json) |
| 26 | `shallenge_nonce_len` | [llvm21-shallenge-checks.json](llvm21-shallenge-checks.json) |
| 27 | `shallenge_is_better` | [llvm21-shallenge-checks.json](llvm21-shallenge-checks.json) |
| 28 | `compare_hashes_lt` | [llvm21-hash-comparison-checks.json](llvm21-hash-comparison-checks.json) |
| 29 | `compare_hashes_gt` | [llvm21-hash-comparison-checks.json](llvm21-hash-comparison-checks.json) |
| 30 | `compare_hashes_eq` | [llvm21-hash-comparison-checks.json](llvm21-hash-comparison-checks.json) |
| 31 | `arith_u32_div_var` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 32 | `arith_u32_div_const` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 33 | `arith_u64_div_var` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 34 | `arith_u64_div_const` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 35 | `arith_u32_rem_var` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 36 | `arith_u64_rem_var` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 37 | `arith_u32_mul_lo` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 38 | `arith_u64_mul_lo` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 39 | `arith_u64_mul_hi` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 40 | `arith_u128_mul` | [llvm21-arithmetic-checks.json](llvm21-arithmetic-checks.json) |
| 41 | `base58_var_len` | [llvm21-discarded-half.json](llvm21-discarded-half.json) |
| 42 | `base58_var_len_leading_zero` | [llvm21-discarded-half.json](llvm21-discarded-half.json) |
| 43 | `base58_all_zeros` | [llvm21-composed-checks.json](llvm21-composed-checks.json) |
| 44 | `xoroshiro_base64_nonce` | [llvm21-composed-checks.json](llvm21-composed-checks.json) |
| 45 | `bech32_p2wpkh` | [llvm21-composed-checks.json](llvm21-composed-checks.json) |
| 46 | `arith_overflowing_add` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 47 | `arith_overflowing_sub` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 48 | `arith_carry_chain_3limb` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 49 | `arith_widening_mul_pair` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 50 | `arith_mad_lo_u64` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 51 | `arith_mad_hi_u64` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 52 | `arith_mul_wide_u32` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 53 | `arith_mask_blend_true` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 54 | `arith_mask_blend_false` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 55 | `arith_var_shr_u64` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 56 | `arith_var_shl_u64` | [llvm21-tier2-arithmetic-checks.json](llvm21-tier2-arithmetic-checks.json) |
| 57 | `arith_blackbox_identity_u64` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 58 | `arith_blackbox_identity_u32` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 59 | `base58_div_by_58` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 60 | `iter_static_table_lookup` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 61 | `iter_mut_slice_partial` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 62 | `iter_mut_alphabet_lookup` | [llvm21-identity-iteration-checks.json](llvm21-identity-iteration-checks.json) |
| 63 | `iter_static_slice_lookup` | Unvalidated |
| 64 | `arith_divrem_by_58_pow_5` | Unvalidated |
| 65 | `arith_i128_chain_add` | Unvalidated |
| 66 | `base58_limb_divrem` | Unvalidated |
| 67 | `dynamic_index_write` | Unvalidated |
| 68 | `arith_widening_mul_chain_3term` | Unvalidated |
| 69 | `base58_inner_mutate_phase` | Unvalidated |
| 70 | `dalek_clamp_integer` | Unvalidated |
| 71 | `dalek_scalar_round_trip_one` | Unvalidated |
| 72 | `dalek_mul_base_scalar_one` | Unvalidated |
| 73 | `k256_secret_from_bytes_one` | Unvalidated |
| 74 | `k256_derive_scalar_one` | Unvalidated |
| 75 | `k256_derive_scalar_two` | Unvalidated |
| 76 | `static_u64_array_lookup` | Unvalidated |
| 77 | `static_struct_wrapped_u64_lookup` | Unvalidated |
| 78 | `k256_encode_generator` | Unvalidated |
| 79 | `k256_double_generator` | Unvalidated |
| 80 | `k256_scalar_one_round_trip` | Unvalidated |
| 81 | `arith_u128_imm_shr_52` | Unvalidated |
| 82 | `static_depth4_newtype_nesting` | Unvalidated |
| 83 | `reverse_range_write` | Unvalidated |
| 84 | `dalek_scalar52_from_bytes` | Unvalidated |
| 85 | `dalek_scalar52_montgomery_reduce_r` | Unvalidated |
| 86 | `dalek_scalar52_mul_internal_then_reduce_one_r` | Unvalidated |
| 87 | `dalek_scalar52_as_bytes_one` | Unvalidated |
| 88 | `dalek_scalar52_sub_no_underflow` | Unvalidated |
| 89 | `dalek_scalar52_sub_with_underflow` | Unvalidated |
| 90 | `dalek_scalar52_montgomery_reduce_with_sub` | Unvalidated |
| 91 | `index_trait_dispatch` | Unvalidated |
| 92 | `dalek_scalar_one_to_bytes_direct` | Unvalidated |
| 93 | `k256_affine_generator_encode` | Unvalidated |
| 94 | `subtle_choice_u8_into_bool` | Unvalidated |
| 95 | `subtle_conditional_select_u64` | Unvalidated |
| 96 | `k256_encoded_point_from_affine_coords` | Unvalidated |
| 97 | `index_trait_const_indices` | Unvalidated |
| 98 | `generic_array_basic_index` | Unvalidated |
| 99 | `generic_array_copy_from_slice` | Unvalidated |
| 100 | `from_affine_coords_replica` | Unvalidated |
| 101 | `generic_array_as_slice_last` | Unvalidated |
| 102 | `dalek_scalar_round_trip_zero` | Unvalidated |
| 103 | `dalek_scalar_from_bytes_wide_zero` | Unvalidated |
| 104 | `field_bytes_into_conversion` | Unvalidated |
| 105 | `base58_min_nonzero` | Unvalidated |
| 106 | `named_field_struct_return` | Unvalidated |
| 107 | `base58_handrolled_no_seq` | Unvalidated |
| 108 | `slice_reverse_partial` | Unvalidated |
| 109 | `dalek_scalar_eq_zero` | Unvalidated |
| 110 | `generic_array_copy_from_ga_source` | Unvalidated |
| 111 | `dalek_zero_eq_zero` | Unvalidated |
| 112 | `dalek_from_canonical_zero` | Unvalidated |
| 113 | `dalek_scalar52_from_bytes_zero` | Unvalidated |
| 114 | `dalek_scalar52_mul_internal_zero` | Unvalidated |
| 115 | `dalek_scalar52_montgomery_reduce_zero` | Unvalidated |
| 116 | `dalek_scalar52_as_bytes_zero` | Unvalidated |
| 117 | `dalek_reduce_pipeline_zero` | Unvalidated |
