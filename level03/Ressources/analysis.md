# level03 analysis record

## Input transformation

`main` passes the user integer and the constant `0x1337d00d` to `test`:

```text
0x080488c6 <+108>: mov    0x1c(%esp),%eax
0x080488ca <+112>: movl   $0x1337d00d,0x4(%esp)
0x080488d5 <+123>: call   test
```

`test` stores `expected - input`, accepts only unsigned values through `0x15`
for its jump table, and otherwise calls `rand`:

```text
0x08048753 <+12>: sub    %eax,%ecx
0x0804875c <+21>: cmpl   $0x15,-0xc(%ebp)
0x08048760 <+25>: ja     0x804884a <test+259>
0x0804884a <+259>: call   rand@plt
```

## Decryption condition

`decrypt` XORs each encoded byte with the low byte of its argument, then
compares 17 bytes with the success literal and calls `system` on equality:

```text
0x080486d2 <+114>: mov    0x8(%ebp),%eax
0x080486d5 <+117>: xor    %edx,%eax
0x080486df <+127>: mov    %dl,(%eax)
0x080486f7 <+151>: mov    $0x11,%ecx
0x08048711 <+177>: test   %eax,%eax
0x08048713 <+179>: jne    0x8048723 <decrypt+195>
0x0804871c <+188>: call   system@plt
```

The observed successful input was `322424827`, which supplies XOR key `0x12`.
No binary, core dump, or generated payload file is retained here.