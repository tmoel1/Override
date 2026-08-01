# level00

## Objective

Use the setuid `level00` binary to obtain a shell with the permissions required to read the next account's `.pass` file.

## Binary observations

```text
file ./level00
setuid setgid ELF 32-bit LSB executable, Intel 80386, not stripped
```

`strings` identifies `__isoc99_scanf`, `Authenticated!`, `Invalid Password!`, and `/bin/sh`. The program therefore accepts a value from standard input and may start a shell.

## Reverse engineering

Run GDB interactively:

```sh
gdb -q ./level00
```

Then disassemble `main`:

```gdb
disassemble main
```

The relevant instructions are:

```text
call   __isoc99_scanf@plt
mov    0x1c(%esp),%eax
cmp    $0x149c,%eax
jne    <failure branch>
call   puts@plt              # "Authenticated!"
call   system@plt            # "/bin/sh"
```

`0x149c` is `5276` in decimal. No memory corruption is needed: the program accepts the hard-coded integer and calls `system("/bin/sh")`.

## Demonstration

```sh
./level00
```

At `Password:`, enter `5276`. At the shell prompt, verify the effective identity before accessing the next-level password:

```sh
id
cat /home/users/level01/.pass
```

The observed `id` output shows `euid=1001(level01)` and `egid=100(users)`, which confirms that the setuid program grants access as `level01` even though the real user remains `level00`.

The command prints the value recorded in `flag`:

```text
uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
```

Use it to authenticate with `su level01` and continue with the next binary.