# System Call Tracing

## Purpose

The tracing feature helps debug user programs by reporting selected system
calls and their return values. Each process has its own trace mask, so enabling
tracing does not turn it on globally or affect unrelated processes.

## Design

The mask is an integer whose bit at position *n* selects system call number
*n*. For example, `read` has system call number 5, so `1 << 5` (32) traces
only `read`. A set bit produces one line after that system call handler
returns:

```text
syscall read -> 1023
```

Tracing state belongs to the process:

- `trace(mask)` replaces the calling process's current mask.
- `fork` copies the mask to the child, allowing a traced process to continue
  tracing the programs it starts.
- `exec` leaves the process and its mask in place, so the newly loaded program
  remains traced.
- Other processes retain their own masks.

## Implementation

| Component | Responsibility |
| --- | --- |
| `kernel/syscall.h` | Assigns `SYS_trace` its system call number (23). |
| `user/user.h`, `user/usys.pl` | Declare `trace(int)` and generate its user-to-kernel system call stub. |
| `kernel/proc.h` | Stores the per-process `trace_mask`. |
| `kernel/proc.c` | Resets the mask when a process slot is freed and copies it during `kfork`. |
| `kernel/sysproc.c` | Implements `sys_trace`, reading the integer mask with `argint` and storing it on the current process. |
| `kernel/syscall.c` | Registers `sys_trace`, maps syscall numbers to printable names, and reports selected calls after the handler returns. |
| `user/trace.c` | Parses the mask, enables tracing, and runs the requested program with `exec`. |
| `Makefile` | Adds `_trace` to the user programs included in the filesystem image. |

The dispatcher checks the syscall number against the process mask after running
the handler, when its return value is available in the trap frame. The trace
syscall itself can therefore be reported if its own bit is selected.

## Usage

Run a program with only its `read` calls traced:

```sh
$ trace 32 grep hello README
syscall read -> 1023
...
```

To trace all currently assigned system call numbers below 31:

```sh
$ trace 2147483647 grep hello README
syscall trace -> 0
syscall exec -> 3
syscall open -> 3
...
```

Running the program directly does not enable tracing:

```sh
$ grep hello README
```

The launcher requires a mask and a command:

```text
trace mask command [args...]
```

If the command cannot be executed, the launcher reports the failure and exits
with a nonzero status.

## Scope and limitations

Tracing reports successful dispatches for known system calls; invalid or
unregistered syscall numbers continue to use the kernel's existing
`unknown sys call` diagnostic. The mask is an `int`, and the usage examples
select syscall numbers that fit its available positive bits.
