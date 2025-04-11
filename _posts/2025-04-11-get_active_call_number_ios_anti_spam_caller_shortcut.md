---
layout: post
title: "Get phone number from active call in ios shortcuts - diy spam call detector"
description: "iOS shortcut to extract the number calling you and checking if its spam online"
date: 2025-04-11 20:24:00 +0100
category: misc
tags: misc hacks
image: /assets/2025-04-11-call-shortcut/hero.png
---

![IOS shortcut spam call detector interface showcase](/assets/2025-04-11-call-shortcut/hero.png)

I’ve been getting a lot of spam calls lately (probably from Klara Sobieraj), and it’s starting to get on my nerves. My usual approach is to google unknown numbers to see if they’re worth answering, but that’s not exactly quick or convenient—especially when the phone is ringing.

I’ve tried a few apps to help with spam call detection, but none of them really worked for me. That’s when I started wondering: could iOS Shortcuts offer a better solution? Turns out, with a little creativity, the answer is yes.

## The problem

iOS Shortcuts doesn’t have any built-in actions for detecting active calls or grabbing the number of an incoming call. This limitation makes it impossible to retrieve the active calling number or determine call status natively within the Shortcuts app.

That said, I was fine with triggering something manually when needed. The real question was: how do I get the calling number? Without that piece of the puzzle, there wasn’t much I could do.

## The Solution

![Shortcut details view](/assets/2025-04-11-call-shortcut/shortcut.png)

After a bit of trial and error, I came up with a hacky workaround that works good enough for me. Here’s how my shortcut is set up:

1. Take a Screenshot

   - When I trigger the shortcut, it takes a screenshot of my phone screen while the call is ringing.

2. Extract the Number Using Regex

   - It processes the screenshot and uses regex to pull out the phone number from the image.
   - It removes whitespaces from matching text

3. Search for Information Online

   - If there was any match, the shortcut scraps a webpage that provides information about phone numbers. Luckily, one of the sites I use allows me to pass the number directly in the URL, which makes this step super easy.
   - The contents of the site are conviniently extracted as plaintext using "get url contents" action

4. Analyze Results
   - The next part scans the webpage for specific keywords like “negative”. If there is a match it displays page contents about why the number might be spam or dangerous. If not, it just shows an “OK” message so I know there’s nothing to worry about.

To make this as seamless as possible, I set up my shortcut to run when I double-tap the back of my phone using iOS’s Back Tap feature. So anytime there is an unknown caller i just tap twice and i have the answer in few seconds. Good enough for me. You can see it in action below (not real time btw):

![Spam call verification demo](/assets/2025-04-11-call-shortcut/demo.gif)

## Summary

This solution might be a bit hacky, but it gets the job done. Plus, being able to extract the currently calling number opens up all kinds of possibilities for other shortcuts you might want to build.

If you’re curious to try this out yourself, you can download my shortcut [here](https://www.icloud.com/shortcuts/8a913c173fe24eb2966b7bd815df0354). Just keep in mind that it’s tailored for Poland (since that’s where I live), so you’ll probably need to tweak it if you’re in another country or using different websites.

Spam calls are annoying, but with a little creativity and some help from iOS Shortcuts, you can take back control!
