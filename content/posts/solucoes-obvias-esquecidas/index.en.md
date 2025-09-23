---
title: "Sometimes a problem's solution..."
date: 2025-04-30
# weight: 1
slug: obvious-forgotten-solutions
# aliases: ["/en/posts/bluetooth-radio-pioneer"]
tags: ["bluetooth", "pioneer"]
author: "SHRMP0" # ["Me", "You"] multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
bsky: "https://bsky.app/profile/saite.shrmp0.com.br/post/3lzjvrdeq4k2i" # link to your bsky post
description: "...is hiding in plain sight, right under our noses."
disableShare: true
disableHLJS: false # to disable highlightjs
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
# images: ["/posts/solucoes-obvias-esquecidas/images/pioneer-deh-s4080bt.png"] # link or path of image for opengraph, twitter-cards
cover:
    image: "/posts/solucoes-obvias-esquecidas/images/pioneer-deh-s4080bt.png" # image path/url
    alt: "Pioneer DEH-S4080BT car radio." # alt text
    caption: "Pioneer DEH-S4080BT car radio." # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/SHRMP0/SHRMP0.github.io/tree/main/content"
    Text: "Suggest changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

It's amazing how everything always ends up breaking down when we're broke and have no spare money to spend, right? If it's something we use almost daily, it's even worse.

Well, a few months ago, the car radio (just like the one in the cover photo of this post) started having a really annoying problem: **the Bluetooth paired devices memory gets erased when the car turns off**. Obviously, this isn't critical, but it's still annoying having to waste time manually connecting it every time I use it.

It's worth noting that this isn't the first time this device has had a problem. I've had this same issue with Bluetooth before, but it was still under warranty. They determined it was a physical defect and replaced the part, fixing the problem.

Furthermore, last year the potentiometer, aka the *little wheel that controls volume and menu navigation*, started failing (something honestly expected from a car radio with over 5 years of use). It was replaced and fixed, but it's now failing again.

Given this history, *you can understand why I'm not considering spending any more money trying to fix it and would rather buy a new one*, right?

This is the moment when you're probably thinking this is just another dumb first world problem and asking yourself "**bitch, why don't you just use the USB port or the 3.5 mm jack??**". So, *my phone doesn't have a 3.5 mm jack* ~~(fuck you Apple for popularizing this)~~, and *plugging my MP3s into the USB wouldn't allow me to answer calls over the radio while driving*.

After suffering for so long, even without money, curiosity made me open the Pioneer website to research possible options that didn't cost an absurd amount and compare them with the current model, where I ended up coming across *that second little button*:

{{< figure align="center" src="/posts/solucoes-obvias-esquecidas/images/screenshot-site-pioneer.png" alt="Product page screenshot on the Pioneer website." caption="A new hope arises...? (buttons says: \"download product firmware\")" >}}

That's when I thought:

> "Why not? It doesn't hurt to try... worst case scenario, this piece of shit will *brick* and serve only as a paperweight, no big deal."

I thought about it for a bit and came to the conclusion that it was worth taking this disproportionate risk.

By the way: congratulations, Pioneer, for doing what should be the bare minimum ~~but many manufacturers don't~~ and making the firmware update files available on the [product page](https://pioneer.com.br/produto/deh-s4080bt/), even though it's already been discontinued.

Anyway, *enough rant*. After downloading and transferring it to the flash drive, all that was left to do was following the instructions to check the installed version and perform the procedure:

{{< video src="/posts/solucoes-obvias-esquecidas/videos/20250424-125106.mp4" type="video/mp4" loop="false" muted="true" caption="Firmware update process demonstration." >}}

After going through those nerve-wracking minutes comparable to updating your PC's BIOS, reconfiguring everything, pairing the cell phone again and turning off the car to test... **holy shit, the problem was actually solved?** I must confess that I wasn't really confident that this would work, but I'm glad that it did because now I won't be tempted to change the radio anymore.
