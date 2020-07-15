---
title: "Hacker101 Oauthbreaker Writeup"
excerpt: "Android challenge from the [Hacker101 CTF](https://ctf.hacker101.com/ctf). This challenge consists of an application with a simple Oauth authentication using WebViews."
date: 2020-07-14T12:59:30-03:00
categories:
  - Cybersecurity
tags:
  - Android
  - Hacker101
  - CTF
---

Android challenge from the [Hacker101 CTF](https://ctf.hacker101.com/ctf). This challenge consists of an application with a simple Oauth authentication. It uses WebViews, which will be our attack vector.

## Source code

After downloading the apk, install it using `adb install oauth.apk`. Next, use [jadx](https://github.com/skylot/jadx) or another tool of your preference to extract the Java code from the apk file.

## First flag

The first place to look is the `AndroidManifest.xml`. This file is important because it provides a lot of information about the app and the actions it can perform.

Notice that there are multiple intent filters defined for this application. In special, there is a [BROWSABLE](https://developer.android.com/reference/android/content/Intent#CATEGORY_BROWSABLE) intent without the `exported=false` parameter, meaning that we can call this intent from outside the app. From [Android documentation](https://developer.android.com/guide/components/intents-filters):

> **Caution**: Using an intent filter isn't a secure way to prevent other apps from starting your components. Although intent filters restrict a component to respond to only certain kinds of implicit intents, another app can potentially start your app component by using an explicit intent if the developer determines your component names. If it's important that only your own app is able to start one of your components, do not declare intent filters in your manifest. Instead, set the exported attribute to "false" for that component.
> Similarly, to avoid inadvertently running a different app's Service, always use an explicit intent to start your own service.

On `MainActivity.java`, an intent is received. If the `redirect_uri` parameter is present, `authRedirectUri` is set as the value of this parameter.

```java
Uri data = getIntent().getData();
if (!(data == null || data.getQueryParameter("redirect_uri") == null)) {
    this.authRedirectUri = data.getQueryParameter("redirect_uri");
}
```

This parameter is later used to create a URL when the user clicks a button:

```java
str = "http://35.190.155.168/1796e9b099/oauth?redirect_url=" + URLEncoder.encode(this.authRedirectUri, StandardCharsets.UTF_8.toString()) + "login&response_type=token&scope=all";
```

It seems to suggest that the user will be redirected to whatever `redirect_url` is. The app then launches an intent to the browser with this URL. 

We know intents can be passed to the app using the intent filter we saw earlier. Let's try to temper with the `redirect_uri` parameter and redirect the user to another page.

Using adb, send an intent to the app that follows the format provided in the intent filter in `AndroidManifest.xml`:

```terminal
$ adb shell am start -a "android.intent.action.VIEW" -n com.hacker101.oauth/.MainActivity -d "oauth://login/?redirect_uri=https://example.com/" 
Starting: Intent { act=android.intent.action.VIEW dat=oauth://login/?redirect_uri=https://example.com/ cmp=com.hacker101.oauth/.MainActivity }
```

{% include figure image_path="/assets/images/cybersec/hacker101-oauthbreaker-flag1.png" alt="Flag 1" caption="Flag is in the URL" %}

Bingo. Our user has been redirected to the page we specified. In a real scenario, an attacker could send the user to a malicious page to steal its OAuth token. Read [this report](https://hackerone.com/reports/328486) for a real-world vulnerability using the same concept as this challenge.

## Second flag

There is another file in the source code, `Browser.java`. This file declares a WebView for the app. What you should notice here is that JavaScript has been enabled in the WebView (`webView.getSettings().setJavaScriptEnabled(true)`), which means it can execute JS code in it. This is dangerous if the WebView can open any web page, as an attacker can convince the user to access a malicious page and execute code inside the application.

The exploit is very similar to the previous one. An intent filter for this class in `AndroidManifest.xml` is exported, meaning we can send our own intents to it. This time, the parameter is `uri`:

```java
Uri data = getIntent().getData();
if (!(data == null || data.getQueryParameter("uri") == null)) {
    str = data.getQueryParameter("uri");
}
```

However, the challenge is not just to redirect the user. We need to explore the fact that JS is enabled. 

In `WebAppInterface.java`, the function `getFlagPath()` has been declared. This function's code is very weird, but its name seems suggestive. So let's craft a page that calls this function when it loads to get the flag path:

```javascript
<div id="result"></div>
<script>document.getElementById("result").innerHTML = iface.getFlagPath()</script>
```

**p.s.:** `iface` is the name of the WebView. It is declared in `Browser.java`.

You might need to store this on a web server. You can use Github pages or the AWS free tier for example.

Finally, craft an intent and send it with adb:

```terminal
$ adb shell am start -d "oauth://final/?uri=<link-of-the-exploit>"
Starting: Intent { dat=oauth://final/?uri=<link-of-the-exploit> }
```

{% include figure image_path="/assets/images/cybersec/hacker101-oauthbreaker-flag2.png" alt="Path to Flag 2" caption="Path to the second flag" %}

And you will get the path for the flag. Append it to the CTF URL to get it: `http://35.190.155.168/1796e9b099/<the-path-here>`