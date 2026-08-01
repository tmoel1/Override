# level04 analysis record

## Process structure

The binary forks before reading input. The child marks itself traceable, then
passes a 128-byte stack buffer directly to `gets`:

```text
0x08048722 <+90>:  call   prctl@plt
0x08048746 <+126>: call   ptrace@plt
0x08048757 <+143>: lea    0x20(%esp),%eax
0x0804875e <+150>: call   gets@plt
```

The parent waits for child stops in a loop, then uses `PTRACE_PEEKUSER` (request
`3`) at offset `0x2c` to inspect the i386 syscall number. It kills the child
when that value is `0xb` (`execve`):

```text
0x080487d5 <+269>: movl   $0x3,(%esp)
0x080487dc <+276>: call   ptrace@plt
0x080487e8 <+288>: cmpl   $0xb,0xa8(%esp)
0x08048814 <+332>: call   kill@plt
```

## Runtime measurements

At the instruction immediately before `gets`, GDB measured a saved-return
offset of 156 bytes. Under `set exec-wrapper env -i`, the buffer began at
`0xffffddc0`. The final `env -i` exploit using that address succeeded.

The shellcode forks, exits in the traced branch, and calls `execve("/bin/sh")`
only in the untraced branch. No binary, core dump, or generated payload file is
retained here.