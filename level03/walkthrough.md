# level03

## Objective

Choose an integer that makes the binary decrypt its embedded success string and
call `system("/bin/sh")` with `level04` effective permissions.

## Control flow

`main` calls:

```c
test(password, 0x1337d00d);
```

`test` computes `delta = 0x1337d00d - password`. Its jump table routes values
from `0` through `0x15` to `decrypt(delta)`. Any other value instead uses the
unpredictable result of `rand()`.

`decrypt` XORs every byte of this embedded 16-byte value with the low byte of
its argument:

```text
51 7d 7c 75 60 73 66 67 7e 73 66 7b 7d 7c 61 33
```

It then compares the result, including its NUL terminator, with
`Congratulations!` and calls `system("/bin/sh")` only on equality.

## Key derivation

The first encoded byte is `0x51` (`Q`), while the required first byte is
`0x43` (`C`):

```text
0x51 XOR 0x43 = 0x12
```

Every remaining byte produces the same key, so the required delta is `0x12`.
It lies inside the accepted `0..0x15` range.

```text
password = 0x1337d00d - 0x12
         = 0x1337cffb
         = 322424827
```

## Exploit

Run the program and enter the computed decimal value:

```sh
./level03
```

```text
Password:322424827
```

At the resulting shell, verify the effective user and read the next password:

```sh
id
cat /home/users/level04/.pass
```

Observed identity:

```text
uid=1003(level03) gid=1003(level03) euid=1004(level04) egid=100(users)
```

The recovered password is recorded in `flag`.