---
layout: post
title: SOP Write Up | BuckeyeCTF
author: Abdo Farwan
date: 2022-11-21 21:00:00 +0800
categories: [Secuirty, WebSec, CTF]
tags: ctf, security, websec, sop
toc: true
image:
  path: /assets/images/blog_1_thumbnail.webp
  width: 800
  height: 500
  alt: Buckeye CTF SOP Write Up thumbnail
---

> This was my first ever CTF writeup & which also won me my first `$100` writeup prize.
{: .prompt-tip }

As a CTF noob coming from solving the Web challenge **“Canary”** that was worth just 70 points, I thought SOP that gave 453 points would be a bit hard. But it turns out pretty simple to some extent.

Anyway before I start I would like to thank **qxxxb** and **ath0** for their responsive replies through out the CTF.

## The application

We should always start by taking a general look around before we start testing and just use the application as a “Normal” user. And so there are two given web pages.


<h2 data-toc-skip>1. The flag page</h2>

When visiting the flag page at <http://18.225.2.140/> you are represented with the following… which turns out to be the source code of the current php file.

Lets go through the code and analyze it.

![Desktop View](/assets/images/blog_1_image_1.webp){: width="972" height="589" }
_Request to http://18.225.2.140/_


Seems simple, if we visit this page with the parameter **?message=flag** it would display the flag …. well at least the fake flag for us.

![Desktop View](/assets/images/blog_1_image_2.webp){: width="972" height="589" }
_Request to http://18.225.2.140/?message=flag_

What if we set it to something else other than flag like **?message=banana** for example.

![Desktop View](/assets/images/blog_1_image_3.webp){: width="972" height="589" }
_Request to http://18.225.2.140/?message=banana_

The first thing which obviously comes to mind when seeing our text being injected to a web page is **XSS**. We know that this challenge is about **SOP** and we are looking for a **SOP bypass** and when we google **SOP bypass** some of the first results are bypasses using **XSS** since if we have an XSS it is essentially a **full and easy** SOP bypass.

Ok cool, lets try XSS

![Desktop View](/assets/images/blog_1_image_4.webp){: width="972" height="589" }

