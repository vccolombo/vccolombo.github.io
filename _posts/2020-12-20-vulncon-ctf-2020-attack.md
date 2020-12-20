---
title: "VULNCON CTF 2020 - Attack writeup"
excerpt: "Writeup for the Attack chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T14:22:30-03:00
categories:
  - Cybersecurity
tags:
  - Forensic
  - VULNCON CTF 2020
  - CTF
---

## Description:

```
There's an attack that happened on one of our client Mr. Innocent Karma and his system has been compromised. We have provided the file, analyze it, and provide answers to our questions for further investigation.

Important Note: This is a series of 5 challenges each containing 200 points (dynamic) and other challenges of this series are hidden behind each other, next part will be visible when you will solve the first challenge. if you have any doubts feel free to DM the author.

What was the IP of the attacker and what attack happened on that machine?

Chall file

credentials : user:root, pass:i\/WH"VJvY5_M55qfe9<

Flag Format: vulncon{ipaddress_attackname}

Author - White_Wolf
```

## Solution

I really liked this chall, because when I saw the description I already knew what to look for.

Short story time: a few months ago I saw that the disk of the server that hosts [my newsletter](http://freegamesnewsletter.tech/) was getting full really fast. The culprits were the log files in the system, which were reaching almost 10GB already. What was happening was that my server was being targeted by multiple attackers trying to brute-force the SSH password. My password was secure against this type of attack, so the server was not compromised, but I made changes in the SSH configurations to stop the attacks.

What Linux does is log every and each login attempt (both sucessfull and unsucessfull) in `/var/log`. More specifically in the `/var/log/auth.log` there is a list of all failed logins.

`grep "Failed password" /var/log/auth.log`

This chall's machine has multiple failed login attempts from the IP `192.168.30.1`. So the flag is:

`vulncon{192.168.30.1_bruteforce}`
