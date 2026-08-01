# level06

## Objective

Derive a valid serial for a chosen login, trigger the binary's normal
`system("/bin/sh")` branch, and read `/home/users/level07/.pass`.

## Authentication algorithm

`auth` removes the trailing newline and requires a login length greater than
five. It rejects execution while GDB is attached with `ptrace(PTRACE_TRACEME)`.
Run the target normally after reverse engineering.

For a valid login, it initializes:

```text
serial = (login[3] XOR 0x1337) + 0x5eeded
```

For every character in the login it updates:

```text
serial += (login[index] XOR serial) % 0x539
```

All login bytes must be greater than `0x1f`. The entered serial is accepted
only when it equals the final calculated value.

## Derivation for `abcdef`

`abcdef` is six printable bytes, so it passes the length and character checks.
The initial value and each update are:

```text
initial: 6226240
a:       6227377
b:       6228379
c:       6229105
d:       6230372
e:       6231562
f:       6232802
```

The accepted serial is therefore `6232802`.

## Exploit

Run the binary outside GDB:

```sh
./level06
```

Provide:

```text
-> Enter Login: abcdef
-> Enter Serial: 6232802
```

The program prints `Authenticated!` and calls `system("/bin/sh")`. Verify the
effective identity and read the next password:

```sh
id
cat /home/users/level07/.pass
```

Observed identity:

```text
uid=1006(level06) gid=1006(level06) euid=1007(level07) egid=100(users)
```

The recovered password is recorded in `flag`.