---
title: "ChatGPT on feature phone"
date: 2025-09-12T11:08:01+02:00
draft: false
cover: "images/2025-09-12-nokia/main.jpg"
tags: ['en','2025']
---
Last weekend, I was wondering, how hard  it would be to create a app for a _featurephone_/_dumbphone_/_brick_ however you want to name it.
I never made any application for pre-android era phones and did not have the opportunity. All i know that it can be done in Java. I never actually programmed in java either, but I decided to give it a shot.

## Setting up the IDE
I thought that this would take much longer, and I will probably need to create a VM with something like windows xp, however i found a easy to follow tutorial with working download links that helped me setup the environment in less than 30 minutes. I was suprised that it worked without issue even on windows 10. 
If you also want to setup J2ME environment [here is the link to youtube](https://youtu.be/QI5v6Wa6hvw) (author disallows playback on external sites :/)


Not sure if all the software in that pack is clean, so for safety reasons i decided to run a VM anyway.
Is there a way to setup a environemnt on linux? Probably yes, but personally i just wanted to get going and don't tinker with 32bit libraries.
While researching i found this guide - https://github.com/ading2210/setup-j2me-sdk - maybe it can be useful for someone

## Now what?

Okay, so i had a idea that sounded funny to me - what if we could interact with **AI** on that brick? Nowadays you have phones and computers certified _(ekgh copilot plus pc egkh)_ to run **AI**, so?
Sorry about the clickbait, we actually won't be inferencing AI model on the device itself, but just using the API. I don't even want to attempt something like that - i wasn't even able to find a chipset name for the nokia phones i have in my drawer.

As i never created a app in J2ME, i did some checkup online - it seems like i can just do HTTP request no problem, so let's try to cook something!

For now, i won't be building fully fledged GPT chat, just a single message and response.

## Issue with networking

as you might expect, the old bricks phone with OS from nearly 20 years ago, will not support modern SSL/TLS protocols - so for the quick demo, i just skipped encryption, as i won't be transmitting any sensitive informations and will tear it down soon after. I'm sure it is possible to establish a TLS connection, but i just didn't bother with that.

Instead of crafting my own proxy, i decided to utilize [LiteLLM](https://github.com/BerriAI/litellm) as a gateway - it allows me to connect pretty much any LLM with openai compatible api, create a virtual keys, log requests and responses from underlaying llm models.

Also LiteLLM was helpful for debugging, as i can enable logging of each request and easily validate why it might failed

![screenshot showing litellm dashboard - showing request body from nokia and response from chatgpt API](images/2025-09-12-nokia/litellmresponse.png)

My next issue was that - of course most of this old phones don't have WiFi - and i needed to use cellular connection - in Poland the 2G/Edge is still available so that was not a issue, one prepaid sim card later i had internet connection on my phone.


## Building the application

Overall, as i mentioned before, i want to keep it as barebones as possible, so we will have just a simple form, and will be crafting json by combining the strings and escape the characters manually. Brutal, but we just want to have some fun :)

I also asked Claude Sonnet 4 about some of this things, and was suprised that it returned me a non-halucinated responses and things that actually worked.

In the end, i was suprised how easy it was to build a simple form based app in J2ME. I'm sure that back in the days, when open source and knowledge sharing on the internet was not that popular (e.g most of the things you will probably learn from a book)

I did published the code in github repository - [https://github.com/mgruszkiewicz/j2me-tinyllm](https://github.com/mgruszkiewicz/j2me-tinyllm)

## Demo time!

{{< video src="https://i.issei.space/3lv76V0D.mov" type="video/mp4" >}}

I did speedup the typing part, it was a while since using T9 for daily texting 😅
