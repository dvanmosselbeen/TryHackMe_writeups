# TryHackMe Writeups

This repository contains a few of my writeups I made for the famous and addictive TryHackMe CTF (Capture The Flag) challenges.

Check out the TryHackMe website for your subscription!

https://tryhackme.com/

[![TryHackMe Profile](itchy.png)](https://tryhackme.com/p/itchy)

## TryHackMe writeups

- [THM - Lazy Admin](laszy-admin) - Easy linux machine to practice your skills
- [THM - Bounty Hunter (Cowboy Hacker)](bounty-hunter/README.md) - You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!
- [THM - Startup](startup/README.md) - Abuse traditional vulnerabilities via untraditional means.
- [THM - Blue](blue/README.md) - Hacking: Enumerating the Eternal blue vulnerability [CVE-2017-0143](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143) with `nmap` and making (bad) use of it with `metasploit`. Finally, dumping and cracking the password hash of a user.
- [THM - Agent Sudo](agent-sudo/README.md) - Hacking: Enumerating a webserver, faking user-agent of web browser, steganography, brute-forcing ftp server, privilege escalation.
- [THM - Wireshark 101](wireshark-101/README.md) - The Wireshark 101 Writeup
- [THM - Basic Pentesting](basic_pentesting/README.md) - Various penetration / cracking. Like brute forcing, hash cracking, service enumeration, Linux enumeration.
- [THM - hydra](hydra/README.md) - Hydra is the ultimate tool for online password attacks.
- [THM - Brooklyn Nine Nine](brooklyn-nine-nine/README.md) - Hack that box!

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
