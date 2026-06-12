---
date: "2021-07-20"
title: "Reverse engineering a mobile game API"
tags: ["old", "reverse-engineering"]
ShowReadingTime: true
---

## How does this game work...

I was playing [Crypto Idle Miner](https://play.google.com/store/apps/details?id=com.horagames.cryptominer), a mobile game about crypto currencies, when this question went through my mind. And I think I'm not the only one who has thought of this using an app.

It's time to get my hands dirty.

## Yes but how

That's a good question. How on earth can I spy on this bad boy... I'm not on a web browser so no "Inspect Element".

My plan was simple: doing a [Man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack), also called MITM, to see the requests between the app and the game server. So, like any human on this planet, I went on google and casually researched: _"MITM on Android apps"_. After looking at the results I found the holy grail, [mitmproxy](https://mitmproxy.org/).

## mitmproxy

This tool makes the whole process of a MITM attack very simple by providing you with the "server" side app (on your PC) and the certificate for the device to be spied on. I downloaded the binaries, connected my phone to my PC, downloaded and installed the certificate and finally started the game.

![no internet error](img/no_internet_error_cim.png#center)

No internet? Why, I'm connected to the WIFI it should work... Once again, I searched on google: _"mitmproxy no internet on apps"_ and realized that it couldn't work. In fact, since Android 7, applications don't trust user certificates for security concerns, I guess. I had to find a workaround. First I looked how can I add my very safe official certificate to the game and found [apk-mitm](https://github.com/shroudedcode/apk-mitm).

## apk-mitm

This script attempts to disable [certificate pinning](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning#what-is-pinning) and tries to allow user certificates by patching the apk code. Then it rebuilds it and signs it. Simple.

![apk mitm example](img/apk_mitm_example.png#center)

I just needed the game apk: for that I tried to get the apk from the Play Store and after from my phone. Both apk were patched but I couldn't install them on my phone... Back to square one. I went back on mitmproxy documentation and found at the bottom the [solution](https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android).

## Android Studio

Who would have thought I would need Android tools to get around Android's own security. I followed mitmproxy guide, installed Android Studio, created a android virtual device (AVD) with the naivety to think that I will be able to use the Google Play Store.

![google play apis](img/android_studio_apis.png#center)

In fact, Google are smart, AVDs with the Google APIs are protected so that I cannot write my certificate in the system. After learning that I recreated another AVD without Google APIs this time. Now that I have an Android with my very safe official certificate in its system, I have to install the game but without the Play Store. For this I downloaded and used [Aurora Store](https://auroraoss.com/). It is a great alternative to Google Play in my opinion. Finally, I can now spy on this game and expose all its secrets.

## Acting as a 3 letter agency

Now that I can see everything my phone does with the internet in clear just like a 3 letter agency does, let's dive into it.

![mitm preview](img/mitm_preview.png#center)

Simply use the application a bit and you will see many requests to the game server. There are unwanted requests from other services but let's focus on the main topic.

Looking at the URLs of these requests, it looks like we found a secret undocumented API known only by the developers: `https://terra.horahub.com/api`. It's time to try everything I can do in the game and write down all the endpoints.

At the time of writing, I found about twenty endpoints, but I saw on Twitter that the developers were going to update their API.

## The end of the story

So, how does this game work? This game uses a private REST API. When you start the game, it requests this API to get the common variables (prices, objects, ...), the infos about researches, buildings, ... and it requests your savegame. From time to time, the game saves your progress and sends it to the server. And there are also endpoints for some functions in the game (achievements, claiming a promo code, ...)

After writing all endpoints, I worked on a JavaScrypt library to interact with this specific API and published it as a NPM package. If you are interested, [here is the link](https://github.com/LockBlock-dev/crypto-idle-miner.js).

## tl;dr

- [mitmproxy](https://mitmproxy.org/) is a nice tool for MITM attacks
- Android tools can get around Android's own security
- Google are smart
- A phone makes many (too) requests
- This game works with a private REST API
