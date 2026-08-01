# level09 analysis record

## Message record

`handle_msg` creates a 192-byte stack record. It initializes the length at
record offset `0xb4` to `0x8c`, then calls `set_username` and `set_msg`:

```text
0x00000000000008cb <+11>:  lea    -0xc0(%rbp),%rax
0x00000000000008ff <+63>:  movl   $0x8c,-0xc(%rbp)
0x0000000000000910 <+80>:  callq  set_username
0x000000000000091f <+95>:  callq  set_msg
```

`set_username` copies up to index 40 into a member that begins at record offset
`0x8c`. Byte 40 therefore modifies the low byte of the length at `0xb4`:

```text
0x0000000000000a5f <+146>: mov    %cl,0x8c(%rdx,%rax,1)
0x0000000000000a6a <+157>: cmpl   $0x28,-0x4(%rbp)
```

`set_msg` reads up to 1023 bytes, loads the modified integer at `0xb4`, and
uses it as the `strncpy` count:

```text
0x0000000000000995 <+99>:  mov    $0x400,%esi
0x000000000000099d <+107>: callq  fgets@plt
0x00000000000009a9 <+119>: mov    0xb4(%rax),%eax
0x00000000000009c6 <+148>: callq  strncpy@plt
```

## Backdoor

The unreferenced `secret_backdoor` reads a command and passes it to `system`.
With ASLR disabled, its observed runtime address was `0x55555555488c`. No
binary, core dump, or generated payload file is retained here.