# level00 analysis record

Commands used during the demonstration:

```sh
file ./level00
strings -a ./level00
gdb -q ./level00
```

The `main` disassembly compares the integer read by `scanf("%d", ...)` against `0x149c` and calls `system("/bin/sh")` only on equality. This record contains no downloaded binaries or account credentials.

Relevant instructions observed in GDB:

```text
0x080484de <+74>: call   __isoc99_scanf@plt
0x080484e3 <+79>: mov    0x1c(%esp),%eax
0x080484e7 <+83>: cmp    $0x149c,%eax
0x080484ec <+88>: jne    0x804850d <main+121>
0x080484fa <+102>: movl   $0x8048649,(%esp)
0x08048501 <+109>: call   system@plt
```

`0x149c` equals `5276` in decimal. With that value, the child shell reported:

```text
uid=1000(level00) gid=1000(level00) euid=1001(level01) egid=100(users)
```

The recovered level01 password is intentionally recorded in `../flag` under
the repository-wide convention.