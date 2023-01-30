# RISC-V C API Specification

This file contains the RISC-V additions to the standard C APIs.  It
merges together the compiler command-line API, the preprocessor API, and
the C language extensions (function attributes and intrinsic functions).

## Copyright and license information

 &copy; 2018 Palmer Dabbelt <palmer@sifive.com>

 &copy; 2018 SiFive, Inc

It is licensed under the Creative Commons Attribution 4.0 International
License (CC-BY 4.0). The full license text is available at
https://creativecommons.org/licenses/by/4.0/.

## Compiler Command-Line Arguments

* `-mabi=ABI`
* `-march=ISA`
* `-mtune=MICRO_ARCHITECTURE`
* `-mplt` `-mno-plt`
* `-mcmodel=CODE_MODEL`
* `-mstrict-align` `-mno-strict-align`
* `-mfdiv` `-mno-fdiv`
* `-mdiv` `-mno-div`
* `-mpreferred-stack-boundary=N`
* `-msmall-data-limit=N`
* `-mexplicit-relocs` `-mno-explicit-relocs`
* `-mrelax` `-mno-relax`
* `-msave-restore` `-mno-save-restore`
* `-mbranch-cost=N`

## Preprocessor Definitions

| Name                | Value | When defined                  |
| ------------------- | ----- | ----------------------------- |
| __riscv             | 1     | Always defined.               |
| __riscv_xlen        | <ul><li>32 for rv32</li><li>64 for rv64</li><li>128 for rv128</ul> | Always defined.             |
| __riscv_flen        | <ul><li>32 if the F extension is available **or**</li><li>64 if `D` extension available **or**</li><li>128 if `Q` extension available</li></ul> | `F` extension is available. |
| __riscv_32e         | 1     | `E` extension is available.   |
| __riscv_vector      | 1     | Implies that any of the vector extensions (`v` or `zve*`) is available |
| __riscv_v_min_vlen    | <N> (see [__riscv_v_min_vlen](#__riscv_v_min_vlen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen     | <N> (see [__riscv_v_elen](#__riscv_v_elen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen_fp  | <N> (see [__riscv_v_elen_fp](#__riscv_v_elen_fp)) | The `V` extension or one of the `Zve*` extensions is available. |

### __riscv_v_min_vlen

The `__riscv_v_min_vlen` macro expands to the minimal VLEN, in bits, mandated
by the available vector extension, if any.

The value of `__riscv_v_min_vlen` is defined by the following rules:
- 128, if the `V` extension is present;
- 32, if one of the `Zve32{x,f}` extensions is present;
- 64, if one of the `Zve64{x,f,d}` extensions is present;
- `N`, if one of the `Zvl<N>b` extensions, `N` in `{32,64,128,256,512,1024}`,
is present.

If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_min_vlen` is undefined.

Examples:

- `__riscv_v_min_vlen` is 128 for `rv64gcv`
- `__riscv_v_min_vlen` is 512 for `rv32gcv_zvl512b`
- `__riscv_v_min_vlen` is 256 for `rv32gcv_zvl32b_zvl256b`
- `__riscv_v_min_vlen` is 128 for `rv64gcv_zvl32b`

### __riscv_v_elen

The `__riscv_v_elen` macro expands to the supported element length, in bits,
of any non-floating-point vector operand of any vector instruction in the
available vector extension, if any. (Stricter upper bounds may apply to
particular operands of particular instructions.)


The value of `__riscv_v_elen` is defined by the following rules:
- 64, if the `V` extension or one of the `Zve64{x,f,d}` extensions is present; and
- 32, if one of the `Zve32{x,f}` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen` is undefined.

### __riscv_v_elen_fp

The `__riscv_v_elen_fp` macro expands to the supported element length, in bits,
of any floating-point vector operand of any vector instruction in the available
vector extension, if any. (Stricter upper bounds may apply to particular
operands of particular instructions.)

The value of `__riscv_v_elen_fp` is defined by the following rules:
- 64, if one of the `V` or `Zve64d` extensions is present;
- 32, if one of the `Zve{32,64}f` extensions is present; and
- 0, if one of the `Zve{32,64}x` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen_fp` is undefined.


### Architecture Extension Test Macro

Architecture extension test macro is a new set of test macro to checking the
availability and version for certain extension, however not all compilers are
supported, so you should check `__riscv_arch_test` to make sure this compiler
is supporting those preprocessor definitions.

The value of architecture extension test macro are defined as its version,
which is compute by following formula:

```
<MAJOR_VERSION> * 1,000,000 + <MINOR_VERSION> * 1,000 + <REVISION_VERSION>
```

For example:
- F-extension v2.2 will define `__riscv_f` as `2002000`.

| Name                    | Value        | When defined                  |
| ----------------------- | ------------ | ----------------------------- |
| __riscv_arch_test       | 1            | Defined if compiler support new architecture extension test macro. |
| __riscv_i               | Arch Version | `I` extension is available.   |
| __riscv_e               | Arch Version | `E` extension is available.   |
| __riscv_m               | Arch Version | `M` extension is available.   |
| __riscv_a               | Arch Version | `A` extension is available.   |
| __riscv_f               | Arch Version | `F` extension is available.   |
| __riscv_d               | Arch Version | `D` extension is available.   |
| __riscv_c               | Arch Version | `C` extension is available.   |
| __riscv_p               | Arch Version | `P` extension is available.   |
| __riscv_v               | Arch Version | `V` extension is available.   |
| __riscv_zba             | Arch Version | `Zba` extension is available. |
| __riscv_zbb             | Arch Version | `Zbb` extension is available. |
| __riscv_zbc             | Arch Version | `Zbc` extension is available. |
| __riscv_zbs             | Arch Version | `Zbs` extension is available. |
| __riscv_zfh             | Arch Version | `Zfh` extension is available. |

### ABI Related Preprocessor Definitions

| Name                     | Value | When defined                  |
| ------------------------ | ----- | ----------------------------- |
| __riscv_abi_rve          | 1     | Defined if using `ilp32e` ABI |
| __riscv_float_abi_soft   | 1     | Defined if using `ilp32`, `ilp32e` or `lp64` ABI. |
| __riscv_float_abi_single | 1     | Defined if using `ilp32f` or `lp64f` ABI. |
| __riscv_float_abi_double | 1     | Defined if using `ilp32d` or `lp64d` ABI. |
| __riscv_float_abi_quad   | 1     | Defined if using `ilp32q` or `lp64q` ABI. |

### Code Model Related Preprocessor Definitions

| Name                  | Value    | When defined                          |
| --------------------- | -------- | ------------------------------------- |
| __riscv_cmodel_medlow | 1        | Defined if using `medlow` code model. |
| __riscv_cmodel_medany | 1        | Defined if using `medany` code model. |

### Deprecated Preprocessor Definitions

| Name                  | Value    | When defined                          | Alternative |
| --------------------- | -------- | ------------------------------------- | ----------- |
| __riscv_cmodel_pic    | 1        | GCC defines this when compiling with `-fPIC`, `-fpic`, `-fPIE` or `-fpie`. | `__PIC__` or `__PIE__` |
| __riscv_mul           | 1        | `M` extension is available.   | `__riscv_m` |
| __riscv_div           | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_muldiv        | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_atomic        | 1        | `A` extension is available.   | `__riscv_a` |
| __riscv_fdiv          | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_fsqrt         | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_compressed    | 1        | `C` extension is available.   | `__riscv_c` |

*[1] Not all compilers provide `-mno-div` and `-mno-fdiv` option.

## Function Attributes

* `__attribute__((naked))`
* `__attribute__((interrupt))`
* `__attribute__((interrupt("user")))`
* `__attribute__((interrupt("supervisor")))`
* `__attribute__((interrupt("machine")))`

## Intrinsic Functions

Intrinsic functions (or intrinsics or built-ins) are expanded into instruction sequences by compilers.
They typically provide access to functionality that is otherwise not synthesizable by compilers.
Some intrinsics expand to different code sequences depending on the available instructions from the enabled ISA extensions.

Compilers typically come with their own architecture-independent intrinsics (e.g. synchronization primitives, byte-swap, etc.).
The RISC-V compiler backend can define additional target-specific intrinsics.
Providing functionality via architecture-independent intrinsics is the preferred method, as it improves code portability.

Some intrinsics are only available if a particular header file is included.
RISC-V header files that enable intrinsics require the prefix `riscv_` (e.g. `riscv_vector.h` or `riscv_crypto.h`).

RISC-V specific intrinsics use the common prefix "__riscv_" to avoid namespace collisions.

The intrinsic name describes the functional behaviour of the function.
In case the functionality can be expressed with a single instruction, the instruction's name (any '.' replaced by '_') is the preferred choice.
Note, that intrinsics that are restricted to RISC-V vendor extensions need to include the vendor prefix (as documented in the RISC-V toolchain conventions).

If intrinsics are available for multiple data types, then function overloading is preferred over multiple type-specific functions.
If an intrinsic function is has parameters or return values that reference registers with XLEN bits, then the data type `long` should be used.
In case a function is only available for one data type and this type cannot be derived from the function's name, then the type should be appended to the function name, delimited by a '_' character.
Typical type postfixes are "32" (32-bit), "i32" (signed 32-bit), "i8m4" (vector register group consisting of 4 signed 8-bit vector registers).

RISC-V intrinsics follow the following naming rule:

```
INTRINSIC ::= PREFIX NAME [ '_' TYPE ]
PREFIX ::= "__riscv_"
NAME ::= Name of the intrinsic function.
TYPE ::= Optional type postfix.
```

RISC-V intrinsics examples:

```
type __riscv_orc_b (type rs); // orc.b rd, rs

long __riscv_clmul (long a, long b); // clmul rd, rs1, rs2

#include <riscv_vector.h> // make RISC-V vector intrinsics available
vint8m1_t __riscv_vadd_vv_i8m1(vint8m1_t vs2, vint8m1_t vs1, size_t vl); // vadd.vv vd, vs2, vs1
```
