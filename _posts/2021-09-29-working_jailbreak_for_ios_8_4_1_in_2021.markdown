---
layout: post
title: "Working jailbreak for iOS 8.4.1 in 2021"
description: "A working method to jailbreak legacy iOS devices"
date: 2021-09-29 18:00:00 +0200
category: misc
tags: misc hacks
image: /assets/2021-09-29-jailbreak-841/jonas-vandermeiren-k_HrQOVGtKs-unsplash.jpg
---

![Jailbreak iOS 8.4.1 in 2021](/assets/2021-09-29-jailbreak-841/jonas-vandermeiren-k_HrQOVGtKs-unsplash.jpg)

I recently needed to jailbreak an old iPad 2 to be able to access it via ssh and use it in one project i'm currently working on. It runs on iOS 8.4.1 (i believe the last version which works somehow acceptable on it). As it's an old iOS so i though jailbreaking it will be easy, but i was wrong!

## Introduction

I started by simply googling "iOS 8.4.1 jailbreak" and followed a few tutorials, visited quite a few shady websites, but not nothing was working anymore. Then i tried to look for a solution on r/legacyjailbreak and narrowed my google search to "ios 8.4.1 jailbreak in **2021**" - still no luck. Some suggested using sideloadly, others to use 3utools. But after spending half a day on setting up virtualbox machine with usb support to try those out (didnt want to mess up my actual system) i also left with nothing.

Although i found some clues:

- Apparently at some point Apple revoked personal dev accounts (without active paid subsctiptions) to sign & install the apps (hence cydiaimpactor doesn't work anymore)
- etas0n (jailbreak for 8.4.1) comes as an app in .ipa - so it needs to be signed by valid certificate to be installed

So i figured i just needed someone with developers certificate to sign it for my device or find some working signing service (paying $99 to get my own dev account was not an option - obviously). And i found it!

## How to jailbreak legacy iOS in 2021?

Fortunately there is a working signing service: [Zeus](https://getzeus.app/)

1. Install the certificate from [https://getzeus.app/](https://getzeus.app/)
2. Install etas0n ipa from [https://etasonjb.tihmstar.net/](https://etasonjb.tihmstar.net/) or just click _jailbreak_ in [3uTools](http://www.3u.com/)

Zeus has etas0n already signed so you don't need to do anything else. And this method will probably work for other iOS then 8.4.1 - you just have to find the right jailbreak ipa for your version.
It works today (2021-09-29) but if you came here later, you just have to find another signing service / someone with certificate.

Disclaimer: _Installing certificate and apps from random source may be not the best idea in terms of security (as well as jailbreak) so do it on your own risk_

---

_Cover photo by [Jonas Vandermeiren](https://unsplash.com/@jonasvandermeiren?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_
![](https://macap.ct8.pl/image.php?url={{site.url}}{{page.url}})
