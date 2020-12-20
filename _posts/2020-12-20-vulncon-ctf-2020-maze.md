---
title: "VULNCON CTF 2020 - Maze writeup"
excerpt: "Writeup for the Maze chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T13:19:30-03:00
categories:
  - Cybersecurity
tags:
  - Web
  - VULNCON CTF 2020
  - CTF
---

## Description:

> So close yet so far away... Can you help John to get out of the maze?
>
> Note: You can use Gobuster
>
> Author - Umair

## Solution

The root page of the site has nothing of interest.

The description suggests the use of [Gobuster](https://github.com/OJ/gobuster):

```terminal
gobuster dir -u http://maze.noobarmy.org/ -w common.txt --timeout 60000s
```

The large timeout was required because the server was unstable, and this setting allows gobuster to wait more time before failing if the page was unreachable.

Gobuster found the `projects` folder in the site:

![projects page](/assets/images/cybersec/vulcon-ctf-2020-maze1.png)

The source of the page gives a hint of where to look:

![projects page source](/assets/images/cybersec/vulcon-ctf-2020-maze2.png)

So, accessing http://maze.noobarmy.org/projects/justsomerandomfoldername/image-0.png shows a QR Code. Scanning it gives the message "Hello".

The projects page hinted that his favorite number was 27. So by going to http://maze.noobarmy.org/projects/justsomerandomfoldername/image-27.png gives a QR code with the message "13". Going to http://maze.noobarmy.org/projects/justsomerandomfoldername/image-13.png, there is a QR Code that translates to the message "not".

When I saw this I thought I made a mistake and this was not the correct path. After some time stuck, I downloaded some of the QR code images. All of them were just normal images, except for the image-13.png, where `strings image-13.png` gives the following:

```
IHDR
iTXtXML:com.adobe.xmp
<?xpacket begin='
' id='W5M0MpCehiHzreSzNTczkc9d'?>
<x:xmpmeta xmlns:x='adobe:ns:meta/' x:xmptk='Image::ExifTool 12.04'>
<rdf:RDF xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'>
 <rdf:Description rdf:about=''
  xmlns:dc='http://purl.org/dc/elements/1.1/'>
  <dc:creator>
   <rdf:Seq>
    <rdf:li>aWh5YXBiYXtqQCRfN3UxJF8zaTNhX0BfajNvX3B1QHl5M2F0Mz99</rdf:li>
   </rdf:Seq>
  </dc:creator>
 </rdf:Description>
</rdf:RDF>
</x:xmpmeta>
<?xpacket end='r'?>
IDATx
tJ9Z
gAlQ
mS8G
nIMs
IEND
```

Looks like there is something hidden in the file. More specifically, `aWh5YXBiYXtqQCRfN3UxJF8zaTNhX0BfajNvX3B1QHl5M2F0Mz99` is a base64 encoded strings, which translates to `ihyapba{j@$_7u1$_3i3a_@_j3o_pu@yy3at3?}`.

We know that the flags start with `vulncon`, so by comparing it with `ihyapba`, it is possible to see that it is a rot 13 cipher. Rotating the entire string we finally have the flag:

`vulncon{w@$_7h1$_3v3n_@_w3b_ch@ll3ng3?}`
