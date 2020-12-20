---
title: "VULNCON CTF 2020 - Compromise writeup"
excerpt: "Writeup for the Compromise chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T14:31:30-03:00
categories:
  - Cybersecurity
tags:
  - Forensic
  - VULNCON CTF 2020
  - CTF
---

## Description:

```
What account was the username and password of the compromised user ?

NOTE: Use file given in 1st Chall Attack

Flag Format: vulncon{username_password}

Author - White_Wolf
```

## Solution

I used Hydra to brute-force the SSH password:

`hydra -l karma -P rockyou.txt 127.0.0.1 -t 64 -s 2222 -V ssh`

The Attack chall gave the information about the targeted user, and I used the rockyou wordlist to find the password, which is `godisgood`:

`vulncon{karma_godisgood}`
