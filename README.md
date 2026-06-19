
> **Practical Considerations in Repairing Reed–Solomon Codes (Implementation)**
> Thi Xinh Dinh, **Nhi Nguyen**, Lakshmi J. Mohan, Serdar Boztaş,
> Tran Thi Luong, and Son Hoang Dau.
> *2022 IEEE International Symposium on Information Theory (ISIT)*, pp. 2607–2612.
> DOI: [10.1109/ISIT50566.2022.9834830](https://doi.org/10.1109/ISIT50566.2022.9834830)
> · IEEE Xplore: <https://ieeexplore.ieee.org/document/9834830>
> · arXiv: [2205.11015](https://arxiv.org/abs/2205.11015)

The paper studies how to repair a single failed node of a Reed–Solomon (RS) code
with **low repair bandwidth** under the *trace-repair* framework, and reports
tables of best-known repair schemes for short-length RS codes over GF(256). This
repository contains the implementation used to construct and evaluate those
schemes.

## Background

The implementation is built on top of Intel's
[ISA-L](https://github.com/intel/isa-l) (Intelligent Storage Acceleration
Library) erasure-code module. ISA-L provides fast, SIMD-accelerated Galois-field
arithmetic and RS encoding/decoding (the `gf_*vect_*` `.asm` kernels,
`ec_base.c`, `ec_highlevel_func.c`, `erasure_code.h`, etc.). We use it as the
high-performance GF(2⁸) back end and add a trace-repair layer on top.

## What we added

On top of the stock ISA-L erasure-code files, this repo adds:

| File | Purpose |
| --- | --- |
| `repair_scheme.c` | Core trace-repair implementation over GF(2⁸): trace evaluation (`repair_trace`), binary-gap / trace helpers (`binGap`, `binSum`, `binRep`), RS/Cauchy generator matrices, and GF arithmetic. Builds and exercises a repair scheme for an `[n, k]` RS code (e.g. RS(9, 6)). |
| `repair_scheme_tbl.h` | Precomputed best-known repair-scheme tables (the schemes tabulated in the paper). |
| `repair_test.c` | Standalone tests for the repair building blocks (trace, binary gap, etc.). |
| `erasure_code_test_optimised.c`, `erasure_code_test_optimised_JC.c` | Repair benchmark that wires the trace-repair scheme into ISA-L's optimised SIMD GF routines to measure repair cost/throughput. |
| `testRS96.c`, `RandomCodeword96.txt`, `testCodewordISAL.txt` | RS test codewords and a driver used to validate repair against known codewords. |
| `F16_calculator.c`, `FFC.c`, `FINITE FIELD CALCULATOR.c`, `testf16.c` | Finite-field calculators used while designing and checking the schemes. |
| `binCon.c`, `binToArr.c`, `bit.c`, `bitshift.c`, `Hex_conv.c`, `cauchy_matrix.c`, `convert_table.c`, `matrix.c`, `Sort.c`, `Ini_tb.c`, `htbl.c`, `XOR.c`, `XORnote.c`, `EQUATION.C`, `output.c` | Small utilities and experiments (bit/hex/format conversion, matrix and table helpers, XOR-repair notes) developed alongside the main scheme. |

Files such as `erasure_code_test.c`, `erasure_code_test-original.c`, and the
`gf_*vect_*` kernels are ISA-L originals (some lightly modified for our tests);
`-original` marks an unmodified upstream copy kept for reference.

## Repository layout

```
erasure_code/
├── repair_scheme.c / .h-tbl     # trace-repair scheme + precomputed tables
├── repair_test.c                # repair unit tests
├── erasure_code_test_optimised* # repair benchmarks on top of ISA-L
├── testRS96.c, *Codeword*.txt   # RS test vectors
├── F16_calculator.c, FFC.c, ... # finite-field calculators
├── gf_*vect_*.asm, ec_base.*    # Intel ISA-L back end (GF(2^8) arithmetic)
├── erasure_code.h, types.h, ... # ISA-L headers
└── Makefile.am                  # ISA-L build fragment
```

## Building and running

The trace-repair programs are self-contained C files (each has its own `main`)
and use a pure-C GF(2⁸) implementation, so they build with a plain C compiler —
no assembler needed:

```bash
cd erasure_code
gcc -g repair_scheme.c -lm -o repair_scheme
./repair_scheme
```

The same pattern builds the other standalone tools (`repair_test.c`,
`testRS96.c`, the finite-field calculators, etc.). Edit the `n`, `k`, and test
data near the top of each file's `main()` to target a different RS code.

The SIMD-optimised benchmark (`erasure_code_test_optimised.c`) depends on the
ISA-L assembly kernels; build it through ISA-L's own toolchain (`nasm` + the
provided `Makefile.am`) so the `gf_*vect_*.asm` routines are assembled and
linked.

## Citation

```bibtex
@inproceedings{dinh2022repairing,
  title     = {Practical Considerations in Repairing Reed-Solomon Codes},
  author    = {Dinh, Thi Xinh and Nguyen, Luu Y Nhi and Mohan, Lakshmi J.
               and Bozta\c{s}, Serdar and Luong, Tran Thi and Dau, Son Hoang},
  booktitle = {2022 IEEE International Symposium on Information Theory (ISIT)},
  pages     = {2607--2612},
  year      = {2022},
  doi       = {10.1109/ISIT50566.2022.9834830}
}
```

## Acknowledgements

GF(2⁸) arithmetic and RS encode/decode build on
[Intel ISA-L](https://github.com/intel/isa-l), used under its BSD-3-Clause
license.
