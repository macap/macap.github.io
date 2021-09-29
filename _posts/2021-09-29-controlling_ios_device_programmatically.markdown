---
layout: post
title: "Controlling iOS device programmatically - make your old iPad useful again with command line"
description: "How to control legacy iOS devices with shell comands or via ssh. Including reacting to events and simulating touch gestures"
date: 2021-09-29 21:41:19 +0200
category: misc
tags: misc hacks
image: /assets/2021-09-29-control-ios/jem-sahagun--kqC3rZEMBI-unsplash.jpg
---

![Control iOS programmatically](/assets/2021-09-29-control-ios/jem-sahagun--kqC3rZEMBI-unsplash.jpg)

There is a chance that you have an old iOS device lying around, and you are wondering if you could make some use of it. It is probably outdated and too slow for daily use, but still - it's not broken so you wouldn't throw it away. If so, you might find this article useful. I'll show you how to control iPad with shell commands (including simulating touch events).

## Outdated iPad as a digital picture frame

I wanted to make a digital picture frame. There are probably some devices on the market, but as i keep all my photos in iCloud I wanted the ability to just select photos to display from my iPhone, instead of sending them somewhere else. My old iPad seemed like a perfect choice. It supports iCloud shared albums, so i could just share one with it and add photos whenever i want.

There was one more thing though - i wanted it to turn it off when its not needed and turn on automatically while i was around. So i needed a way to control the iPad programmatically with a script or API. So how can i do that?

(demo at the end of article)

## Controlling iOS device programmatically

The first thing is to jailbreak your device. As far as i know there is no way to achieve what i describe below without jailbreak. So after getting my tablet out of jail (which wasn't that easy, as it is running on old iOS 8.4.1 - see: [Working jailbreak for iOS 8.4.1 in 2021](https://macap.github.io/misc/2021/09/29/working_jailbreak_for_ios_8_4_1_in_2021.html)) i started looking for a solution to lock, unlock and start slideshow in photos app in programmatical way.

First, I installed [activator](https://cydia.saurik.com/package/libactivator/) from cydia. Its quite popular tool to customize the way buttons, some system events or gestures work. For example you can open specific app when your device connects to WiFi or start playing music when you connect the charger.

With activator i was able to make iPad unlock if i connect a charger and lock after disconnecting it - a thing i could control from my homeassistant box. But the issue was Photos app will stop slideshow after screen was locked, so i'd have to restart it manually.

There is probably some slideshow app which would handle that, but nowadays getting an app which works on 8.4.1 is not that easy. And I wanted to keep things simple by sticking to native Photos app.

I needed something to restart the Photos slideshow after iPad is unlocked. Something that would just touch the start button. I've found out that `simulatetouch` app can handle that.

Last but not least as i wanted to control the process from another device - `openssh` came in handy.

## Installing packages from cydia

You may not need all of them, it depends on what you want to achieve. But I installed:

- [activator](https://cydia.saurik.com/package/libactivator/)
- [simulatetouch](http://cydia.saurik.com/package/kr.iolate.simulatetouch/)
- openSSH
- nano (to easily edit scripts in iPad terminal)
- Terminal (to access iPad terminal from the device itself)

You may also consider [Activate command](http://cydia.saurik.com/package/org.rdharris.activatecommand/) so you can run shell command using activator events without ssh.

## Configuring ssh access

There are plenty of tutorials on how to use openssh on jailbroken device, so i won't duplicate them. tldr:

- connect with `ssh root@<ipad-ip>`
- default password is `alpine`
- **REMEMBER** to change the default password to something else

## Finding the right activator command

Besides of exposing the UI in the device settings, there is also `activator` cli. So while logged in on iPad console you can:

- find the command which you'd like to run with `activator listeners`
- `activate send <listener>` for example `activator send libactivator.system.first-springboard-page` which will show springboard on the screen

## Simulating touches with simulatetouch

SimulateTouch only offers a cli so you can use `stouch` to simulate touch or swipe anywhere on the screen

- run `stouch` to see usage manual:

```
Maciejs-iPad:~ root# stouch
[Usage]
 1. Touch:
    stouch touch x y [orientation]

 2. Swipe:
   stouch swipe fromX fromY toX toY [duration(0.3)] [orientation]

[Example]
   # stouch touch 50 100
   # stouch swipe 50 100 100 200 0.5

[Orientation]
    Portrait:1 UpsideDown:2 Right:3 Left:4
```

For example in my case `stouch touch 40 40 3` will click the app's top left action button in portrait mode. You can also simulate swipes by providing start and end point.

After unlocking, i needed to click "Back", "Slideshow" and "Run" to re-activate the slideshow, so i'd do:

```
stouch touch 40 40 3 # back (left top)
stouch touch 900 35 3 # slideshow (right top)
stouch touch 1000 300 3 # run (right mid)
```

How did i get those coordinates? Well, by try and error. My iPad has 1024x768 resolution and 0x0 point is in left top corner (that info helped me estimate the point coordinates). You can also just take a screenshot and measure the distance in some photo editor.

## The final script run with ssh

At this point i had everything i needed to do the task. Below you can see my scripts:

### To unlock and start slideshow:

```
activator send com.apple.mobileslideshow && sleep 1 && stouch touch 40 40 3 && sleep 1 && stouch touch 900 35 3 && sleep 1 && stouch touch 1000 300 3
```

note: i added some delays with sleep just to make sure interface is ready to handle the click.

### Lock screen

```
activator send libactivator.lockscreen.show
```

I put those commands in two .sh files which i can run by `ssh root@<ip> './slideshow.sh` and `ssh root@<ip> './lockscreen.sh'`. Works like a charm!

## Final thoughts

So that's how i got a nice and cheap digital photo frame. But there are endless possibilites - if you have a little bit of skill in scripting - you can control any app you want by simulating touches, and do a lot with the device itself with automator actions. You could also expose an http endpoint to have an easy to integrate API to your iOS device. For me the next step is to integrate it with homeassistant and trigger my script with motion sensor (so the iPad will start slideshow when someone is around).

- You may find `uiopen http://url` command useful to open http urls (in safari) or app urls
- You don't need ssh - you can add your script to automator action using Activate command package
- I also enabled ssh key based authentication just to avoid typing password every time i want to run the script. SSHd config is pretty standard and its located in `/etc/ssh/sshd_config`
- If cydia UI is too slow for you you can also install packages from console using `apt-get install <package>` like `apt-get install nano`

I hope you find it useful!

## Result

Below you can see a demo how i unlock and start slideshow on my iPad via ssh:

<video src="/assets/2021-09-29-control-ios/output.mp4" width="740" controls muted></video>

---

_Cover photo by [Jem Sahagun](https://unsplash.com/@jemsahagun?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_
![](https://macap.ct8.pl/image.php?url={{site.url}}{{page.url}})
