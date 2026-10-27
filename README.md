# Assignment 9 — Page Tables, Lazy Allocation, Page Replacement
Name: Parepally Aravind
Roll Number: 2401MC36
Course: Operating Systems
Assignment: Assignment 9 (Page Tables, Lazy Allocation, Page Replacement)

**Base:** xv6-riscv (Sv39, 3-level tables). Adaptations from x86 PDF: `walkpgdir` → recursive Sv39 walk; `rcr2()` → `r_stval()`; `T_PGFLT` → scause 13/15; `P` flag → `PTE_V`; "ignore kernel" → print only `PTE_U` leaves. Every kernel edit is commented `// A9-Q1` / `// A9-Q2`.

## Q1 — vmprint()
**Changes:** `kernel/vm.c`: `vmprint()` + recursive `vmprint_walk()`; prints VA, PA (`PTE2PA`) and P/W/U for every valid user leaf PTE; totals user pages and page-table pages (root + each non-leaf table). Wired via `defs.h`, `sysproc.c` (`sys_vmprint`), `syscall.h/.c`, `usys.pl`, `user.h`. `tests/pttest.c` dumps at start-up and after `sbrk(4*4096)` + touching the 4 new pages.
**Output:** 3 user pages initially (0x0 W=0, 0x1000, 0x3000), 5 page-table pages; afterwards 7 user pages (0x4000-0x7000 added) = exactly four new pages; page-table pages still 5.

## Q2 — Lazy allocation
**Changes:** `proc.h/.c`: `page_fault_count` in `struct proc`, zeroed in `allocproc`. `sysproc.c`: `sys_sbrk` with n>0 only grows `sz` (base's eager-mode flag disabled); n<0 still frees; `sys_getfaults()` returns the counter. `vm.c`: `uvmcopy`/`uvmunmap` `continue` instead of `panic` on unmapped PTEs. `trap.c`: scause 13/15 with `va < p->sz` -> `kalloc`+`memset(0)`+`mappages(PTE_R|PTE_W|PTE_U)` and `page_fault_count++`; otherwise print pid+address and kill only that process. `tests/faulttest.c`: sbrk(32*4096), 5 writes, delta via `getfaults()`; child touches above `sz`.
**Output:** "faults caused by the 5 writes = 5 — PASS"; child dies with `pid 4: fault at 0x26000 (sz=0x25000), killed` while the parent continues — PASS.

## Q3 — FIFO vs LRU
**Changes:** `tests/pagerepl.c` only (pure user space). FIFO via circular victim pointer; LRU via last-use timestamps; per-reference frames + HIT/FAULT; table for 3/4/5 frames with faults and hit ratios.
**Output:** string `7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1`, 3 frames -> FIFO 15 faults, LRU 12 faults (expected values). Table: 3: 15/12 (25%/40%), 4: 10/8 (50%/60%), 5: 9/7 (55%/65%) — LRU better at every size.

## Contents
- `diffs/`: vm.c.diff, trap.c.diff, proc.diff, sysproc.c.diff, syscall_wiring.diff
- `tests/`: pttest.c, faulttest.c, pagerepl.c, run_tests.sh
- `screenshots/`: q1_pttest.png, q2_faulttest.png, q3_pagerepl_frames.png, q3_pagerepl_table.png
