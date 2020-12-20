---
title: "VULNCON CTF 2020 - can_you_c_the_password? writeup"
excerpt: "Writeup for the can_you_c_the_password? chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T14:45:30-03:00
categories:
  - Cybersecurity
tags:
  - Cryptography
  - VULNCON CTF 2020
  - CTF
---

## Description:

> Decrypt the password and submit it as the flag!
>
> Chall file
>
> Author - maniac

## Solution

This is a chall about the Active Directory's Group Policy Preferences. It is a Windows feature that allows sysadmins to configure machines remotely. [It is vulnerable because the password can be easily recovered by an attacker](https://support.microsoft.com/en-in/help/2962486/ms14-025-vulnerability-in-group-policy-preferences-could-allow-elevati).

The file `Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml` contains a field `cpassword="HlQWFdlPXQTU7n8W9VbsVTP245DcAJAUQeAZZfkJE/Q8ZlWgwj7CqKl6YiPvKbQFO7PWS7rSwbVtSSZUhJSj5YzjbkKtyXR5fP9VQDEieMU"`, which is the crypted password that we want.

I used [gp3finder](https://bitbucket.org/grimhacker/gpppfinder) to decrypt it, which yields:

`vulncon{s3cur1ty_h4s_3volv3d_s0__much}`
