---
title: "VULNCON CTF 2020 - Game Over writeup"
excerpt: "Writeup for the Game Over chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T15:31:30-03:00
categories:
  - Cybersecurity
tags:
  - Memory Forensic
  - VULNCON CTF 2020
  - CTF
---

## Description:

```
My friend D E V I N E R was searching for shortcut to earn money. He visited some online sites for that and registered there with his email, but unfortunately he infected his PC with some malware. He gave me the memory dump of his PC and want me to find out the installed malware and remove it. Can you find out when and which website he visited to earn money?

Chall File

Flag Format: vulncon{example.com-DD-MM-YYYY}

Author - r3curs1v3_pr0xy
```

## Solution

The `dump.raw` file is a memory dump of the system being analyzed. I used Volatility to explore the dump and solve the challenge.

First, I used `imageinfo` to get information about the system:

```terminal
$ python2 ~/tools/volatility/vol.py imageinfo -f dump.raw
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/vccolombo/cybersec/ctf/2020/vulncon/game over/dump.raw)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002bf20a0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002bf3d00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-12-12 14:05:05 UTC+0000
     Image local date and time : 2020-12-12 19:35:05 +0530
```

With the Profile, I can get the process list:

```
$ python2 ~/tools/volatility/vol.py --profile=Win7SP1x64 pslist -f dump.raw
```

The list is large, but it is possible to see that the browser used is Google Chrome. To get the browsing history, the Volatility plugin [chromehistory](https://github.com/superponible/volatility-plugins):

```
$ python2 ~/tools/volatility/vol.py --plugins=volatility-plugins/ --profile=Win7SP1x64 -f dump.raw chromehistory

Index  URL                                                                              Title                                                                            Visits Typed Last Visit Time            Hidden Favicon ID
------ -------------------------------------------------------------------------------- -------------------------------------------------------------------------------- ------ ----- -------------------------- ------ ----------
     7 https://www.google.com/search?source=hp...QCgAQGqAQdnd3Mtd2l6sAEA&sclient=psy-ab facebook - Google खोजी                                                        2     0 2020-12-12 13:46:13.497778        N/A
     1 http://google.com/                                                               Google                                                                                2     2 2020-12-12 13:46:06.035590        N/A
     4 https://www.google.com/search?source=hp...nO2MycjtAhW3zzgGHcBJDXIQ4dUDCAc&uact=5 online betting game - Google खोजी                                             3     0 2020-12-12 13:42:15.187823        N/A
     8 https://www.facebook.com/                                                        Facebook - Log In or Sign Up                                                          2     0 2020-12-12 13:46:16.862696        N/A
     2 http://www.google.com/                                                           Google                                                                                2     0 2020-12-12 13:46:06.035590        N/A
     6 https://www.gamblingsites.org/                                                   Online Gambling Sites - Best Real Money Gambling Sites 2020                           1     0 2020-12-12 13:43:07.638967        N/A
     3 https://www.google.com/                                                          Google                                                                                2     0 2020-12-12 13:46:06.035590        N/A
     4 https://www.google.com/search?source=hp...nO2MycjtAhW3zzgGHcBJDXIQ4dUDCAc&uact=5 online betting game - Google खोजी                                             3     0 2020-12-12 13:42:15.187823        N/A
     7 https://www.goov%;|eo������...�i�y�Yd�(�                                                                                       1     0 1601-01-01 00:00:00               N/A
     1 http://google.com/                                                               Google                                                                                1     1 2020-12-12 13:41:40.947451        N/A

```

The history shows the site `gamblingsites.org`, which was accessed on 2020-12-12. So the flag is:

`vulncon{gamblingsites.org-12-12-2020}`
