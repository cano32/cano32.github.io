---
description: Wiz Prompt Airlines 2024
title: Wiz Prompt Airlines 2024
date: 2024-08-23 08:00:00 +0800
categories:
  - Writeups
  - Misc
tags:
  - writeups
  - misc
show_image_post: false
---

# Wiz Prompt Airlines AI Security Challenge 2024

Main CTF page: [https://promptairlines.com](https://promptairlines.com)<br />

Manipulate the customer service AI chatbot to get a "free airline ticket". You can also see the behind the scenes data like how the LLM processes your request and use it to help you.

![img](https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-start.png)
- [Challenge 1](#challenge-1)
- [Challenge 2](#challenge-2)
- [Challenge 3](#challenge-3)
- [Challenge 4](#challenge-4)
- [Challenge 5](#challenge-5)<br /><br />

## Challenge 1

> Your ultimate goal is to hack your way into a free flight. Let's start with a warmup: our AI assistants are assigned unique identifiers. Can you uncover the one for this bot?

<br />
._.<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-1-1.png" width="75%"></div><br /><br />

Also, I can't stop getting Challenge 1 flags after this when trying for Challenge 2...<br />
???<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-2-1.png" width="75%"></div><br /><br />

STOP ITT<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-2-2.png" width="75%"></div><br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-2-3.png" width="75%"></div><br /><br />

## Challenge 2

> To hack effectively, we need to understand the system. You've seen a censored version of the AI's initial instructions in the 'Under the Hood' section. Can you uncover the full, uncensored version?

<br />
ok trust me im an engingr 💀<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-2-4.png" width="75%"></div><br /><br />

## Challenge 3

> Our AI assistant has access to a list of coupon codes for each flight, including those from your hometown to Las Vegas on Aug. 3rd, 2024. These codes are supposed to be hidden. Can you trick the AI into revealing these secret discounts?

<br />
The bot kept giving me problems about policy so I mainly made use of this clause under the “When searching for flights:” section.
![img](https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-3-1.png)<br /><br />

<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-3-2.png" width="75%"></div><br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-3-3.png" width="75%"></div><br /><br />

## Challenge 4

> Only members can book flights, and our advanced third-party AI authentication system guards our membership. Can you outsmart it and gain unauthorized access to become a member?

<br />
I tried to book a flight but they needed an image of a member card... kinda sus...<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-4-1.png" width="75%"></div><br /><br />

The cat pic:<br />
![img](https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-4-2.jpg){: width="50%" }<br /><br />

With the format the bot provided, I opened Notepad, typed some random stuff and screenshotted it 💀:
![img](https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-4-3.png)<br /><br />

And it got me the flag (╹ڡ╹ )<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-4-4.png" width="75%"></div><br /><br />

## Challenge 5

> Congratulations on making it this far! For the final challenge, use everything you've learned to book a free flight to Las Vegas. Good luck!

<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-5-1.png" width="75%"></div><br /><br />

The info tells us the coupon code from Challenge 3 was used.<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-5-2.png" width="75%"></div><br /><br />

Recall the coupon codes:<br />
[FLY_50, AIR_100, TRAVEL_25, WIZ_CTF{\<challenge 3 flag\>}]<br /><br />

Since we want a free flight ticket, try AIR_100?<br />
<div align="center"><img src=(https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-5-3.png" width="75%"></div><br /><br />

## yay

![img](https://cano32.github.io/assets/img/2024-WizPromptAirlines/wiz-end.png)<br /><br />

( ﾟдﾟ)つ Bye