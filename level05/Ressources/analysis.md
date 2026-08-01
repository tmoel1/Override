# level05 analysis record

## Relevant control flow

The input is read into the local stack buffer at `esp + 0x28`. Only bytes from
`0x41` through `0x5a` are modified with XOR `0x20`:

```text
0x0804846e <+42>:  lea    0x28(%esp),%eax
0x08048475 <+49>:  call   fgets@plt
0x08048495 <+81>:  cmp    $0x40,%al
0x080484a7 <+99>:  cmp    $0x5a,%al
0x080484bb <+119>: xor    $0x20,%edx
```

The resulting buffer becomes the format string, then the program calls exit:

```text
0x08048500 <+188>: lea    0x28(%esp),%eax
0x08048507 <+195>: call   printf@plt
0x0804850c <+200>: movl   $0x0,(%esp)
0x08048513 <+207>: call   exit@plt
```

## Observed exploit facts

The stack probe placed the first attacker-controlled word at argument 10. The
successful staged writes redirected `exit@GOT` to `main`, then `printf@GOT` to
the stable libc `system` address. No binary, core dump, or generated payload
file is retained here.