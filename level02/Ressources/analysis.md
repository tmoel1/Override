# level02 analysis record

## Relevant disassembly

The program reads 41 bytes from the next-level password file into the local
buffer at `rbp - 0xa0`:

```text
0x00000000004008e6 <+210>: lea    -0xa0(%rbp),%rax
0x00000000004008f4 <+224>: mov    $0x29,%edx
0x0000000000400901 <+237>: callq  fread@plt
```

The username and supplied password are separate 100-byte inputs:

```text
0x00000000004009cd <+441>: lea    -0x70(%rbp),%rax
0x00000000004009d1 <+445>: mov    $0x64,%esi
0x00000000004009d9 <+453>: callq  fgets@plt
0x0000000000400a10 <+508>: lea    -0x110(%rbp),%rax
0x0000000000400a17 <+515>: mov    $0x64,%esi
0x0000000000400a1f <+523>: callq  fgets@plt
```

On a failed comparison, the username buffer becomes the format argument:

```text
0x0000000000400a96 <+642>: lea    -0x70(%rbp),%rax
0x0000000000400a9a <+646>: mov    %rax,%rdi
0x0000000000400aa2 <+654>: callq  printf@plt
```

On a successful 41-byte `strncmp`, the binary greets the user and invokes its
own `system("/bin/sh")` success path. No binary, core dump, or generated
payload file is retained here.