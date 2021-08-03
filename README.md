# TryHackMe Writeups

This repository contains a few of my writeups I made for the famous and addictive TryHackMe CTF (Capture The Flag) challenges.

Check out the TryHackMe website for your subscription!

https://tryhackme.com/

[![TryHackMe Profile](itchy.png)](https://tryhackme.com/p/itchy)

## TryHackMe writeups

* [THM - Agent Sudo](agent-sudo/README.md) - Hacking: Enumerating a webserver, faking user-agent of web browser, steganography, brute-forcing ftp server, privilege escalation.
* [THM - Wireshark 101](wireshark-101/README.md) - The Wireshark 101 Writeup
* [THM - Basic Pentesting](basic_pentesting/README.md) - Various penetration / cracking. Like brute forcing, hash cracking, service enumeration, Linux enumeration.
* [THM - hydra](hydra/README.md) - Hydra is the ultimate tool for online password attacks.
* [THM - Brooklyn Nine Nine](brooklyn-nine-nine/README.md) - Hack that box!

## Extra Notes

### Internet access not working with the TryHackMe VPN?

Information difficult to find without a link on the website :-P

Source: https://docs.tryhackme.com/docs/openvpn/troubleshooting/openvpn-troubleshooting/#external-access-not-working

```                                  
  $ nmcli connection   # Note the name of the VPN connection here
  $ nmcli connection edit <CONNECTION_NNAME>
  > set ipv4.never-default true
  > set ipv6.never-default true
  > save
  > quit
```

These changes are persistent.
