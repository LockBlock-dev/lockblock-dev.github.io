---
date: "2022-03-13"
title: "Corsair iCUE is a Spyware"
tags: ["old", "privacy"]
ShowReadingTime: true
---

## Privacy Policy

From [Corsair Privacy Policy](https://www.corsair.com/us/en/privacy-policy):

> Personal information that Corsair collects: Name, email, address, city, state and zip code, country, phone number, details of purchase(s), credit card information, [...] IP address, device type, unique device identification numbers, browser, location and other technical information [...], how your device has interacted with our website, pages accessed and links clicked.

## Installation

So I spinned up my best Windows Virtual Machine and installed [mitmproxy](https://mitmproxy.org/) and iCUE. In the picture below you can see all the requests the installer made. The installer shows a YouTube video but I removed thoses requests since it's not the topic here.

First, we can see that iCUE installers tries and fails multiple times to get datacollection/config.json.. weird. Then it requests the data about the YouTube videos it shows when installing. Not interesting. But the 4th request is different. In fact, [ipify.org](https://www.ipify.org/) is an API to get your public IP address. This is clearly a spyware behavior.

Then, the iCUE installer requests data that it needs: game sdk effects, videos and its latest version. And finally, we can see that it requests [mixpanel.com](https://mixpanel.com/): a well known tracker.

![Corsair installation mitm results](img/mitm_install.png#center)

iCUE sends base64 encoded data to mixpanel. After decoding it, we can see it sends everything. Every info about your PC, iCUE config, and your ip address requested earlier. You can see a detailed view [here](data_clean.json) (all values removed).

## Usage

When using iCUE, the same tracking is used.

![Corsair usage mitm results](img/mitm_usage.png#center)

## I'm scared

Me too.

## Final words

Seeing that Corsair is doing such shameless tracking really pisses me off. If that's the case for you too, you should get an alternativ to iCUE:

- [ckb-next](https://github.com/ckb-next/ckb-next) (Linux only)
- [OpenRGB](https://openrgb.org/)
