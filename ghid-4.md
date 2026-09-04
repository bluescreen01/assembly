# Subiectul 3 — Build errors & binary inspection

x86-64 NASM/C project on the PCLP2 VM. Answers in statement order. All fixes are
verified end-to-end on the actual files.

---

## a) Make the compiled stack executable, and prove it

**Root cause.** Modern GCC emits a `.note.GNU-stack` note that tells the linker to
mark the `PT_GNU_STACK` segment non-executable (`RW`). That default flows through to
`a/main`, so its stack is not executable.

**Minimal fix** — the `main.c` source is left untouched; only the link flags change
(the buffer that makes an executable stack meaningful, `int buffer[128]`, already
exists in the source):

```diff
- LDFLAGS = -g -no-pie
+ LDFLAGS = -g -no-pie -z execstack
```

`-z execstack` overrides the note and flips the `GNU_STACK` segment to `RWE`. This
respects the "make the necessary modifications" constraint: the change lives entirely
in the link line, and the C file is not modified.

Full corrected `a/Makefile`:

```make
CC = gcc
CFLAGS = -g -fno-PIC
LDFLAGS = -g -no-pie -z execstack   # mark the program's stack RWE
BIN = main

all: $(BIN)

$(BIN).o: $(BIN).c

clean:
	rm -f $(BIN) *.o
```

**VERIFICATION:**

- Static: `make -C a && readelf -l a/main | grep -A1 GNU_STACK` → the flags column
  shows **`RWE`** (was `RW` before the fix).
- Runtime: a tiny `est.c` test copies a `0xC3` (`ret`) byte onto a stack buffer and
  calls it. Built with `-z execstack` it returns cleanly (`exit=0`); built without it
  the CPU enforces W^X and it dies with **SIGSEGV (exit 139)** — proving the stack is
  genuinely executable only with the fix.

  ```c
  /* Copies a `ret` (0xC3) into a stack buffer and calls it. Runs only if the
   * stack is executable; otherwise SIGSEGV. Runtime proof, complementary to the
   * readelf GNU_STACK flag. */
  int main(void){ unsigned char code[16]={0xC3}; ((void(*)(void))code)(); return 0; }
  ```

  ```
  gcc -g -no-pie -z execstack -o est_x est.c && ./est_x   # exit=0
  gcc -g -no-pie             -o est_n est.c && ./est_n   # Segmentation fault, exit=139
  ```

---

## b) Call the function from `b/b.o` to print "You're on a streak."

**Inspection.** `nm b.o` exposes the symbol `T f` plus a private `counter.0` in
`.bss` and an undefined `puts`. The disassembly (`objdump -dr b.o`) decodes to:

```c
static int counter;              // .bss, starts at 0
void f(int arg) {
    counter++;                   // ++ on every call
    if (counter / arg == 8)      // unsigned div counter/arg
        puts("You're on a streak.");   // R_X86_64_32 -> .rodata string
}
```

So the message fires on the call where `counter / arg == 8`. With `arg == 1` that is
the 8th call, and it prints exactly once (`counter/1 == 8` only when `counter == 8`).
`arg` must be non-zero or the `div` faults (SIGFPE). The Makefile already links
`main.o` with `b.o`, so `main.c` only needs the `extern` declaration and the calls:

```c
extern void f(int arg);

int main(void)
{
    for (int i = 0; i < 12; i++)   // reaches the 8th call; prints once
        f(1);
    return 0;
}
```

**VERIFICATION:** `make -C b && ./b/b` prints exactly:

```
You're on a streak.
```

---

## c) Trigger "You hacked me!" in `c/c`

**Inspection.** `main` (from `objdump -d -M intel c`, strings from
`objdump -s -j .rodata c`) does:

1. `scanf("%d", &index)` — reads an integer.
2. `getchar()` — eats the newline.
3. `row = base + index*32; fgets(row, 32, stdin)` — `base = [rbp-0x70]`, rows are 32
   bytes each.
4. `strcmp(base + 0x40, "write beyond the map\n")` — the compared buffer is at offset
   `0x40 = 64 = 2*32`, i.e. **row index 2**. The target string (`.rodata` at
   `0x402007`) includes the trailing `\n` that `fgets` keeps.
5. Equal → `puts("You hacked me!")` (`0x40201d`); else the "empty" message
   (`0x40202c`).

So input **index `2`** to make `fgets` land in the compared row, then type the exact
target line. The section-scoped dump `objdump -s -j .rodata c` is the correct tool
here (over `strings`) because it ties each byte string to its address, letting you
match `0x402007` to the `strcmp` operand.

Exact command:

```sh
printf '2\nwrite beyond the map\n' | ./c/c
```

**VERIFICATION:** the command above prints exactly:

```
You hacked me!
```

Cross-check the addresses with `objdump -s -j .rodata c` (`0x402007` =
`"write beyond the map\n"`, `0x40201d` = `"You hacked me!"`) or, equivalently,
`readelf -x .rodata c`.
