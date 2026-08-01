# level08 analysis record

## Relevant calls

The program opens a relative log path, logs the supplied name, opens the
argument as a source, and creates a separately constructed backup path:

```text
0x0000000000400a4a <+90>:  fopen("./backups/.log", "w")
0x0000000000400aae <+190>: callq  log_wrapper
0x0000000000400acc <+220>: callq  fopen@plt
0x0000000000400b7d <+397>: callq  strncat@plt
0x0000000000400b98 <+424>: callq  open@plt
```

The copy loop reads one byte from the attacker-selected source and writes it to
the constructed destination:

```text
0x0000000000400bf5 <+517>: callq  fgetc@plt
0x0000000000400be6 <+502>: callq  write@plt
```

The successful workspace contained a symlink named `source` to level09's
password file and a real mode-777 `backups` directory. No binary, copied flag,
or generated file is retained here.