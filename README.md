# VNUHCM-US
# FACULTY OF INFORMATION TECHNOLOGY
# OPERATING SYSTEM - CLASS CQ2024/3 - GROUP 12

## MEMBERS

- Thiều Quới Gia Bảo — 24120025
- Nguyễn Phan Khánh Đăng — 24120032
- Nguyễn Văn Hạ — 24120045

## TASK

Add a **system call tracing feature** to xv6 for debugging later labs.

- Implement a new `trace` system call that takes an integer `mask`.
- Each bit in `mask` specifies a system call to trace.
  - For example, `trace(1 << SYS_read)` enables tracing for `read`.
- When a traced system call is about to return, print:
  - The system call name
  - Its return value
- Tracing should only affect the process that enabled it, not other processes.
- Implement a user-level `trace` program (`user/trace.c`) that:
  1. Enables tracing using the provided mask.
  2. Executes another program with tracing enabled.
- The expected output should show each traced system call and its return value.