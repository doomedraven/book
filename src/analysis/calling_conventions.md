# Calling Conventions

Rizin uses calling conventions to help in identifying function formal arguments and return types.
It is used also as a guide for basic function prototype and type propagation.

```
[0x00000000]> afc?
Usage: afc[lor]   # Calling convention
| afc [<convention>] # Set/Get calling convention for current function
| afcl[j*kl]         # List all available calling conventions
| afco <db_path>     # Open Calling Convention sdb profile from given path
| afcr[j]            # Show register usage for the current function
[0x00000000]>
```

To list all available calling conventions for current architecture using `afcl` command

```
[0x00000000]> afcl
amd64
amd64syscall
ms
reg
swift
```

The default calling convention for a particular architecture/binary is defined with
`analysis.cc` for user-mode calls and `analysis.syscc` for syscalls.

All this information is loaded via sdb under `/librz/analysis/d/cc-[arch]-[bits].sdb`

```
default.cc=amd64

ms=cc
cc.ms.name=ms
cc.ms.arg1=rcx
cc.ms.arg2=rdx
cc.ms.arg3=r8
cc.ms.arg3=r9
cc.ms.argn=stack
cc.ms.ret=rax
```

`cc.x.argi=rax` is used to set the ith argument of this calling convention to register name `rax`

`cc.x.argn=stack` means that all the arguments (or the rest of them in case there was `argi` for any `i` as
counting number) will be stored in the stack from left to right

`cc.x.argn=stack_rev` same as `cc.x.argn=stack` except for it means argument are passed right to left

## Golang Calling Conventions

Rizin automatically detects and applies Golang calling conventions for stripped and non-stripped Golang binaries during `aaa` (when `bin.lang` is detected as `go`) or when running `aalg` manually on binaries containing a valid `gopclntab` (`sym.go.*` functions).

Supported Golang calling conventions:
* `golang`:
  * On `x86-64` (Go 1.17+) and `arm-64` (Go 1.18+), this represents `ABIInternal`, which passes arguments and return values primarily through registers.
  * On `x86-32`, `golang` is stack-based (`cc.golang.argn=stack`).
* `golang_abi0` (`x86-64` only): The traditional stack-based `ABI0` calling convention used in older Go versions (prior to Go 1.17) and for Go assembly functions (`*.abi0`).

On `x86-64`, Rizin uses a prologue voting heuristic to inspect recovered function prologues and distinguish between `golang` (`ABIInternal`) and `golang_abi0` (`ABI0`), even when the build header is stripped. On `arm-64` and `x86-32`, `golang` is selected directly.
