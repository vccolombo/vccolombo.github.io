---
title: "VULNCON CTF 2020 - Pcaped writeup"
excerpt: "Writeup for the Pcaped chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T13:42:30-03:00
categories:
  - Cybersecurity
tags:
  - Misc
  - VULNCON CTF 2020
  - CTF
---

## Description:

> Too Easy if you know how it works :)
>
> Author - D3V1LaL

## Solution

The file is a network dump. I used Wireshark to analyze it. There is a lot of TCP dumps in it. Each TCP was a one-byte character, that I copied by hand (I couldn't find an automatic way of doing it). The result is:

`VGhlIGZsYWcgaXMtPiBCMXRfYnlfQjF0X3YxYV9uYwo=`

This looks like a base64 encoded string. By decoding it we have:

`The flag is-> B1t_by_B1t_v1a_nc`

So the flag is:

`vulcon{B1t_by_B1t_v1a_nc}`
