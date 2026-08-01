# level07 analysis record

## Command dispatch

The three command strings are `store`, `read`, and `quit`. `main` dispatches
them to `store_number`, `read_number`, or normal return respectively.

## Unchecked store

`store_number` calculates `index % 3` and applies the high-byte value filter,
but performs no bound check before multiplying the index by four and writing:

```text
0x0804866e <+62>:  mov    -0xc(%ebp),%ecx
0x08048688 <+88>:  je     0x8048697 <store_number+103>
0x0804868a <+90>:  mov    -0x10(%ebp),%eax
0x08048690 <+96>:  cmp    $0xb7,%eax
0x080486c2 <+146>: mov    -0xc(%ebp),%eax
0x080486c5 <+149>: shl    $0x2,%eax
0x080486c8 <+152>: add    0x8(%ebp),%eax
0x080486ce <+158>: mov    %edx,(%eax)
```

The measured saved-return index was 114. The successful wrapped index was
`1073741938`, followed by regular indices 115 and 116 for the ret2libc frame.
No binary, core dump, or generated payload file is retained here.