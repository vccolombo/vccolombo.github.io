---
title: "VULNCON CTF 2020 - All I know was zip writeup"
excerpt: "Writeup for the All I know was zip chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T13:37:30-03:00
categories:
  - Cybersecurity
tags:
  - Misc
  - VULNCON CTF 2020
  - CTF
---

## Description:

> My friend mailed me some hex numbers and told that, It's a zip. Can you help me to covert the these numbers to zip and get the flag? note:- all letters are lowercase and underscore between words formate:- vulncon{}
>
> Author - _5h4rk_

## Solution

The file has a lot of hex numbers separated by commas and new lines. I changed it to be like this:

```
\x50\x4B\x03\x04\x14\x00\x00\x00\x08...
```

Then I used `echo -ne "\x50\x4B\x03\x04\x14\x00\x00\x00\x08..." > file.zip` to create the zip file from it. It is indeed a zip file as can be seen by the file's magic numbers. Unziping it gives a password protected PDF file.

To get the password, I used John the Ripper to bruteforce it:

```terminal
$ df2john.pl encrption.pdf > pdf.hash
$ john pdf.hash
```

The password found was "butterfly". By opening the PDF, there is some text from a different alphabet and an image from Game of Thrones.

The alphabet is actually from the dragon language in The Elder Scrolls. It translates to `vulneyondraeyonieyiseyool`

Knowing the flag format, it was possible to derive it by changing the "ey" to "c" and separating the words as requested in the description:

`vulncon{draconic_is_cool}`
