# TryHackMe Writeups

![alt text](TryHackMe-logo.png "Writeup Image")

This repository contains a few of my writeups I made for the famous and addictive TryHackMe CTF (Capture The Flag) challenges.

Check out the TryHackMe website for your subscription!

Find more information on the TryHackMe website: <https://tryhackme.com>

Here's a link to my profile on TryHackMe:

[![TryHackMe Profile](itchy.png)](https://tryhackme.com/p/itchy)

Note: you can also look at these documents here through the `GitHub Pages`, <https://dvanmosselbeen.github.io/TryHackMe_writeups/>. Which looks slightly better. But no navigation bar.

## TryHackMe writeups

| Room Name | Description | Tools Used |
|---|---|---|
| [ColddBox: Easy](colddboxeasy/README.md) | An easy level machine with multiple ways to escalate privileges. |   |
| [Mr Robot](mrrobot/README.md) | Based on the Mr. Robot show, can you root this box? | `nmap`, `gobuster`, `nikto`, `Burp Suite`, `hydra` |
| [Chocolate Factory](chocolatefactory/README.md) | A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory! | `nmap`, `exif`, `exiftool`, `strings`, `steghide` |
| [Lazy Admin](laszy-admin) | Easy linux machine to practice your skills. | `nmap`, `gobuster`, `webshell`, `nc` |
| [Bounty Hunter (Cowboy Hacker)](bounty-hunter/README.md) | You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker! | `nmap`, `hydra` |
| [Startup](startup/README.md) | Abuse traditional vulnerabilities via untraditional means. | `nmap`, `gobuster`, `ftp`, `nc`, `curl`, `python`, `wireshark`, `strings`, `pspy`, `linpeas` |
| [Blue](blue/README.md) | Hacking: Enumerating the Eternal blue vulnerability [CVE-2017-0143](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143) with `nmap` and making (bad) use of it with `metasploit`. Finally, dumping and cracking the password hash of a user. | `nmap`, `metasploit`, `john` |
| [Agent Sudo](agent-sudo/README.md) | Hacking: Enumerating a webserver, faking user-agent of web browser, steganography, brute-forcing ftp server, privilege escalation. | `nmap`, `gobuster`, `Burp Suite`, `FoxyProxy`, `User-Agent Switcher and Manager`, `hydra`, `steghide`, `binwalk``binwalk`, `john`, `zip2john` |
| [Wireshark 101](wireshark-101/README.md) | The Wireshark 101 Writeup. | `wireshark` |
| [Basic Pentesting](basic_pentesting/README.md) | Various penetration / cracking. Like brute forcing, hash cracking, service enumeration, Linux enumeration.  | `nmap`, `gobuster`, `enum4linux`, `hydra`, `linpeas`, `john` |
| [Hydra](hydra/README.md) | Hydra is the ultimate tool for online password attacks. | `hydra` |
| [Brooklyn Nine Nine](brooklyn-nine-nine/README.md) | Hack that box! | `nmap`, `ftp`, `hydra`, `linpeas` |

## Extra Notes

### My Security Cheat Sheet Wikeypaydia 

I have a few notes, not yet very well organised, but you can take a look at it if you want. Could be helpful.

- <https://github.com/dvanmosselbeen/security-cheat-sheet/wiki>

### Internet access not working with the TryHackMe VPN?

Information difficult to find without a link on the website :-P

Source: <https://docs.tryhackme.com/docs/openvpn/troubleshooting/openvpn-troubleshooting/#external-access-not-working>

```commandline                             
$ nmcli connection   # Note the name of the VPN connection here
$ nmcli connection edit <CONNECTION_NNAME>
> set ipv4.never-default true
> set ipv6.never-default true
> save
> quit
```

These changes are persistent.

### TODO

#### Make gobuster go even slower !

Actually make it 5x time faster or more if you don't mind that we can use the `-t 50` parameter on `gobuster`.

````commandline
$ gobuster help dir | grep -e "-t"
...
  -t, --threads int       Number of concurrent threads (default 10)
````

Merge the wordlist files:

Found this very interesting to do, for room https://tryhackme.com/room/wgelctf for example. As i mainly use the `dirbuster` wordlist, did not found the required files in first instance.

- `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
- `/usr/share/wordlists/dirb/common.txt`

As for example the common.txt would find out a folder `.ssh` which is extreme important. However, common.txt is also too common and does not find a lot of things.

Quick and dirty fix for this:

````commandline
cat /usr/share/wordlists/dirb/common.txt >> /tmp/gobusted.txt
cat  /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt >> /tmp/gobusted.txt
# Then removed dupplicates, sort it and go home. Bad idea to sort, it feels slower at the start
#uniq /tmp/gobusted.txt | sort /tmp/gobusted.txt > ~/tools/gobusted2.txt

# See man uniq or https://www.howtoforge.com/linux-uniq-command/
# This is good, but no, still not working as expected!!!!!!!!!
uniq /tmp/gobusted.txt ~/tools/gobusted.txt
````