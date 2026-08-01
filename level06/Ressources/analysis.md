# level06 analysis record

## Main success path

`main` reads a 32-byte login, reads an unsigned serial, and invokes `auth`.
Only an `auth` result of zero reaches `system("/bin/sh")`:

```text
0x080488e0 <+103>: lea    0x2c(%esp),%eax
0x080488e7 <+110>: call   fgets@plt
0x0804892d <+180>: call   __isoc99_scanf@plt
0x08048941 <+200>: call   auth
0x08048946 <+205>: test   %eax,%eax
0x08048948 <+207>: jne    0x8048969 <main+240>
0x0804895d <+228>: call   system@plt
```

## Auth recurrence

The disassembly establishes the length requirement, anti-debug guard, initial
serial value, and per-character update:

```text
0x08048786 <+62>:  cmpl   $0x5,-0xc(%ebp)
0x080487b5 <+109>: call   ptrace@plt
0x080487ba <+114>: cmp    $0xffffffff,%eax
0x080487f9 <+177>: xor    $0x1337,%eax
0x080487fe <+182>: add    $0x5eeded,%eax
0x08048845 <+253>: imul   $0x539,%eax,%eax
0x08048854 <+268>: add    %eax,-0x10(%ebp)
```

The final comparison accepts only an equal supplied serial. No binary, core
dump, or generated payload file is retained here.