---
title: "VULNCON CTF 2020 - Find the Coin writeup"
excerpt: "Writeup for the Find the Coin chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T13:25:30-03:00
categories:
  - Cybersecurity
tags:
  - OSINT
  - VULNCON CTF 2020
  - CTF
---

## Description:

> Hackers stole lot of money from Kucoin(Popular exchanger), we found a recent transaction of the value 100,000,000 DX at 26 Nov 2020 happened from the hacker's wallet can you find the tx id for me ?
>
> Flag Format: vulncon{drop_what_you_got}
>
> Author - Kick

## Solution

I found trading information about DX in https://coinmarketcap.com/currencies/dxchain-token/. There is a link called `Explorer` there that redirects you to https://etherscan.io/token/0x973e52691176d36453868D9d86572788d27041A9.

On this page, it is possible to download a CSV with the trade history for this coin. Looking at the date of November 26, it was possible to find the following line:

`"0xfdef5b6f6dece6b29695b9fd8d0cadaff944876e598fd443125e1f8c2db15160","11333292","1606384469","2020-11-26 09:54:29","0xd32dbed0609ac3169cc4dd6b781f04cee5ba9550","0xa1d8d972560c2f8144af871db508f0b0b10a3fbf","100000000"`

So the flag is:

`vulncon{0xfdef5b6f6dece6b29695b9fd8d0cadaff944876e598fd443125e1f8c2db15160}`
