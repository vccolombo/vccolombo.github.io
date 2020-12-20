---
title: "VULNCON CTF 2020 - Phishy Email writeup"
excerpt: "Writeup for the Phishy Email chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T15:45:30-03:00
categories:
  - Cybersecurity
tags:
  - Memory Forensic
  - VULNCON CTF 2020
  - CTF
---

## Description:

```
To make things easy for me, D E V I N E R told me that he got an email and he believes that the backdoor is installed from that email. Now it's your job to find out from where that email was sent. He is using desktop application for email.

Note: He only remembers that there was smiley sign i.e. ":)" in the email.

Flag Format: vulncon{email}

Author - r3curs1v3_pr0xy
```

## Solution

The process list can be found with `python2 ~/tools/volatility/vol.py --profile=Win7SP1x64 pslist -f dump.raw`. The email client has multiple processes. The memory of process of one of them, PID=2596, can be dumped with Volatility:

`python2 ~/tools/volatility/vol.py --profile=Win7SP1x64 memdump -f dump.raw -D dump/ -p 2596`

The resulting file is huge, but we have a hint about the email: the presence of a smiley face. So by searching it with `strings`:

`strings dump/2596.dmp | grep ":)"`

There will be a lot of garbage, but an object of interest is shown:

```
{"__cls":"Message","_sa":1607780454,"_suc":0,"aid":"c0a7bff6","bcc":[],"cc":[],"date":1607780420,"draft":false,"extraHeaders":{},"files":[],"folder":{"__cls":"Folder","aid":"c0a7bff6","id":"6AaPq2XB7mnhvh5TEUZ2YqHQF64pB8Uw6jckCReHC","path":"[Gmail]/&CTgJLAlI- &CS4JRwky-","role":"all","v":26},"from":[{"email":"sarojchaudhary581@gmail.com","name":"Technical Boy"}],"gMsgId":"1685879973808830832","hMsgId":"CA+MyQpu0vhsQAu__k04LDCPC5ehev4ZWxod0R6DsqwKcB0cA-A@mail.gmail.com","id":"rmUEgN9sa19c9d1WL63d643moWHer4JZgaSYSJHM9","labels":["\\Important","\\Inbox"],"plaintext":false,"remoteFolder":{"__cls":"Folder","aid":"c0a7bff6","id":"6AaPq2XB7mnhvh5TEUZ2YqHQF64pB8Uw6jckCReHC","path":"[Gmail]/&CTgJLAlI- &CS4JRwky-","role":"all","v":26},"remoteUID":5,"replyTo":[{"email":"sarojchaudhary581@gmail.com","name":"Technical Boy"}],"rthMsgId":null,"snippet":"Congratulations! You have 1000$ from an online betting game. Open the attachment and take your money :)","starred":false,"subject":"You have won 1000$","threadId":"t:rmUEgN9sa19c9d1WL63d643moWHer4JZgaSYSJHM9","to":[{"email":"tempmails12345678@gmail.com"}],"unread":false,"v":4}CA+MyQpu0vhsQAu__k04LDCPC5ehev4ZWxod0R6DsqwKcB0cA-A@mail.gmail.com1685879973808830832You have won 1000$_
```

Which contains a field `"from":[{"email":"sarojchaudhary581@gmail.com","name":"Technical Boy"}]`. So the flag is:

`vulncon{sarojchaudhary581@gmail.com}`