Never mind [**definitely**](https://youtu.be/u0jq42xgTzo) not XSS.

Any way lets move on to analyze the second given web page.

<h2 data-toc-skip>The BOT page</h2>

On visiting the Admin Bot page at <http://18.225.2.140:8000/> you are represented with the following

![Desktop View](/assets/images/blog_1_image_5.webp){: width="972" height="589" }

Pretty straight forward, we give the bot a URL , the bot visits the URL and tells us what happened.

Lets try different URLs:

![Desktop View](/assets/images/blog_1_image_6.webp){: width="972" height="589" }
_Our own web hook_

![Desktop View](/assets/images/blog_1_image_7.webp){: width="972" height="589" }
_https://webhook.site_

So the bot can request external URLs.

![Desktop View](/assets/images/blog_1_image_8.webp){: width="972" height="589" }
_Admin bot 127.0.0.1 request_

It can’t request localhost or 127.0.0.1.

As a complete CTF newbie I stopped here and didn’t know what to do next, the frustration kinda kicked in and I felt lost for some reason.

## Time to take a break.

So in this section I am here to give a quick lesson on hacking and life in general, when you feel frustration kicking in, leave what ever you are doing and take a break. Go to the gym, a store, walk around the house and just come back later.

Fast Forward 5 hours later, I realised that I haven’t really looked up ways to bypass SOP or even read about SOP itself.

Quick research time:

```
https://hackerone.com/reports/761726
https://resources.infosecinstitute.com/topic/bypassing-same-origin-policy-sop/
https://medium.com/swlh/hacking-the-same-origin-policy-f9f49ad592fc⭐
https://dev.to/th3lazykid/day-3-bypassing-the-sop-4a8j
https://thesecurityvault.com/appsec/understanding-cors-and-sop-bypass-techniques/
https://portswigger.net/web-security/cors/same-origin-policy⭐
https://www.hahwul.com/2020/01/12/json-hijacking-sop-bypass-technic-with-cache/
https://github.com/mpgn/ByP-SOP
https://medium.com/datamindedbe/cors-and-the-sop-explained-f59de3a5078 ⭐
...
```

If you haven’t solved the challenge yet, try solving it now while its still online.

## Recap

<h2 data-toc-skip>So… things we understand for now</h2>

1. Flag page only displays the flag for servers/clients? with the IP addresses of **172.16.0.10** and **172.16.0.11**.
2. The bot has no restrictions and can visit any page.
3. There is no XSS in the main application.

<h2 data-toc-skip>Lets do some more poking around</h2>

I have read a write up where the author simply made a request to an endpoint with sensitive information through JavaScript and simple got back the data.

His POC looks like this:

```
<script>
const Http = new XMLHttpRequest();
const url = "http://18.225.2.140/?message=flag";
Http.open("GET", url);
Http.send();
Http.onreadystatechange = (e) => {
console.log(Http.responseText);
};
</script>
```
I tried it but knew for a fact this would not work for us because of ….. drum rolls ….. SOP.

![Desktop View](/assets/images/blog_1_image_9.webp){: width="972" height="589" }
_Loading http://18.225.2.140/?message=flag using a XMLHttpRequest (Failed)_


## JSONP SOP Bypass

While researching I found a SOP bypass method using JSONP.

```
https://medium.com/@minosagap/same-origin-policy-and-ways-to-bypass-250effdc4a12 (More about JSONP)
```

JSONP = JSON with Padding

JSONP is a technique that works around SOP and enables the sharing of data.

And so the `<script>` element is allowed to execute JavaScript code that is retrieved from foreign origins which means that SOP doesn’t block `<script>` element from retrieving content from another origin.

I tried adding the flag page to my html using a script element and it worked!
  
```
<script src="http://18.225.2.140/?message=flag"></script>
```
![Desktop View](/assets/images/blog_1_image_10.webp){: width="972" height="589" }
_Loading http://18.225.2.140/?message=flag using a script element (Worked)_

![Desktop View](/assets/images/blog_1_image_11.webp){: width="972" height="589" }
_Loading http://18.225.2.140/?message=flag using a script element(Worked)_

The script loaded and I now kinda have access to the flag. I say **kinda** because I didn’t know how to leak it to my web hook.

## Str_replace in flag page

Going through the flag page again I realised that I can enter text before and after the flag and php code would just replace the string “Flag”.

![Desktop View](/assets/images/blog_1_image_12.webp){: width="972" height="589" }
_str_replace being used_

So giving the **?message** parameter “the flag” would result in “the buckeye{not_the_flag}” being printed out.

![Desktop View](/assets/images/blog_1_image_13.webp){: width="972" height="589" }

At this point I knew I solved the challenge, using the method above I could easily change the file imported by our script element to valid javascript.

![Desktop View](/assets/images/blog_1_image_14.webp){: width="972" height="589" }
_Payload used : http://18.225.2.140/?message=var%20a%20=%22flag%22;_

![Desktop View](/assets/images/blog_1_image_15.webp){: width="972" height="589" }
_set the flag to a JavaScript variable “a”_

## Exploit

I now made my exploit html page which had a script element with the source set to the flag page with our payload and an image element with source set to “http://webhookURL.com/secret/+ a” to leak the flag to my web hook.

![Desktop View](/assets/images/blog_1_image_16.webp){: width="972" height="589" }
_exploit html page_

Sent the exploit to the Admin Bot:

![Desktop View](/assets/images/blog_1_image_17.webp){: width="972" height="589" }
_exploit worked on Admin BOT but the real flag not leaked_

Somehow this works but doesn’t steal the real flag.

Going through the source code given to us I found this docker-compose file which states that the **flag page** runs on “172.16.0.10” port 80 and the **admin bot page** runs on “172.16.0.11” port 8000.

![Desktop View](/assets/images/blog_1_image_18.webp){: width="972" height="589" }

I quickly check if http://172.16.0.10:80 and http://172.16.0.11:8000 are accessible by the **admin bot** and … they are.

![Desktop View](/assets/images/blog_1_image_19.webp){: width="972" height="589" }
_Admin bot request to http://172.16.0.11:8000 (Success: 200 ok)_

![Desktop View](/assets/images/blog_1_image_20.webp){: width="972" height="589" }
_Admin bot request to http://172.16.0.10:80 (Success: 200 ok)_

<h2 data-toc-skip>Same exploit but using http://172.16.0.10 as the host:</h2>
  
![Desktop View](/assets/images/blog_1_image_21.webp){: width="972" height="589" }
_Final Exloit_

We send the exploit and we quickly get a hit in our web hook with the flag.

![Desktop View](/assets/images/blog_1_image_22.webp){: width="972" height="589" }
_The flag finally_

```
Flag => buckeye{th3_l3tt3r5_0f_S0P_Ar3_1n_JS0NP}
```

This would have been really easy if I didn’t over think stuff.

Thanks for reading.
